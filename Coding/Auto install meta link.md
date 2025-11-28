```python

#!/usr/bin/env python
# mypy: strict
"""
Code generator for a local Chocolatey package that installs
Meta Quest Link (Oculus PC app).

- Creates a NEW build folder in the current directory.
- Downloads the official installer to a subfolder.
- Computes SHA-256 of the downloaded installer.
- Emits:
    meta-quest-link/
      meta-quest-link.nuspec
      tools/chocolateyinstall.ps1
      tools/chocolateyuninstall.ps1

Usage (PowerShell):

    cd C:\some\empty\working\folder
    python generate_meta_quest_link_choco.py
    cd meta-quest-link-build\meta-quest-link
    choco pack
    choco install meta-quest-link -s .

This script does NOT modify files directly in your current directory;
it only creates new subfolders under it.
"""

from __future__ import annotations

import hashlib
import sys
import urllib.request
from pathlib import Path
from typing import Final


# You can update this URL if Meta change their download endpoint.
INSTALLER_URL: Final[str] = (
    "https://securecdn.oculus.com/binaries/download/?id=3754726804808697"
)

PACKAGE_ID: Final[str] = "meta-quest-link"
PACKAGE_VERSION: Final[str] = "0.1.0"

BUILD_ROOT_NAME: Final[str] = "meta-quest-link-build"
DOWNLOAD_DIR_NAME: Final[str] = "downloads"
PACKAGE_DIR_NAME: Final[str] = PACKAGE_ID
INSTALLER_BASENAME: Final[str] = "OculusSetup.exe"


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
        "    <title>Meta Quest Link (Oculus PC App)</title>",
        "    <authors>Your Name</authors>",
        "    <owners>Your Name</owners>",
        "    <projectUrl>https://www.meta.com/quest/</projectUrl>",
        "    <licenseUrl>"
        "https://www.meta.com/legal/quest/meta-quest-terms/"
        "</licenseUrl>",
        "    <requireLicenseAcceptance>true</requireLicenseAcceptance>",
        "    <description>",
        "      Installs the Meta Quest Link / Oculus PC software by",
        "      downloading the official installer from Meta's servers.",
        "      This package does not redistribute the installer binary.",
        "    </description>",
        "    <tags>meta quest link oculus vr pc</tags>",
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
        "# Official Meta / Oculus setup URL. Update if Meta change it.",
        f"$url64 = '{INSTALLER_URL}'",
        "",
        f"$checksum = '{checksum}'",
        "$checksumType = 'sha256'",
        "",
        "$silentArgs = '--unattended'",
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
    """
    lines = [
        "$ErrorActionPreference = 'Stop'",
        "",
        "$setupPath = Join-Path $env:ProgramFiles 'Oculus\\OculusSetup.exe'",
        "",
        "if (Test-Path $setupPath) {",
        "    Write-Host \"Uninstalling Meta Quest Link via $setupPath\"",
        "    Start-Process -FilePath $setupPath \\",
        "                  -ArgumentList '--unattended /uninstall' \\",
        "                  -Wait -NoNewWindow",
        "} else {",
        "    Write-Warning 'OculusSetup.exe not found, nothing to uninstall.'",
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
    print("  choco install meta-quest-link -s .")


if __name__ == "__main__":
    main()


```

end
