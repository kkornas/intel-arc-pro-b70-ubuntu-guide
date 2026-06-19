
# Intel Arc B70 OpenCL Level Zero Setup Guide

>**Important:** Before proceeding, check the [README.md] file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide

This guide explains how to setup  ** OpenCL & Level Zero  **.  This is the necessary configuration for the Ollama workload [docs/workloads/ollama-dockered-oneapi.md](docs/workloads/ollama-dockerized-oneapi.md)


## Prerequisites

Ensure your system has:
- Intel Arc GPU installed & detected
	- Check PCIe Devices for e.g. "Battlemage G31"
	- Check Kernel module for "i915|xe"
- Optional: Basic Funcion, Desktop mode via DisplayPort

Arc Driver Guide: [01-base-driver-setup.md]

Check:
```bash
lspci -v | grep -i VGA
lsmod | grep -E "i915|xe"
```


## OpenCL & Level Zero Stack

In this Guide you will install ( ✔ ) :

```text
Applications
├── Ollama / AI
│   ├── OpenCL Runtime
│   │   ├── intel-opencl-icd  					✔
│   │   └── ocl-icd-libopencl1  				✔
│   │
│   └── Level Zero Runtime (Compute API)
│       ├── libze-intel-gpu1  					✔
│       └── libze1  							✔
│
├── Games / Rendering
│   ├── OpenGL
│   │   ├── mesa-utils  						✔
│   │   └── mesa-utils-bin  					✔
│   │
│   └── Vulkan (Verification only)
│       └── vulkan-tools   						-> opional
│
└── Video / Media Acceleration
    ├── VA-API Driver 						
    │   └── intel-media-va-driver-non-free  	✔					
    │
    └── VA-API Verification
        └── vainfo  							✔
		
		
Developer / Tooling Layer (Intel oneAPI)
├── intel-oneapi-base-toolkit 					✔
│   ├── sycl-ls (device enumeration over OpenCL + Level Zero)
│   └── oneAPI developer tooling (compiler, runtime utilities)

Monitoring / Diagnostics Tools
├── clinfo  									✔
│   └── OpenCL platform & device inspection
│
├── intel-gpu-tools  							✔
│   └── low-level Intel GPU engine / counters
│
└── nvtop  										✔
    └── real-time GPU utilization + memory tracking


Userspace Drivers / Runtime Libraries
├── Mesa Stack 									✔
│   ├── mesa-utils 
│   ├── mesa-utils-bin 
│   └── Vulkan/OpenGL userspace drivers (Mesa ANV / Intel driver stack)
│
├── Intel Compute Runtime
│   ├── intel-opencl-icd  						✔
│   ├── libze-intel-gpu1  						✔
│   └── libze1  								✔
│
└── VA-API Stack
    └── intel-media-va-driver-non-free 
				↓
Kernel Driver Layer
├── i915
└── xe (next-gen Intel GPU driver path)
                ↓
Intel Arc GPU (BMG G31 / Pro B70)
```

## 1. Add Repository 
	
```bash
sudo add-apt-repository ppa:kobuk-team/intel-graphics  # falls empfohlen
sudo apt update
```

## 2. Install Main Packages
	
```bash
sudo apt update
sudo apt install -y libze-intel-gpu1 libze1 intel-opencl-icd clinfo
```


## 3. Monitoring and Debug

Install following tools to Verify the Installation of OpenCL + Level Zero

```bash
sudo apt update && sudo apt install -y mesa-utils mesa-utils-bin ocl-icd-libopencl1 intel-media-va-driver-non-free intel-gpu-tools vainfo nvtop
```


### 3.1 Check PCIe Devices (Arc detected?)

```bash
lspci -v | grep -i VGA
```

**Expected Output:**

```
00:02.0 VGA compatible controller: Intel Corporation RocketLake-S GT1 [UHD Graphics 750] (rev 04) (prog-if 00 [VGA controller])
04:00.0 VGA compatible controller: Intel Corporation Battlemage G31 [Intel Graphics] (prog-if 00 [VGA controller])
```

>*Note: This output shows both the integrated Intel UHD Graphics and the Intel Arc Battlemage G31.  Yours may vary.*

### 3.2 Check Kernel module `i915` is loaded

```bash
lsmod | grep -E "i915|xe"
```
* Depending on kernel version, Intel GPUs may use either i915 or xe.

