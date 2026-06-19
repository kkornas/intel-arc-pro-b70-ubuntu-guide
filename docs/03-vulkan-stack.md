# Intel Arc B70 Vulkan Setup Guide

> **Important:** Before proceeding, check the [README.md] file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide

This guide explains how to setup the  ** Vulkan Driver ** for llama.cpp workload [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md])

## Prerequisites

Ensure your system has:
- Intel Arc GPU installed & detected
	- Check PCIe Devices for e.g. "Battlemage G31"
	- Check Kernel module for "i915|xe"
- Optional: Basic Funcion, Desktop mode via DisplayPort

-> [Arc Driver Guide](docs/01-base-driver-setup.md)

Check:
```bash
lspci -v | grep -i VGA
lsmod | grep -E "i915|xe"
```

## Vulkan Stack

### Architecture Overview

In this Guide you will install (✔)

```text
Application (AI / Inferenc / Game / Vulkan App)		-> [docs/workloads/llama-cpp-vulkan.md]
              ↓
libvulkan1 (Vulkan Loader, Userspace Runtime)		✔
              ↓
mesa-vulkan-drivers (Intel ANV Driver)				✔
              ↓
Kernel Driver (i915 / xe)							-> [docs/01-base-driver-setup.md]
              ↓
Intel Arc GPU
```										

### Full Stack

In this Guide you will install (✔)

```text
Applications
├── AI / Inference
│   └── llama.cpp									-> [docs/workloads/llama-cpp-vulkan.md]
│       ├── Vulkan Backend (GGML_VULKAN=ON)			
│       └── CPU Backend (fallback)					
│
├── Games / Rendering
│   ├── OpenGL
│   │   ├── mesa-utils 								✔
│   │   └── mesa-utils-bin 							✔
│   │
│   └── Vulkan
│       └── vulkan-tools 							✔
│
└── Video / Media Acceleration
    ├── VA-API Driver
    │   └── intel-media-va-driver-non-free 			✔
    │
    └── VA-API Verification
        └── vainfo 									✔


Monitoring / Diagnostics Tools
├── vulkaninfo 										✔
│   └── Vulkan device enumeration
│
├── intel-gpu-tools 								✔
│   └── Low-level Intel GPU engine counters
│
└── nvtop 											✔
    └── Real-time GPU utilization + VRAM tracking


Userspace Drivers / Runtime Libraries
├── Mesa Graphics Stack
│   ├── mesa-utils 									✔
│   ├── mesa-utils-bin 								✔
│   ├── mesa-vulkan-drivers 						✔
│   └── Intel ANV Vulkan driver 					✔
│
└── VA-API Stack
    └── intel-media-va-driver-non-free 				✔
                ↓
Kernel Driver Layer									-> [docs/01-base-driver-setup.md]
├── i915 											
└── xe (next-gen Intel GPU driver path)
                ↓
Intel Arc GPU (BMG G31 / Pro B70)

```


## 1. Install Vulkan

```bash
sudo apt update 
sudo apt install -y mesa-vulkan-drivers libvulkan-dev libvulkan1 vulkan-tools
```

## 2. Check Installation 
	
```bash
vulkaninfo | grep deviceName
```

**Expected Output:**
```
'DISPLAY' environment variable not set... skipping surface info
WARNING: [Loader Message] Code 0 : ICD for selected physical device does not export vkGetPhysicalDeviceDisplayPlanePropertiesKHR!
WARNING: [Loader Message] Code 0 : ICD for selected physical device does not export vkGetPhysicalDeviceDisplayPropertiesKHR!
        deviceName        = Intel(R) Graphics (BMG G31)
        deviceName        = Intel(R) Graphics (RKL GT1)
        deviceName        = llvmpipe (LLVM 20.1.8, 256 bits)
```

### 2.1 vkcube (Desktop oly)

> will render a "visual cube" -> Mesa + Intel libs test
```bash
vkcube
```

### 2.2 Packages installed

