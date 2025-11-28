
```


import ctypes
import os
import sys
import winreg


TARGET_DIR = r"C:\miniconda3"
TARGET_EXE = os.path.join(TARGET_DIR, "python.exe")


def ensure_miniconda_exists() -> None:
    if not os.path.isfile(TARGET_EXE):
        print(f"ERROR: {TARGET_EXE} does not exist.")
        print("Check that Miniconda is installed in C:\\miniconda3.")
        sys.exit(1)


def find_path_value(
    root: int,
    subkey: str,
) -> tuple[str | None, str | None, int | None]:
    """
    Locate the PATH value in a registry key, case-insensitively.

    Returns (actual_name, value, reg_type) or (None, None, None) if not present.
    """
    try:
        with winreg.OpenKey(root, subkey, 0, winreg.KEY_READ) as key:
            i = 0
            while True:
                try:
                    name, value, reg_type = winreg.EnumValue(key, i)
                except OSError:
                    break
                if name.lower() == "path":
                    return name, value, reg_type
                i += 1
    except FileNotFoundError:
        return None, None, None
    return None, None, None


def update_path_value(
    root: int,
    subkey: str,
    label: str,
) -> None:
    """
    Prepend TARGET_DIR (and its Scripts dir) to PATH for the given root/subkey.

    root: HKEY_CURRENT_USER or HKEY_LOCAL_MACHINE
    subkey: registry path
    label: label used for printing (e.g. "User" or "System")
    """
    actual_name, current_value, reg_type = find_path_value(root, subkey)

    if current_value is None:
        current_value = ""
        actual_name = "Path"
        reg_type = winreg.REG_EXPAND_SZ

    parts = [p for p in current_value.split(";") if p]

    # Remove any existing miniconda entries (case-insensitive match)
    def is_miniconda_path(p: str) -> bool:
        normalized = os.path.normcase(os.path.normpath(p.strip()))
        return normalized.startswith(os.path.normcase(TARGET_DIR))

    parts = [p for p in parts if not is_miniconda_path(p)]

    # Prepend our desired entries
    new_parts = [TARGET_DIR, os.path.join(TARGET_DIR, "Scripts")] + parts
    new_path = ";".join(new_parts)

    print(f"[{label}] Old PATH begins with:")
    print("  " + ";".join(parts[:3]))
    print(f"[{label}] New PATH begins with:")
    print("  " + ";".join(new_parts[:3]))

    # Write back
    access = winreg.KEY_SET_VALUE
    try:
        with winreg.OpenKey(root, subkey, 0, access) as key:
            winreg.SetValueEx(key, actual_name, 0, reg_type, new_path)
        print(f"[{label}] PATH updated successfully.")
    except PermissionError:
        print(f"[{label}] ERROR: Permission denied when writing PATH.")
        print("  -> Run this script from an elevated (Run as administrator) console")
        print("     if you want to modify the system PATH.")
        raise
    except FileNotFoundError:
        # Create the key if it doesn't exist
        with winreg.CreateKey(root, subkey) as key:
            winreg.SetValueEx(key, actual_name, 0, winreg.REG_EXPAND_SZ, new_path)
        print(f"[{label}] PATH created and set successfully.")


def broadcast_env_change() -> None:
    """
    Notify Windows that environment variables changed, so new processes
    see the updated PATH without a reboot. Existing shells still need restart.
    """
    HWND_BROADCAST = 0xFFFF
    WM_SETTINGCHANGE = 0x001A

    ctypes.windll.user32.SendMessageW(
        HWND_BROADCAST,
        WM_SETTINGCHANGE,
        0,
        "Environment",
    )


def main() -> None:
    ensure_miniconda_exists()

    # 1) Update user PATH (no admin needed)
    update_path_value(
        winreg.HKEY_CURRENT_USER,
        r"Environment",
        label="User",
    )

    # 2) Try to update system PATH (needs admin)
    system_error = False
    try:
        update_path_value(
            winreg.HKEY_LOCAL_MACHINE,
            r"SYSTEM\CurrentControlSet\Control\Session Manager\Environment",
            label="System",
        )
    except PermissionError:
        system_error = True

    # 3) Broadcast change so new consoles see it
    broadcast_env_change()

    print()
    print("DONE.")
    print(f"- Miniconda directory set to: {TARGET_DIR}")
    print("- Close and reopen any Command Prompt / PowerShell windows.")
    if system_error:
        print(
            "- System PATH was NOT changed (no admin). "
            "User PATH is updated, but a system-level Python may still win."
        )
    print()
    print("After reopening a terminal, check with:")
    print("  where python")
    print("  python --version")


if __name__ == "__main__":
    main()



```


ernd