**Expected Output:**

```
xe                   3948544  59
i915                 4931584  4
gpu_sched              65536  1 xe
drm_gpuvm              49152  1 xe
drm_gpusvm_helper      36864  1 xe
drm_ttm_helper         16384  1 xe
drm_buddy              28672  2 xe,i915
drm_exec               12288  2 drm_gpuvm,xe
ttm                   131072  3 drm_ttm_helper,xe,i915
drm_suballoc_helper    24576  1 xe
drm_display_helper    294912  2 xe,i915
cec                   106496  3 drm_display_helper,xe,i915
i2c_algo_bit           16384  2 xe,i915
intel_vsec             24576  3 intel_pmc_ssram_telemetry,pmt_telemetry,xe
video                  77824  3 dell_wmi,xe,i915
```


### 3.3 Check OpenCL platform   (Compute-Test)

```bash
clinfo | grep -E "Device Name|Driver Version"
```

**Expected Output:**
```
Device Name                                     Intel(R) Arc(TM) Pro B70 Graphics
  Driver Version                                  26.18.38308.1
  Device Name                                     Intel(R) UHD Graphics 750
  Driver Version                                  26.18.38308.1
    Device Name                                   Intel(R) Arc(TM) Pro B70 Graphics
    Device Name                                   Intel(R) Arc(TM) Pro B70 Graphics
    Device Name                                   Intel(R) Arc(TM) Pro B70 Graphics
```

Or

**Expected Output (older Driver Version  25.31.034666 when i installed it):**
```
Device Name 									Intel(R) Graphics [0xe223] 
  Driver Version                                  25.31.034666 
  Device Name                                     Intel(R) UHD Graphics 750 
  Driver Version 								  25.31.034666 
    Device Name                                    Intel(R) Graphics [0xe223] 
    Device Name                                    Intel(R) Graphics [0xe223] 
    Device Name                                    Intel(R) Graphics [0xe223]
```

**Note**:  [0xe223] the card’s ID -> important identifier to detect the Arc in various outputs

### 3.4 Verify Level Zero Runtime Installation

#### a) check libze installation
```bash
ls /usr/lib/x86_64-linux-gnu/libze*
```

**Expected Output**

```
/usr/lib/x86_64-linux-gnu/libze_intel_gpu.so.1           /usr/lib/x86_64-linux-gnu/libze_loader.so.1.28.2         /usr/lib/x86_64-linux-gnu/libze_validation_layer.so
/usr/lib/x86_64-linux-gnu/libze_intel_gpu.so.1.15.38308  /usr/lib/x86_64-linux-gnu/libze_tracing_layer.so         /usr/lib/x86_64-linux-gnu/libze_validation_layer.so.1
/usr/lib/x86_64-linux-gnu/libze_loader.so                /usr/lib/x86_64-linux-gnu/libze_tracing_layer.so.1       /usr/lib/x86_64-linux-gnu/libze_validation_layer.so.1.28.2
/usr/lib/x86_64-linux-gnu/libze_loader.so.1              /usr/lib/x86_64-linux-gnu/libze_tracing_layer.so.1.28.2
```

#### b) check dpkg installation
```bash
dpkg -l | grep libze
```

**Expected Output**

```
ii  libze-dev:amd64                                          1.28.2-1~25.10~ppa1                          amd64        oneAPI Level Zero -- development files
ii  libze-intel-gpu1                                         26.18.38308.1-1~25.10~ppa1                   amd64        Intel oneAPI L0 support implementation for Intel GPUs -- shared library
ii  libze1:amd64                                             1.28.2-1~25.10~ppa1                          amd64        oneAPI Level Zero -- share libraries
```

### 3.4 Verfiy Level Zero with SYCL (final compute test)

>*Note* Level Zero runtime can be verified synthetically via sycl-ls (if intel-oneapi-base-toolkit installed) or functionally via a real workload such as Ollama (Docker-based) 
If you struggle to install intel-oneapi-base-toolkit, continue with workload setup [docs/workloads/ollama-dockered-oneapi.md](docs/workloads/02-ollama-dockered-oneapi.md). 

The intel-oneapi-base-toolkit Requires a 2GB download spcae + tools: cmake, pkg-config, and build-essential.

>*Note* Many guides incorrectly instruct you to install intel-oneapi-base-toolkit via apt -> in my case this package did not exist

### a) install build tools

