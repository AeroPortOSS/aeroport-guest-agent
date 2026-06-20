# aeroport-guest-agent

This repository contains the Windows guest subsystem for the AeroPort platform. It includes the binary execution layer, user-space interception utilities, and display driver modules. All components compile natively into ARM64 binaries inside a single Microsoft Visual Studio Solution.

## Repository Architecture

* **src/shared/**: Shared C/C++ header protocols (`protocol.h`) defining VM synchronization primitives, VSOCK message formats, and shared memory structures used by both the guest and macOS host applications.
* **src/aeroport-launch/**: A lightweight Win32 application launcher that verifies application Portable Executable (PE) headers, initializes target processes in a suspended configuration, and performs remote thread execution injection.
* **src/aeroport-hook/**: The core runtime interception engine built on Microsoft Detours. It injects directly into targeted x64/ARM64 processes to hook Direct3D 12 and DXGI swapchain functions, routing graphic state tracking parameters straight to memory-mapped channels.
* **src/aeroport-idd-driver/**: A Windows User-Mode Driver Framework (UMDF 2.x) Indirect Display Driver. It creates virtual headless monitor layouts inside the guest OS, enabling native HiDPI retina scaling matching the host Mac displays.

## Build Requirements

* Windows 11 ARM64 development instance
* Microsoft Visual Studio 2022 (with "Desktop development with C++" enabled)
* Windows 11 SDK (10.0.22621.0 or higher)
* Windows Driver Kit (WDK) configured for UMDF 2.x pipelines

## Compilation Steps

1. Launch Visual Studio 2022.
2. Open the file `aeroport-windows-core.sln`.
3. Set the active Solution Configuration dropdown to `Release` and the Solution Platform to `ARM64`.
4. Run `Build -> Build Solution` from the main menu toolbar.
5. All compiled assets (`aeroport-launch.exe`, `aeroport-hook.dll`, and the `aeroport-idd-driver` package) will be outputted to the `ARM64/Release/` project directory.

## Testing Driver Packages Manually

To bypass signature verification when testing the display driver locally in a development container:
1. Open an elevated Command Prompt.
2. Run the following command:
   ```cmd
   bcdedit /set testsigning on
