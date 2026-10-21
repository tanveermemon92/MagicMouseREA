# Magic Mouse Lightning Port 2020 — Windows Driver & Scroll Fix

**Author: Tanveer Ahmed**

A Windows driver project for restoring two-finger scrolling on Apple Magic Mouse hardware, developed through reverse engineering, Windows HID investigation, and AI-assisted development using Cursor and Claude.

The project investigates why scrolling can stop working on Windows after a Bluetooth idle disconnect, even while cursor movement continues. It documents two independent approaches: registering Apple's existing Boot Camp mouse filter driver and implementing a separate KMDF driver.

- **Project name:** Magic Mouse Lightning Port 2020
- **Author:** Tanveer Ahmed
- **Platform:** Windows 10/11
- **License:** MIT (see the repository's `LICENSE` file)

## Read the Guides

- [Magic Mouse Windows Driver Website](https://drivers.magictray.app/)
- [Install the Driver on Windows 11](https://drivers.magictray.app/install.html)
- [Fix Magic Mouse Scrolling](https://drivers.magictray.app/scroll-fix.html)
- [Magic Mouse Battery Percentage on Windows](https://drivers.magictray.app/battery.html)
- [Magic Mouse Driver FAQ](https://drivers.magictray.app/faq.html)

## Companion Application — Magic Tray

[Magic Tray](https://github.com/LesleyMurfin/magic-tray) is a companion Windows system-tray utility that identifies the current mouse-driver state and reports pointer, scrolling, and battery information.

Website: [magictray.app](https://magictray.app/)

Magic Tray supports three states:

- Windows default Apple/Microsoft driver stack.
- Apple's driver registered as a lower filter.
- The custom KMDF driver.

It adapts its battery-reading method to the active driver configuration.

## Contents

- [Project Overview](#project-overview)
- [Reverse Engineering and Development](#reverse-engineering-and-development)
- [AI-Assisted Development with Cursor and Claude](#ai-assisted-development-with-cursor-and-claude)
- [Which Driver Should You Choose?](#which-driver-should-you-choose)
- [What This Project Fixes](#what-this-project-fixes)
- [Supported Hardware](#supported-hardware)
- [System Requirements](#system-requirements)
- [Driver 1 — Apple Filter Driver](#driver-1--apple-filter-driver)
- [Driver 2 — Custom KMDF Driver](#driver-2--custom-kmdf-driver)
- [Quick Installation](#quick-installation)
- [Verify the Installation](#verify-the-installation)
- [Uninstall](#uninstall)
- [Troubleshooting](#troubleshooting)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Attribution and Acknowledgements](#attribution-and-acknowledgements)
- [License](#license)
- [Support and Security](#support-and-security)

## Project Overview

This repository investigates and documents Windows scrolling issues affecting Apple Magic Mouse devices, particularly the 2024 USB-C model identified by Bluetooth product ID `0x0323`.

The problem occurs when scrolling works immediately after pairing but stops responding after an extended idle period and Bluetooth disconnect. Cursor movement may continue normally even though two-finger scrolling has stopped.

This project documents two independent approaches to addressing the problem.

| Feature | Driver 1 — Apple Filter | Driver 2 — KMDF |
|---|---|---|
| Implementation | Apple's existing `applewirelessmouse.sys` | Custom KMDF driver |
| Scroll processing | Apple's multitouch translation | Custom touch-report processing |
| Scroll sensitivity | Apple's default behaviour | Configurable through `ScrollStep` |
| Battery reporting | Through Magic Tray, using a temporary mode switch | Through Magic Tray, reading HID Input `0x90` |
| Build required | No | Yes, when building from source |
| Test signing | Not expected; needs verification on the target configuration | Required for the community-signed build |
| Main advantage | Reuses Apple's existing driver | Custom scrolling and recovery logic |

**Important:** These drivers are alternative approaches. Do not install both at the same time.

## Reverse Engineering and Development

This project was developed by **Tanveer Ahmed** through investigation of Windows Bluetooth HID behaviour, driver configuration, and Apple Magic Mouse multitouch functionality.

The reverse-engineering work focuses on understanding the relationship between Bluetooth device initialization, HID collections, touch reports, and Windows input handling.

The investigation examines a failure mode in which scrolling stops after a Bluetooth idle disconnect and subsequent device-property synchronization while pointer movement continues.

The project explores two different solutions:

- **Apple filter approach:** Register Apple's existing mouse filter as a lower filter on the relevant Bluetooth HID device.
- **KMDF approach:** Process touch reports directly and generate Windows scroll events independently of Apple's multitouch translation path.
- **System diagnostics:** Inspect device hardware IDs, driver states, HID collections, Windows Plug and Play behaviour, and system event logs.
- **Recovery testing:** Evaluate scrolling after Bluetooth reconnection, sleep and wake, device rescans, and system reboots.

The goal is to understand the underlying failure, document reproducible tests, and provide alternative approaches to restoring scrolling on Windows.

## AI-Assisted Development with Cursor and Claude

Cursor and Claude were used as AI-assisted development tools during the investigation and implementation process.

They assisted with development activities such as:

- Analysing driver source code and PowerShell scripts.
- Investigating possible causes of HID collection and Bluetooth reconnection problems.
- Exploring Windows driver architecture and configuration.
- Structuring implementation changes and reviewing code.
- Drafting installation, verification, and troubleshooting scripts.
- Organising test observations and technical documentation.
- Investigating build errors and potential fixes.
- Improving the readability and maintainability of the project.

The development process combined AI-assisted coding with reverse engineering, debugging, and technical testing.

AI-generated suggestions are not a substitute for validation. Kernel-mode driver changes must be tested on the intended hardware and Windows configuration before being considered reliable.

## Which Driver Should You Choose?

The repository documents three possible driver states.

### State 0 — Windows Default

Windows uses its default Bluetooth HID driver stack.

- Pointer movement works.
- Scrolling may be missing or may stop after an idle disconnect.
- No additional driver installation is required.

### Driver 1 — Apple Filter Driver

This approach uses Apple's existing `applewirelessmouse.sys` without modifying the binary.

- Reuses Apple's multitouch translation.
- Does not require compiling a driver.
- Is expected to retain Apple's original driver signature.
- Is expected to work without Test Mode, although this must be verified with Test Mode disabled on the target system.

### Driver 2 — Custom KMDF Driver

This approach uses a separately implemented Windows kernel-mode driver.

- Generates scroll events directly from touch reports.
- Supports configurable scroll sensitivity.
- Includes multitouch recovery logic.
- Requires a community-signing setup for the unsigned-by-Microsoft development build.

Choose one approach based on your hardware, Windows security configuration, and technical requirements.

## What This Project Fixes

The project investigates a Magic Mouse scrolling failure on Windows 10/11 in which:

- Scrolling works initially after Bluetooth pairing.
- Scrolling may stop after approximately 15–30 minutes of inactivity followed by a Bluetooth disconnect.
- Cursor movement can continue despite the scrolling failure.
- Reconnecting the mouse may not restore scrolling.
- Reinstalling the driver or pairing the mouse again may temporarily restore functionality.

The two documented approaches address the problem differently.

Driver 1 attempts to preserve Apple's existing scrolling path. Driver 2 implements its own touch-to-scroll processing.

## Supported Hardware

| Magic Mouse Model | Year | Bluetooth PID | Driver 1 | Driver 2 |
|---|---|---|---|---|
| Magic Mouse v1 | 2009 | `0x030D` | Supported by the documented approach | Not supported |
| Magic Mouse v2 | 2015 | `0x0269` | Supported by the documented approach | Not supported |
| Magic Mouse v2 (alternate PID) | 2015 | `0x0310` | Supported by the documented approach | Not supported |
| Magic Mouse v3 | 2024 | `0x0323` | Supported by the documented approach | Supported by the documented implementation |

Driver 2 is intended for the v3 device identified by PID `0x0323`. Do not assume it supports older Magic Mouse models.

### How to Verify Your Mouse PID

1. Open Device Manager.
2. Locate the Apple Magic Mouse under Human Interface Devices or the relevant device category.
3. Open **Properties**.
4. Select the **Details** tab.
5. Choose **Hardware Ids**.
6. Look for the product ID, such as `PID&0323`.

Confirm the hardware ID before installing either driver.

## System Requirements

Common requirements:

- Windows 10 build 14393 or later, or Windows 11.
- An Apple Magic Mouse paired over Bluetooth.
- Administrator access.
- Permission to restart Windows.
- A backup and recovery plan before changing kernel-driver configuration.

### Driver 1 Requirements

- The appropriate Apple filter driver binary.
- The correct device instance ID.
- Permission to modify the device's `LowerFilters` registry value.
- A reboot after installation.

The installer may encounter access restrictions when writing to protected Plug and Play registry keys. Follow the project's installation documentation if this occurs.

### Driver 2 Requirements

- The appropriate KMDF driver source or package.
- The Windows Driver Kit or EWDK if building from source.
- A local development signing certificate.
- Windows Test Mode for the community-signed driver.
- Secure Boot and memory integrity may need to be disabled for the development configuration.

Disabling Windows security features reduces protection. Understand the implications before proceeding, and restore your normal security configuration when no longer testing the driver.

## Driver 1 — Apple Filter Driver

**Driver file:** `applewirelessmouse.sys`

This approach uses Apple's existing Boot Camp mouse filter and registers it as a lower filter on the relevant Bluetooth HID device.

The binary is intended to remain unmodified. Instead of patching Apple's executable, the installer changes how Windows registers and loads the driver.

### How It Works

The driver is inserted into the relevant HID device stack to help preserve the multitouch processing path.

The documented investigation describes a failure mode in which Windows device-property synchronization and Bluetooth service-cache changes are associated with a transition from separate HID collections to a configuration where scrolling no longer works correctly.

The Apple filter approach attempts to prevent this failure from disrupting the existing scrolling path.

### Main Characteristics

- Uses Apple's existing driver binary.
- Does not require compiling a new driver.
- Retains Apple's original scroll behaviour.
- Requires device-specific registry configuration.
- May need to be reinstalled or rebound if the device instance ID changes after re-pairing.
- Requires verification on the target Windows installation.

### Installation Documentation

See:

- [Driver 1 README](v1-binary-patch/README.md)
- [Installation script](v1-binary-patch/installer/Install-MagicMousePatch.ps1)
- [Uninstallation script](v1-binary-patch/installer/Uninstall-MagicMousePatch.ps1)

### Verify the Driver Binary

If the repository includes the expected Apple driver file, verify its integrity before installing it.

```powershell
$sys = ".\v1-binary-patch\apple-driver\applewirelessmouse.sys"

# Calculate the SHA-256 hash
(Get-FileHash $sys -Algorithm SHA256).Hash

# Check the Authenticode signature
(Get-AuthenticodeSignature $sys).Status

# Display the signing certificate subject
(Get-AuthenticodeSignature $sys).SignerCertificate.Subject
