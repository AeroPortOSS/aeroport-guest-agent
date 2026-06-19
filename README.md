# aeroport-guest-agent

This repository contains the consolidated source code for the AeroPort Windows guest subsystem. Operating entirely in user space via API translation and the User-Mode Driver Framework (UMDF 2.x), this module eliminates kernel-mode panics (BSODs) and isolates runtime execution directly within the targeted application threads.

---

## Workspace Architecture

The repository is organized into a single Visual Studio workspace containing three specialized modules:

### 1. `aeroport-launch`
The primary loader executable. It spawns the targeted 3D application process in a suspended state, uses Microsoft Detours to inject the translation library into its runtime footprint, and resumes execution.

### 2. `aeroport-hook`
The core runtime translation library (`.dll`). It hooks direct entry points inside `dxgi.dll` and `d3d12.dll` to manage the paravirtualized pipeline:
* **Adapter Spoofing:** Overrides DXGI Factory enumeration to report a high-performance discrete GPU, bypassing the Windows WARP software rasterization fallback.
* **Pipeline State Capture:** Intercepts full Pipeline State Objects (PSOs), bundling DXIL shader bytecode alongside Root Signatures and descriptor metadata.
* **VEH Memory Tracking:** Registers a Vectored Exception Handler utilizing `PAGE_GUARD` memory faults to trap and map inline bindless descriptor heap alterations without breaking execution loops.
* **Vectorized Delta Diffing:** Implements aligned ARM NEON assembly loops to scan and transmit sparse memory subranges (`AEROPORT_TOKEN_WRITE_SUBRANGE`) inside upload heaps right at command execution boundaries.

### 3. `aeroport-idd-driver`
A User-Mode Indirect Display Driver. It enumerates the virtual monitor interface, targets desktop swapchains, and maps guest timeline fences to host synchronization primitives via an asynchronous thread pool to prevent multi-threaded application deadlocks.

---

## Compilation and Toolchain

This workspace compiles exclusively on Windows platforms targeting the ARM64 architecture:
* **Toolchain:** Visual Studio containing the "Desktop development with C++" workload.
* **SDK / WDK:** The matching generation Windows SDK and Windows Driver Kit (WDK) with UMDF 2.x configurations enabled.
* **Dependencies:** Microsoft Detours.

*Note: The guest operating system must have test-signing enabled (`testsigned on`) via the `aeroport-iso-tool` configurations to successfully load the user-mode display driver components during development.*
