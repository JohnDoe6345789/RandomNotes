```python

#!/usr/bin/env python
# mypy: strict
"""
Code generator for a local Chocolatey package that installs
Parsec for Windows (remote desktop / game streaming).

- Creates a NEW build folder in the current directory.
- Downloads the official installer to a subfolder.
- Computes SHA-256 of the downloaded installer.
- Emits:
    parsec/
      parsec.nuspec
      tools/chocolateyinstall.ps1
      tools/chocolateyuninstall.ps1

Usage (PowerShell):

    cd C:\some\empty\working\folder
    python generate_parsec_choco.py
    cd parsec-build\parsec
    choco pack
    choco install parsec -s .

This script does NOT modify files directly in your current directory;
it only creates new subfolders under it.
"""

from __future__ import annotations

import hashlib
import sys
import urllib.request
from pathlib import Path
from typing import Final


# Official Parsec Windows installer endpoint.
# NOTE: This may change over time; update if Parsec modify their URL.
INSTALLER_URL: Final[str] = (
    "https://builds.parsec.app/package/parsec-windows.exe"
)

PACKAGE_ID: Final[str] = "parsec"
PACKAGE_VERSION: Final[str] = "0.1.0"

BUILD_ROOT_NAME: Final[str] = "parsec-build"
DOWNLOAD_DIR_NAME: Final[str] = "downloads"
PACKAGE_DIR_NAME: Final[str] = PACKAGE_ID
INSTALLER_BASENAME: Final[str] = "parsec-windows.exe"


def die(message: str) -> "NoReturn":
    """Exit with an error message."""
    print(f"ERROR: {message}", file=sys.stderr)
    raise SystemExit(1)


def ensure_new_dir(path: Path) -> None:
    """
    Ensure a brand new directory is created.

    Refuses to continue if the path already exists.
    """
    if path.exists():
        die(f"Refusing to overwrite existing path: {path}")
    path.mkdir(parents=True, exist_ok=False)


def download_bytes(url: str) -> bytes:
    """
    Download raw bytes from the given URL.

    Keeps the function small and testable by not dealing
    with file IO here.
    """
    with urllib.request.urlopen(url) as response:
        data = response.read()
    if not data:
        die("Downloaded zero bytes from installer URL")
    return data


def compute_sha256(data: bytes) -> str:
    """
    Compute SHA-256 checksum as a hex string.
    """
    hasher = hashlib.sha256()
    hasher.update(data)
    return hasher.hexdigest()


def write_bytes(path: Path, data: bytes) -> None:
    """
    Write raw bytes to a file, creating parent dirs if needed.
    """
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_bytes(data)


def write_text(path: Path, content: str) -> None:
    """
    Write text content to a file, creating parent dirs if needed.
    """
    path.parent.mkdir(parents=True, exist_ok=True)
    path.write_text(content, encoding="utf-8")


def build_nuspec() -> str:
    """
    Build the .nuspec XML content.
    """
    lines = [
        '<?xml version="1.0"?>',
        '<package '
        'xmlns="http://schemas.microsoft.com/packaging/2015/06/nuspec.xsd">',
        "  <metadata>",
        f"    <id>{PACKAGE_ID}</id>",
        f"    <version>{PACKAGE_VERSION}</version>",
        "    <title>Parsec (Remote Desktop / Game Streaming)</title>",
        "    <authors>Your Name</authors>",
        "    <owners>Your Name</owners>",
        "    <projectUrl>https://parsec.app/</projectUrl>",
        "    <licenseUrl>"
        "https://unity.com/legal/parsec-additional-terms"
        "</licenseUrl>",
        "    <requireLicenseAcceptance>true</requireLicenseAcceptance>",
        "    <description>",
        "      Installs the Parsec Windows client by downloading the official",
        "      installer directly from Parsec's servers.",
        "      This package does not redistribute the installer binary.",
        "    </description>",
        "    <tags>parsec remote-desktop streaming gaming</tags>",
        "  </metadata>",
        "</package>",
        "",
    ]
    return "\n".join(lines)


def build_install_ps1(checksum: str) -> str:
    """
    Build chocolateyinstall.ps1 with enforced checksum.
    """
    lines = [
        "$ErrorActionPreference = 'Stop'",
        "",
        f"$packageName = '{PACKAGE_ID}'",
        "$toolsDir = Split-Path -Parent $MyInvocation.MyCommand.Definition",
        "",
        "# Official Parsec setup URL. Update if Parsec change it.",
        f"$url64 = '{INSTALLER_URL}'",
        "",
        f"$checksum = '{checksum}'",
        "$checksumType = 'sha256'",
        "",
        "# /silent and /nocleanuser are documented Parsec installer flags.",
        "$silentArgs = '/silent /nocleanuser'",
        "",
        "Install-ChocolateyPackage \\",
        "  -PackageName $packageName \\",
        "  -FileType 'exe' \\",
        "  -SilentArgs $silentArgs \\",
        "  -Url64bit $url64 \\",
        "  -Checksum $checksum \\",
        "  -ChecksumType $checksumType",
        "",
    ]
    return "\n".join(lines)


def build_uninstall_ps1() -> str:
    """
    Build chocolateyuninstall.ps1 content.

    Parsec installs an uninstaller at:
      C:\\Program Files\\Parsec\\uninstall.exe

    There is no officially documented silent-uninstall flag,
    so this script just calls the uninstaller and lets it
    handle any UI it needs.
    """
    lines = [
        "$ErrorActionPreference = 'Stop'",
        "",
        "$uninstallPath = Join-Path $env:ProgramFiles 'Parsec\\\\uninstall.exe'",
        "",
        "if (Test-Path $uninstallPath) {",
        "    Write-Host \"Running Parsec uninstaller at $uninstallPath\"",
        "    Start-Process -FilePath $uninstallPath -Wait",
        "} else {",
        "    Write-Warning 'Parsec uninstaller not found; nothing to run.'",
        "}",
        "",
    ]
    return "\n".join(lines)


def main() -> None:
    """
    Orchestrate the build:

    - Create build root folder.
    - Download installer bytes and save them.
    - Compute SHA-256.
    - Generate Chocolatey package skeleton wired
      to the URL and checksum.
    """
    cwd = Path.cwd()
    build_root = cwd / BUILD_ROOT_NAME
    ensure_new_dir(build_root)

    download_dir = build_root / DOWNLOAD_DIR_NAME
    package_dir = build_root / PACKAGE_DIR_NAME
    tools_dir = package_dir / "tools"

    print(f"Downloading installer from: {INSTALLER_URL}")
    data = download_bytes(INSTALLER_URL)
    checksum = compute_sha256(data)

    installer_path = download_dir / INSTALLER_BASENAME
    write_bytes(installer_path, data)

    nuspec_path = package_dir / f"{PACKAGE_ID}.nuspec"
    install_ps1_path = tools_dir / "chocolateyinstall.ps1"
    uninstall_ps1_path = tools_dir / "chocolateyuninstall.ps1"

    write_text(nuspec_path, build_nuspec())
    write_text(install_ps1_path, build_install_ps1(checksum))
    write_text(uninstall_ps1_path, build_uninstall_ps1())

    print(f"Build root: {build_root}")
    print(f"Installer saved to: {installer_path}")
    print(f"SHA-256 checksum: {checksum}")
    print("Next steps:")
    print(f"  cd {build_root / PACKAGE_DIR_NAME}")
    print("  choco pack")
    print("  choco install parsec -s .")


if __name__ == "__main__":
    main()


```

end
