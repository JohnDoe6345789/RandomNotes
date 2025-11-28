


```python


import json
import os
import subprocess
import sys
from datetime import datetime
from typing import Dict, List, Tuple

from PyQt6.QtCore import QTimer, QObject, pyqtSlot
from PyQt6.QtWidgets import (
    QApplication,
    QMenu,
    QMessageBox,
    QSystemTrayIcon,
)
from PyQt6.QtGui import QIcon


CHECK_INTERVAL_MS: int = 60_000

WATCHED_FIELDS: List[str] = [
    "AMServiceEnabled",
    "AntivirusEnabled",
    "RealTimeProtectionEnabled",
    "BehaviorMonitorEnabled",
    "IoavProtectionEnabled",
    "NISRealTimeProtectionEnabled",
    "IsTamperProtected",
]

LOG_FILE: str = os.path.join(
    os.path.dirname(os.path.abspath(__file__)),
    "defender_watchdog_qt.log",
)


def ensure_windows() -> None:
    """Exit if not running on Windows."""
    if os.name != "nt":
        print("Windows only.", file=sys.stderr)
        sys.exit(1)


def run_powershell(ps_command: str) -> str:
    """Run a PowerShell command and return stdout."""
    completed = subprocess.run(
        [
            "powershell",
            "-NoLogo",
            "-NoProfile",
            "-NonInteractive",
            "-Command",
            ps_command,
        ],
        capture_output=True,
        text=True,
        check=False,
    )
    if completed.returncode != 0:
        raise RuntimeError(
            f"PowerShell failed: {completed.returncode}\n{completed.stderr}"
        )
    return completed.stdout.strip()


def build_ps_status_command() -> str:
    """Build the PowerShell command to fetch Defender status."""
    selection = ",".join(WATCHED_FIELDS)
    return (
        "Get-MpComputerStatus | "
        f"Select-Object {selection} | "
        "ConvertTo-Json -Compress"
    )


def get_defender_status() -> Dict[str, bool]:
    """Return Defender status for watched fields."""
    ps_command = build_ps_status_command()
    output = run_powershell(ps_command)
    if not output:
        raise RuntimeError("Empty output from Get-MpComputerStatus.")
    try:
        data = json.loads(output)
    except json.JSONDecodeError as exc:
        raise RuntimeError(f"Bad JSON from PowerShell: {exc}") from exc
    if not isinstance(data, dict):
        raise RuntimeError(f"Unexpected JSON shape: {data}")
    status: Dict[str, bool] = {}
    for field in WATCHED_FIELDS:
        value = data.get(field, None)
        status[field] = bool(value is True)
    return status


def find_bad_fields(status: Dict[str, bool]) -> List[str]:
    """Return fields that are not OK."""
    return [name for name, ok in status.items() if not ok]


def now_iso() -> str:
    """Return current local time as ISO-like string."""
    return datetime.now().strftime("%Y-%m-%d %H:%M:%S")


def log_line(message: str) -> None:
    """Append message to log file with timestamp."""
    line = f"{now_iso()}  {message}\n"
    try:
        with open(LOG_FILE, "a", encoding="utf-8") as handle:
            handle.write(line)
    except OSError:
        print(line, file=sys.stderr, end="")


def build_alert_message(bad_fields: List[str]) -> str:
    """Human-readable alert message."""
    joined = ", ".join(bad_fields)
    return (
        "Windows Defender watchdog detected a problem.\n\n"
        "The following protection flags are NOT enabled:\n"
        f"  {joined}\n\n"
        "Open Windows Security → Virus & threat protection to investigate."
    )


class DefenderWatchdog(QObject):
    """Qt-based watchdog that lives in the system tray."""

    def __init__(
        self,
        tray: QSystemTrayIcon,
        parent: QObject | None = None,
    ) -> None:
        super().__init__(parent)
        self._tray = tray
        self._timer = QTimer(self)
        self._timer.setInterval(CHECK_INTERVAL_MS)
        self._timer.timeout.connect(self.check_once)
        self._last_bad_fields: List[str] = []
        self._first_run_done = False

    def start(self) -> None:
        """Start periodic checking."""
        self._timer.start()
        self.check_once()

    @pyqtSlot()
    def check_once(self) -> None:
        """Run one status check and alert if needed."""
        try:
            status = get_defender_status()
            bad_fields = find_bad_fields(status)
        except Exception as exc:  # noqa: BLE001
            log_line(f"ERROR during check: {exc}")
            if not self._first_run_done:
                self._show_error_dialog(str(exc))
            self._first_run_done = True
            return

        self._first_run_done = True
        if bad_fields:
            log_line(f"STATUS: BAD fields = {', '.join(bad_fields)}")
        else:
            log_line("STATUS: All fields OK.")

        if sorted(bad_fields) == sorted(self._last_bad_fields):
            return

        self._last_bad_fields = list(bad_fields)
        if bad_fields:
            self._raise_alert(bad_fields)
        else:
            self._notify_recovered()

    def _raise_alert(self, bad_fields: List[str]) -> None:
        """Show tray notification and dialog about bad status."""
        message = build_alert_message(bad_fields)
        log_line("ALERT: " + message.replace("\n", " "))
        self._tray.showMessage(
            "Windows Defender Watchdog",
            "Protection issue detected. Click for details.",
            QSystemTrayIcon.MessageIcon.Warning,
            10_000,
        )
        QMessageBox.warning(
            None,
            "Windows Defender Watchdog",
            message,
            QMessageBox.StandardButton.Ok,
        )

    def _notify_recovered(self) -> None:
        """Notify that status is back to normal."""
        log_line("INFO: All monitored Defender flags are OK again.")
        self._tray.showMessage(
            "Windows Defender Watchdog",
            "All monitored Defender flags are OK again.",
            QSystemTrayIcon.MessageIcon.Information,
            5000,
        )

    def _show_error_dialog(self, message: str) -> None:
        """Show an error dialog on startup failures."""
        QMessageBox.critical(
            None,
            "Windows Defender Watchdog - Error",
            f"Failed to check Defender status:\n\n{message}",
            QMessageBox.StandardButton.Ok,
        )


def create_tray_icon(app: QApplication) -> QSystemTrayIcon:
    """Create a tray icon with a simple menu."""
    tray = QSystemTrayIcon()
    # You can point this to a real .ico if you like.
    tray.setIcon(app.style().standardIcon(
        getattr(QStyle, "StandardPixmap", object)().__dict__.get(
            "SP_ShieldIcon", app.style().standardIcon(0)
        )
    ))
    menu = QMenu()
    quit_action = menu.addAction("Quit")
    quit_action.triggered.connect(app.quit)
    tray.setContextMenu(menu)
    tray.setToolTip("Windows Defender Watchdog")
    tray.show()
    return tray


def main() -> None:
    """Main entry point."""
    ensure_windows()
    app = QApplication(sys.argv)
    tray = QSystemTrayIcon()
    # Fallback icon: generic if shield not available.
    tray.setIcon(app.style().standardIcon(0))
    menu = QMenu()
    quit_action = menu.addAction("Quit")
    quit_action.triggered.connect(app.quit)
    tray.setContextMenu(menu)
    tray.setToolTip("Windows Defender Watchdog")
    tray.show()

    watchdog = DefenderWatchdog(tray)
    watchdog.start()

    sys.exit(app.exec())


if __name__ == "__main__":
    main()



```



end