```bash
sudo apt update
sudo apt -y install cmake pkg-config build-essential

```
### b) Download intel-oneapi-base-toolkit

*Check latestes Version here https://www.intel.com/content/www/us/en/developer/tools/oneapi/oneapi-toolkit-download.html?packages=oneapi-toolkit&oneapi-toolkit-os=linux&oneapi-lin=offline*

```bash
wget https://registrationcenter-download.intel.com/akdlm/IRC_NAS/71180075-e4e3-4c6f-bbbb-19017ed0cf7d/intel-oneapi-toolkit-2026.0.0.198_offline.sh
sudo sh ./intel-oneapi-toolkit-2026.0.0.198_offline.sh -a --silent --cli --eula accept
```

### c) Install intel-oneapi-base-toolkit

```bash
sudo sh ./intel-oneapi-toolkit-2026.0.0.198_offline.sh -a --silent --cli --eula accept
```

**Expected Output**
```
Extract intel-oneapi-toolkit-2026.0.0.198_offline to /home/milla/intel-oneapi-toolkit-2026.0.0.198_offline...
[##########################################################################################################################################################################################]
Extract intel-oneapi-toolkit-2026.0.0.198_offline completed!
Checking system requirements...
Done.
Wait while the installer is preparing...
Done.
Launching the installer...
This machine uses operating system "Ubuntu version 25.10". Compatibility issues may occur.
Installation can continue; however, product functionality may not meet expectations because this product is untested on this operating system. Suggestion: Check the Release Notes for a list of supported operating systems and install this product on a compliant system.
Start installation flow...
Installed Location: /opt/intel/oneapi
Log files: /tmp/root/intel_oneapi_installer/2026.06.16.07.32.31.627
Installation has successfully completed
Remove extracted files: /home/milla/intel-oneapi-toolkit-2026.0.0.198_offline...
```

### d) Source

```bash
which cmake pkg-config make gcc g++  #verify the installation location,
source /opt/intel/oneapi/setvars.sh
```

**Expected Output (source)**
```
:: initializing oneAPI environment ...
   -bash: BASH_VERSION = 5.2.37(1)-release
   args: Using "$@" for setvars.sh arguments:
:: ccl -- latest
:: compiler -- latest
:: debugger -- latest
:: dev-utilities -- latest
:: dnnl -- latest
:: dpl -- latest
:: ipp -- latest
:: ippcp -- latest
:: mkl -- latest
:: mpi -- latest
:: tbb -- latest
:: tcm -- latest
:: umf -- latest
:: vtune -- latest
:: oneAPI environment initialized ::
```
### e) Run SYCL Command

```bash
sycl-ls
```

**Expected Output**
```bash
[level_zero:gpu][level_zero:0] Intel(R) oneAPI Unified Runtime over Level-Zero V2, Intel(R) Arc(TM) Pro B70 Graphics 20.2.0 [1.15.38308+1]
[level_zero:gpu][level_zero:1] Intel(R) oneAPI Unified Runtime over Level-Zero V2, Intel(R) UHD Graphics 750 12.1.0 [1.15.38308+1]
[opencl:cpu][opencl:0] Intel(R) OpenCL, 11th Gen Intel(R) Core(TM) i7-11700 @ 2.50GHz OpenCL 3.0 (Build 0) [2026.21.3.0.31_160000]
[opencl:gpu][opencl:1] Intel(R) OpenCL Graphics, Intel(R) Arc(TM) Pro B70 Graphics OpenCL 3.0 NEO  [26.18.38308.1]
[opencl:gpu][opencl:2] Intel(R) OpenCL Graphics, Intel(R) UHD Graphics 750 OpenCL 3.0 NEO  [26.18.38308.1]
```


### 3.5 Verfiy Level Zero with Ollama (dockered)

-> [docs/workloads/ollama-dockered-oneapi.md](docs/workloads/02-ollama-dockered-oneapi.md).

> *Note* Docker Images like the official intel image: "intelanalytics/ipex-llm-inference-cpp-xpu:latest" are delivered with SYCL / oneAPI component -> if the docker workload works native sycl-ls installation is not requiered anymore.


## 4. Package Overview 

## OpenCL + Level Zero (Compute / AI)

