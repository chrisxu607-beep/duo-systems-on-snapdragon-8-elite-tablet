# why I am doing this

In early 2026, we've seen the price of SSDs and RAM rocketed up, and the cost of purchasing a new laptop for university is now being too much for many students. This project is an attempt to explore whether a modern mobile SoC can be used as a general-purpose Linux computer, and whether it can be used for CS undergraduate study. 

# Duo Systems on Snapdragon 8 Elite Tablet

A dual-system computing platform built on a Lenovo Y900 tablet with a Qualcomm Snapdragon 8 Elite SoC, combining Android with a full ARM64 Ubuntu Linux environment.

The project explores how far a modern mobile SoC can be used as a general-purpose Linux computer, including desktop Linux, GPU-accelerated computing, Vulkan graphics, native ARM64 development, and local AI inference.

## Overview

The system uses a Lenovo tablet (Y900) based on the **Qualcomm Snapdragon 8 Elite** platform.

Instead of treating the tablet purely as an Android device, the project creates a second Linux environment running alongside Android:

```text
┌─────────────────────────────────────────────┐
│                 Android                     │
│                                             │
│  Android applications                       │
│  Android kernel / hardware interfaces       │
│  Qualcomm device drivers                    │
└──────────────────────┬──────────────────────┘
                       │
                    Termux
                       │
        ┌──────────────┴──────────────┐
        │                             │
   Termux:X11                    Ubuntu 24.04
        │                         ARM64 rootfs
        │                             │
        │                    ┌────────┴────────┐
        │                    │                 │
        │                  XFCE          Development
        │                                    │
        │                              Vulkan / Turnip
        │                                    │
        └─────────────── X11 ────────────────┘
                                             │
                                         applications (codium, llama.cpp, etc.)
```

The Linux environment runs on the same Snapdragon 8 Elite hardware while retaining access to the device's CPU and GPU capabilities.

## System Architecture

```text
┌───────────────────────────────────────────────┐
│                  Android                      │
│                                               │
│  Android userspace                            │
│  Qualcomm hardware / kernel interfaces        │
│                                               │
└───────────────────────┬───────────────────────┘
                        │
                     Termux
                        │
                Ubuntu ARM64 rootfs
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      XFCE          Development       Vulkan
        │               │                │
   Termux:X11      GCC / Python       Turnip
                        │                │
                    code-server      Adreno 830
                        │
                     llama.cpp
```

## Hardware

- **Device:** Lenovo Y900 tablet
- **SoC:** Qualcomm Snapdragon 8 Elite
- **CPU:** Qualcomm Snapdragon 8 Elite
- **GPU:** Qualcomm Adreno 830
- **Memory:** 16 GB RAM
- **Storage:** 512 GB
- **Operating systems:**
  - Android
  - Ubuntu 24.04 ARM64

## Linux Environment

The Ubuntu environment is launched from Android using Termux.

### Software stack

```text
Android
└── Termux
    ├── Termux:X11
    └── Ubuntu 24.04 ARM64
        ├── XFCE
        ├── Mesa
        ├── Turnip
        ├── Vulkan
        └── llama.cpp
```

The system uses an Ubuntu ARM64 root filesystem launched from Termux, with selected Android kernel interfaces and devices exposed to the Linux environment.

## Desktop Environment

XFCE is used as the Linux desktop environment.

The system runs:

- XFCE4
- XFCE Window Manager (`xfwm4`)
- Termux:X11
- D-Bus
- SSH
- code-server

XFWM4 compositing is disabled because the Vulkan/Zink compositor path caused X11 swapchain initialization failures on this configuration.

The desktop therefore uses:

```text
XFCE
  ↓
xfwm4
  ↓
Compositor disabled
  ↓
Termux:X11
```

while Vulkan remains available to applications independently of the desktop compositor.

## GPU Acceleration

The Adreno 830 is exposed to the Linux environment through Qualcomm's KGSL kernel interface.

Mesa's **Turnip** Vulkan driver provides native Vulkan support for the Adreno GPU.

The working configuration is:

```text
GPU:     Adreno (TM) 830v1
Vulkan:  1.4.362
Driver:  Mesa Turnip
Mesa:    26.3.0-devel
```

The Vulkan device can be verified with:

```bash
vulkaninfo --summary
```

Expected output includes:

```text
deviceName = Adreno (TM) 830v1
driverName = turnip Mesa driver
```

## Vulkan Memory

Because the Snapdragon platform uses unified memory, the GPU does not have conventional dedicated VRAM.

Vulkan exposes approximately:

```text
Device-local heap:
~11.26 GiB
```

The actual available budget is dynamic because the GPU shares system memory with Android/Linux and other applications.

This makes the tablet capable of running substantially larger GPU-accelerated workloads than a traditional mobile GPU with a small dedicated-memory allocation might suggest.

## Development Environment

The Linux environment provides a conventional ARM64 development environment including:

- GCC / G++
- CMake
- Git
- Python
- Conda
- VS Code / code-server
- SSH
- Vulkan development tools
- llama.cpp

This makes the tablet usable as a portable ARM64 development workstation rather than only as a mobile device.

## Applications

The platform is used for several experiments and projects, including:

- Local LLM inference
- Vulkan GPU computing
- ARM64 Linux development
- Mobile GPU experimentation
- Desktop Linux on mobile hardware
- llama.cpp acceleration
- Model performance benchmarking

## Relationship to Local LLM Project

This repository documents the **underlying hardware and Linux platform**.

The local AI inference work is documented separately:

**[`local-llm-on-phone-chip`](../local-llm-on-phone-chip)**

That repository focuses specifically on running quantized LLMs on the Snapdragon 8 Elite / Adreno 830 platform.

## Key Results

The project successfully achieved:

- ARM64 Ubuntu 24.04 running on Snapdragon 8 Elite
- XFCE desktop through Termux:X11
- Adreno 830 GPU detection from Linux
- Vulkan 1.4 support through Mesa Turnip
- llama.cpp Vulkan device detection
- GPU-accelerated local LLM inference
- Development environment suitable for native ARM64 software development

## Project Goals

Future experiments may include:

- Vulkan kernel optimization
- Larger local language models
- MTP / speculative decoding
- ARM64 compiler optimization
- GPU performance benchmarking
- CPU/GPU workload partitioning
- Additional Linux desktop workloads
- Qualcomm-specific acceleration paths

## Technologies

- Qualcomm Snapdragon 8 Elite
- Adreno 830
- ARM64
- Android
- Termux
- Termux:X11
- Ubuntu 24.04
- XFCE
- Mesa
- Turnip
- Vulkan
- llama.cpp
- C/C++
- Python
- Linux

## Disclaimer

This project is an experimental Linux environment running on mobile hardware. Some components depend on device-specific kernel interfaces, Mesa development drivers, and Android/Termux integration.

The configuration may therefore require modification for other Snapdragon devices or Android versions.