# aeroport-guest-agent

This repository contains the guest runtime components that execute inside the headless Windows 11 ARM64 virtual machine, consisting of the User-Mode Driver (`aeroport.dll`) and the background integration management daemon.

---

## Core Components

### 1. User-Mode Driver (UMD)
Functions as an Installable Client Driver (ICD) that hooks directly into user-space application layers:
* **Shader Extraction:** Intercepts Pipeline State Object (PSO) initialization to isolate raw DXIL shader bytecode tokens before hardware compilation.
* **Shadow Heap Optimization:** Maintains a local memory shadow block of active application maps. Utilizes vectorized ARM NEON memory evaluations to compare, diff, and submit sparse delta updates rather than processing full frame buffers.

### 2. VSOCK Integration Daemon
A lightweight background service that links user space environment states back to the macOS window server over hypervisor sockets:
* **Shell Integration:** Monitors the guest Start Menu registry paths, extracts native application shortcuts and application `.png` visual icons, and serializes them to the host.
* **RAIL Protocol UI Routing:** Passes window boundaries, focal states, and coordinate changes over the local hypervisor socket layers.

---

## Development and Testing

The agent architecture can be compiled using modern C++ toolchains on Windows. For sandbox verification, communication structures can be tested locally using internal TCP loopback sockets (`127.0.0.1`) before switching to native `AF_HYPERV` / `VMADDR_CID_HOST` socket types on actual Apple Silicon hypervisor setups.