```bash
dpkg -l | grep -E "vulkan|mesa-vulkan"
```
**Expected Output:**
```
ii  libvulkan-dev:amd64                                      1.4.321.0-1                                  amd64        Vulkan loader library -- development files
ii  libvulkan1:amd64                                         1.4.321.0-1                                  amd64        Vulkan loader library
ii  mesa-vulkan-drivers:amd64                                25.2.8-0ubuntu0.25.10.2                      amd64        Mesa Vulkan graphics drivers
ii  vulkan-tools                                             1.4.304.0+dfsg1-1                            amd64        Miscellaneous Vulkan utilities
```

## 3. Package Overview

### Vulkan + llama.cpp (Compute / AI)

| Layer                      | Package / Component | Installed Version | Purpose |
|----------------------------|---------------------|------------------:|----------|
| Vulkan Runtime             | `libvulkan1`        | `1.4.321.0-1` | Vulkan userspace runtime |
| Vulkan Development Files   | `libvulkan-dev`     | `1.4.321.0-1` | Vulkan headers / development files |
| Mesa Vulkan Driver         | `mesa-vulkan-drivers` | `25.2.8-0ubuntu0.25.10.2` | Intel Vulkan userspace driver (Mesa ANV) |
| Vulkan Verification Tools  | `vulkan-tools` | `1.4.304.0+dfsg1-1` | Verify Vulkan support (`vulkaninfo`, `vkcube`) |
| GGUF Model Support            | built-in        | -                 | model format support |
| AI Inference Backend       | `llama.cpp` (source build) | `9436 (d6588daa8, gcc 11.4)` | LLM inference with Vulkan backend |
> llama.cpp setup here -> [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md)
---

### Graphics / Rendering

| Layer                            | Package          |   Installed Version | Purpose                   |
| -------------------------------- | ---------------- | ------------------: | ------------------------- |
| OpenGL Tools                     | `mesa-utils`     |           `9.0.0-2` | `glxinfo`, `glxgears`     |
| OpenGL Helper Binaries           | `mesa-utils-bin` |           `9.0.0-2` | additional Mesa utilities |
| Vulkan Tools (verification only) | `vulkan-tools`   | `1.4.304.0+dfsg1-1` | Vulkan diagnostics (optional) `vulkaninfo`, `vkcube`    |

---

### Video Acceleration (Media)

| Layer                  | Package                          |     Installed Version | Purpose                           |
| ---------------------- | -------------------------------- | --------------------: | --------------------------------- |
| VA-API Driver          | `intel-media-va-driver-non-free` | `26.2.0-1~25.10~ppa1` | Intel hardware video acceleration |
| Video API Verification | `vainfo`                         | `2.23.0-1~25.10~ppa4` | Verify VA-API support             |

---

### Monitoring / Diagnostics

| Layer | Package | Installed Version | Purpose |
|--------|---------|------------------:|----------|
| Vulkan Device Inspection | `vulkan-tools` | `1.4.321.0-1` | Vulkan device enumeration (`vulkaninfo`) |
| Intel GPU Debug Tools | `intel-gpu-tools` |           `2.0-1` | Low-level GPU metrics |
| GPU Monitoring            | `nvtop`           |         `3.2.0-1` | Real-time GPU utilization tracking |


---

### Developer / Tooling Layer

| Layer | Package / Component | Installed Version | Purpose |
|--------|---------------------|------------------:|----------|
| Vulkan Development Kit | `libvulkan-dev` | `1.4.321.0-1 ` | Required to compile Vulkan applications |
| llama.cpp Build System | `cmake` | `3.31.6-2ubuntu6` | Required to compile llama.cpp from source |
| llama.cpp Build System | `build-essential` | `.12.12ubuntu1` | Required to compile llama.cpp from source |

---

### Kernel / Driver Layer

| Layer                                     | Component | Notes                            |
| ----------------------------------------- | --------- | -------------------------------- |
| Intel GPU Kernel Driver                   | `i915`    | current Intel DRM driver         |
| Intel GPU Kernel Driver *(newer kernels)* | `xe`      | next-generation Intel GPU driver |

---

## Installed Versions (Commands)

Check installed package versions:

```bash
apt list --installed | grep -E "vulkan|mesa|intel-media|vainfo|intel-gpu-tools|nvtop"
```

Check Vulkan runtime:

```bash
apt policy libvulkan1 libvulkan-dev mesa-vulkan-drivers vulkan-tools
```