| Layer                             | Package              |            Installed Version | Purpose                     |
| --------------------------------- | -------------------- | ---------------------------: | --------------------------- |
| OpenCL Runtime                    | `intel-opencl-icd`   | `26.18.38308.1-1~25.10~ppa1` | Intel OpenCL GPU runtime    |
| OpenCL Loader                     | `ocl-icd-libopencl1` |                    `2.3.3-1` | OpenCL ICD loader           |
| Level Zero GPU Runtime            | `libze-intel-gpu1`   | `26.18.38308.1-1~25.10~ppa1` | Intel GPU backend           |
| Level Zero Loader                 | `libze1`             |        `1.28.2-1~25.10~ppa1` | oneAPI Level Zero loader    |
| Level Zero Dev Files              | `libze-dev`          |        `1.28.2-1~25.10~ppa1` | Development headers / debug |
| OpenCL Diagnostic Tool            | `clinfo`             |             `3.0.25.02.14-1` | Verify OpenCL devices       |

👉 **Additional Repository required**:
```bash id="repo"
ppa:kobuk-team/intel-graphics
```
---

## Graphics / Rendering

| Layer                            | Package          |   Installed Version | Purpose                   |
| -------------------------------- | ---------------- | ------------------: | ------------------------- |
| OpenGL Tools                     | `mesa-utils`     |           `9.0.0-2` | `glxinfo`, `glxgears`     |
| OpenGL Helper Binaries           | `mesa-utils-bin` |           `9.0.0-2` | additional Mesa utilities |
| Vulkan Verification *(optional)* | `vulkan-tools`   | `1.4.304.0+dfsg1-1` | Vulkan diagnostics (optional) `vulkaninfo`, `vkcube`    |

---

## Video Acceleration (Media)

| Layer                  | Package                          |     Installed Version | Purpose                           |
| ---------------------- | -------------------------------- | --------------------: | --------------------------------- |
| VA-API Driver          | `intel-media-va-driver-non-free` | `26.2.0-1~25.10~ppa1` | Intel hardware video acceleration |
| Video API Verification | `vainfo`                         | `2.23.0-1~25.10~ppa4` | Verify VA-API support             |

---

## Monitoring / Debugging

| Layer                     | Package           | Installed Version | Purpose                     |
| ------------------------- | ----------------- | ----------------: | --------------------------- |
| Intel GPU Debug Utilities | `clinfo`             |           `3.0.25.02.14-1` | OpenCL device enumeration |
| Intel GPU Debug Tools | `intel-gpu-tools` |           `2.0-1` | Low-level GPU metrics |
| GPU Monitoring            | `nvtop`           |         `3.2.0-1` | Real-time GPU utilization tracking |

---

## Developer / Tooling Layer (Intel oneAPI)
| Layer                     | Package           | Installed Version | Purpose                     |
| ------------------------- | ----------------- | ----------------: | --------------------------- |
| oneAPI Base Toolkit (sycl-ls)     | offline installer    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |
| SYCL Device Inspector     | oneAPI Base Toolkit (sycl-ls)    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |

> oneAPI Base Toolkit is installed via offline installer (not apt). Installation steps-> [3.4 Verfiy Level Zero with SYCL (final compute test)] or https://www.intel.com/content/www/us/en/developer/tools/oneapi/oneapi-toolkit-download.html?packages=oneapi-toolkit&oneapi-toolkit-os=linux&oneapi-lin=offline*


## Kernel / Driver Layer

| Layer                                     | Component | Notes                            |
| ----------------------------------------- | --------- | -------------------------------- |
| Intel GPU Kernel Driver                   | `i915`    | current Intel DRM driver         |
| Intel GPU Kernel Driver *(newer kernels)* | `xe`      | next-generation Intel GPU driver |

---

## 5. Trouebelshooting and Monitoring

### 5.1 Check GPU passthrough und DRM-Devices 

```bash
ls /dev/dri
```
**Expected Output:**
```
by-path card1 card2 renderD128 renderD129
```

### 5.2 nvtop

Monitor GPU utilization while running a workload (e.g. Ollama model inference) ->  open in a separate shell

```bash
nvtop
```
**Expected Output:**
```
 Device 0 [Battlemage G31 (Intel Graphics)]     PCIe GEN 3@16x RX: N/A TX: N/A                 Device 1 [RocketLake-S GT1 (UHD Graphics 750)] Integrated GPU RX: N/A TX: N/A
 GPU 2800MHz MEM N/A MHz TEMP  50fC FAN  712RPM POW 217 W                                      GPU 350MHz  MEM N/A MHz TEMP N/AfC   CPU-FAN   POW N/A W
 GPU[                                       0%] MEM[||||||||||||||||||||||||30.858Gi/31.891Gi] GPU[                                       0%] MEM[                             N/A/30.487Gi]
   lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
100xGPU0 %                                                                                                                                                                                  x
   xGPU0 mem%                          lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
 75x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
 50x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
 25x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
   x                                   x                                                                                                                                                    x
  0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   m92sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq69sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq46sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq23sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq0sj
   lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
100xGPU1 %                                                                                                                                                                                  x
   xGPU1 mem%                                                                                                                                                                               x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
 75x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
 50x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
 25x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
   x                                                                                                                                                                                        x
  0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   m92sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq69sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq46sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq23sqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq0sj
    PID  USER DEV     TYPE  GPU        GPU MEM    CPU  HOST MEM Command
 ```


### 5.3 Verify Userspace Drivers (Mesa + Intel libs) via vulkan-tools 

Vulkan tools help to verify Userspace Drivers (Mesa + Intel libs). Yes it will install the Vulkan driver described here [docs/03-vulkan-stack.md]  but not the Vulkan driver!

```bash
sudo apt install vulkan-tools
vulkaninfo | grep deviceName
```

If llvmpipe appears, Vulkan is not using Intel GPU acceleration. (llvmpipe = CPU fallback)

**Expected Output:**

```
WARNING: [Loader Message] Code 0 : ICD for selected physical device does not export vkGetPhysicalDeviceDisplayPlanePropertiesKHR!
WARNING: [Loader Message] Code 0 : ICD for selected physical device does not export vkGetPhysicalDeviceDisplayPropertiesKHR!
        deviceName        = Intel(R) Graphics (BMG G31)
        deviceName        = Intel(R) Graphics (RKL GT1)
        deviceName        = llvmpipe (LLVM 20.1.8, 256 bits)
```

### 5.4 vkcube (Desktop oly)

- will render a "visual cube" -> Mesa + Intel libs test
```bash
vkcube
```

### 5.5 Verify VA-API -> Video API test

This checks if VA-API (Video Acceleration API) is working correctly.

```bash
sudo apt install vainfo
vainfo
```

**Expected Output:**

```
Trying display: wayland
libva info: VA-API version 1.23.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_23
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.23 (libva 2.23.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 26.2.0 ()
vainfo: Supported profile and entrypoints
      [...]
```

### 5.6 Check OpenGL Renderer 

This shows you which driver is currently used by your system.

```bash
sudo apt install mesa-utils
glxinfo | grep "OpenGL renderer"
```

**Expected Output (when run from a desktop session):**

```
OpenGL renderer string: Mesa Intel(R) Graphics (BMG G31)
```

**Note:**  If running this command over SSH (without a graphical desktop environment), you may get an error:

**Expected Output (when run over SSH):**

```
Error: unable to open display
```

### 5.7 Check OpenCL Number of platforms" (first Line) 

in my setup 2 iGPU UHD + Intel Arc

```bash
clinfo 
# or
clinfo | head -n 5
```

**Expected Output**

```
Number of platforms                               2
  Platform Name                                   Intel(R) OpenCL Graphics
  Platform Vendor                                 Intel(R) Corporation
  Platform Version                                OpenCL 3.0
[...]
```

## 6. Things That Didn't Work 

*   `sudo apt install intel-level-zero-gpu level-zero` 
Note: The packages intel-level-zero-gpu and level-zero are not available in official Intel repositories or other standard repositories (as of Ubuntu 25.10). Many guides incorrectly instruct you to install them using sudo apt install intel-level-zero-gpu level-zero."


*   `sudo intel_gpu_top` detects allways the iGPU -> couldn't get the Arc detected by this tool

```bash
sudo intel_gpu_top drm:/dev/dri/card1
sudo intel_gpu_top drm:/dev/dri/card0
```

-> same output from iGPU

```bash
intel-gpu-top: Intel Rocketlake (Gen12) @ /dev/dri/card1 -    0/   0 MHz;   0% RC6;        0 irqs/s
[...]
```

## 7. Comment

> Congratulations: Your system is ready to proceed with the Ollama workload [docs/workloads/ollama-dockered-oneapi.md](docs/workloads/ollama-dockerized-oneapi.md)

