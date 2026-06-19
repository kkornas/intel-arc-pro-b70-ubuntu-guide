# Intel Arc Pro B70 on Ubuntu

The Intel Arc Pro graphics card offers incredible potential on Linux, but the initial software setup can be complex. This guide is designed to simplify that process and help you unlock the full potential of your Arc. 

It's also a living document, continuously updated with new information, workloads, and contributions from the community – a resource for anyone looking to get their Arc up and running smoothly.

### Note
> **This guide reflects a real-world working configuration. Different CPUs, motherboards, power supplies, Ubuntu versions, kernels, and Mesa versions may behave differently.**

> **Intel Arc support on Linux evolves rapidly, so newer kernels and Mesa releases may improve stability and performance.**

## Content Overview 

- Intel Arc Pro B70 driver installation
- Ubuntu kernel and Mesa requirements
- Intel oneAPI / SYCL setup
- Vulkan compute configuration
- Validation and diagnostics
- installation and setup of workloads llama-cpp, ollama, (openclaw, n8n, comfyui, whisper follwoing)

## Table of Contents README.md
1.  **Introduction**
2.  **Quick Start / Where to Start**
    -   Step 1. Intel Arc B70 Base Driver Setup Guide
    -   Step 2. Driver & Workload
        -   OpenCL + Level Zero + Ollama (Dockerized)
        -   Vulkan + llama.cpp
3.  **Tested Hardware**
4.  **Operating System**
5.  **Package Overview**
    -   OpenCL + Level Zero (Compute / AI)
    -   Graphics / Rendering
    -   Video Acceleration (Media)
    -   Monitoring / Debugging
    -   LLM Runtime (llama.cpp)
    -   Developer / Tooling Layer (Intel oneAPI)
6.  **Final Stack**
7.  **General Tips**
    -   Hardware Setup
    -   Disk Space
    -   Driver and Workload Options
8.  **Use Cases**
    -   Vulkan
    -   oneAPI / SYCL
9.  **Coming Soon**
10.  **Contributing**
11.  **License**
 
## 1. Quick Start / Where to Start

### Step 1. Intel Arc B70 Base Driver Setup Guide

-> (/docs/01-base-driver-setup.md)
 
### Step 2. Driver & Workload 
Decide between 
``OpenCL + Level Zero + Ollama (Dockerized)``
or 
``Vulkan + llama.cpp`` 
or
``both`` 
> check out chapter: 5. General Tips - Driver and Workload Options

#### a) OpenCL + Level Zero + ollama dockerized
Install Driver 	  		-> [Intel Arc B70 OpenCL Level Zero Setup Guide](docs/02-aneapi-stack.md)
Install Workload	-> [Intel Arc B70 Ollama (Dockerized) Oneapi Guide](docs/workloads/ollama-dockerized-oneapi.md)

 
#### b) Vulkan + llama.cpp
Install Driver 			->  [Intel Arc B70 Vulkan Setup Guide](docs/03-vulkan-stack.md)
Install Workload	-> [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md)


## 2. Tested Hardware

| Component   | Specification                   |
| ----------- | ------------------------------- |
| GPU         | Intel Arc Pro B70 32GB VRAM     |
| CPU         | Intel i7-11700                  |
| RAM         | 32 GB DDR4                      |
| Motherboard | Dell XPS 8940 Mainboard         |
| PSU         | 500W Dell XPS 8940 Power Supply (Platinum)|
| Case        | Dell XPS 8940 Case              |
| Storage     | NVMe SSD M2 Slot                |
| Cooling     | Stock Dell cooling              |

>Basically it's a Dell XPS 8940 + Arc B70


## 3. Operating System

| Component       | Version           |
| --------------- | ----------------- |
| OS              | Ubuntu 25.10 LTS  |
| Kernel          | Linux 6.x 6.17.0-35-generic        |

## 4. Package Overview 

Following Tables will show you the **complete Stack** for ``text Vulkan + llama.cpp`` & ``OpenCL + Level Zero + ollama dockerized``

### OpenCL + Level Zero (Compute / AI)

| Layer                             | Package              |            Installed Version | Purpose                     |
| --------------------------------- | -------------------- | ---------------------------: | --------------------------- |
| OpenCL Runtime                    | `intel-opencl-icd`   | `26.18.38308.1-1~25.10~ppa1` | Intel OpenCL GPU runtime    |
| OpenCL Loader                     | `ocl-icd-libopencl1` |                    `2.3.3-1` | OpenCL ICD loader           |
| Level Zero GPU Runtime            | `libze-intel-gpu1`   | `26.18.38308.1-1~25.10~ppa1` | Intel GPU backend           |
| Level Zero Loader                 | `libze1`             |        `1.28.2-1~25.10~ppa1` | oneAPI Level Zero loader    |
| Level Zero Dev Files              | `libze-dev`          |        `1.28.2-1~25.10~ppa1` | Development headers / debug |
| OpenCL Diagnostic Tool            | `clinfo`             |             `3.0.25.02.14-1` | Verify OpenCL devices       |

 **Additional Repository required**:
```bash id="repo"
ppa:kobuk-team/intel-graphics
```
---

### Graphics / Rendering

| Layer                            | Package            | Installed Version | Purpose |
| -------------------------------- | ------------------ | ----------------: | -------- |
| OpenGL Tools                     | `mesa-utils`      | `9.0.0-2`         | `glxinfo`, `glxgears` |
| OpenGL Helper Binaries           | `mesa-utils-bin`  | `9.0.0-2`         | additional Mesa utilities |
| Vulkan Tools (verification only) | `vulkan-tools`    | `1.4.304.0+dfsg1-1` | `vulkaninfo`, `vkcube` | 
| Vulkan Loader (Mesa / system)    | `libvulkan1`      | `1.4.321.0-1`   | Vulkan runtime loader |
| Intel Vulkan Driver (Mesa ANV)   | `mesa-vulkan-drivers` | `25.2.8-0ubuntu0.25.10.2` | Intel Arc Vulkan backend |
---

### Video Acceleration (Media)

| Layer                  | Package                          |     Installed Version | Purpose                           |
| ---------------------- | -------------------------------- | --------------------: | --------------------------------- |
| VA-API Driver          | `intel-media-va-driver-non-free` | `26.2.0-1~25.10~ppa1` | Intel hardware video acceleration |
| Video API Verification | `vainfo`                         | `2.23.0-1~25.10~ppa4` | Verify VA-API support             |
---

### Monitoring / Debugging

| Layer                     | Package           | Installed Version | Purpose                     |
| ------------------------- | ----------------- | ----------------: | --------------------------- |
| Intel GPU Debug Utilities | `clinfo`             |           `3.0.25.02.14-1` | OpenCL device enumeration |
| Intel GPU Debug Tools | `intel-gpu-tools` |           `2.0-1` | Low-level GPU metrics |
| GPU Monitoring            | `nvtop`           |         `3.2.0-1` | Real-time GPU utilization tracking |
| Vulkan Device Inspection | `vulkan-tools` | `1.4.321.0-1` | Vulkan device enumeration (`vulkaninfo`) |
---

### LLM Runtime (llama.cpp)

| Layer                          | Component        | Installed Version | Purpose |
| ----------------------------- | ---------------- | ----------------: | -------- |
| llama.cpp (source build)     | compiled from source | `9436 (d6588daa8, gcc 11.4)` | local LLM inference engine |
| GGUF Model Support            | built-in        | -                 | model format support |
| Vulkan Backend (optional)     | compile flag     | enabled/disabled  | GPU acceleration via Vulkan |
| OpenCL Backend (optional)     | compile flag     | enabled/disabled  | Intel GPU compute backend |
>  llama.cpp is compiled from source.  Installation steps described in [docs/llama-cpp-vulkan.md] 
---
### Developer / Tooling Layer (Intel oneAPI)
| Layer                     | Package           | Installed Version | Purpose                     |
| ------------------------- | ----------------- | ----------------: | --------------------------- |
| oneAPI Base Toolkit (sycl-ls)*    | offline installer    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |
| SYCL Device Inspector     | oneAPI Base Toolkit (sycl-ls)    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |

> oneAPI Base Toolkit is installed via offline installer (not apt). Installation steps described in  [docs/02-opencl-level-zero.md]>  *3.4 Verify Level Zero with SYCL (final compute test)*

##  5. Final Stack

Following Tables will show you the **complete Stack** for ``Vulkan + llama.cpp`` & ``OpenCL + Level Zero + ollama dockerized``

```text
Applications
├── Ollama / AI (Host or Docker Workload)
│   ├── OpenCL Runtime
│   │   ├── intel-opencl-icd 
│   │   └── ocl-icd-libopencl1 
│   │
│   ├── Level Zero Runtime (Compute API)
│   │   ├── libze-intel-gpu1 
│   │   └── libze1 
│   │
│   └── Workload Layer        
│       ├── Ollama (Dockered oneAPI / SYCL runtime) 
│       └── llama.cpp (optional backend: Vulkan / OpenCL / SYCL) 
│
├── llama.cpp (native inference engine)   
│   └── Vulkan backend (Mesa ANV)   
│
├── Games / Rendering
│   ├── OpenGL
│   │   ├── mesa-utils 
│   │   └── mesa-utils-bin 
│   │
│   └── Vulkan Tools (optional, verification / compute fallback)
│       └── vulkan-tools 
│
└── Video / Media Acceleration
    ├── VA-API Driver
    │   └── intel-media-va-driver-non-free 
    │
    └── VA-API Verification
        └── vainfo 

Developer / Tooling Layer
├── intel-oneapi-base-toolkit (optional, heavy install)
│   ├── sycl-ls  (device enumeration: OpenCL + Level Zero)
│   ├── compiler / runtime tools
│   └── oneAPI dev utilities
│
└── Docker Workloads (recommended for AI stacks)
    ├── intelanalytics/ipex-llm-inference-cpp-xpu 
    └── uberchuckie/ollama-intel-gpu 

Monitoring / Diagnostics
├── clinfo 
│   └── OpenCL platform/device inspection
│
├── intel-gpu-tools 
│   └── low-level GPU engine counters / debugging
│
├── nvtop 
│   └── real-time GPU utilization + memory tracking
│
└── sycl-ls  (if oneAPI installed or inside Docker)
    └── unified view: OpenCL + Level Zero devices

Userspace Drivers / Runtime Libraries
├── Mesa Stack
│   ├── mesa-utils 
│   ├── mesa-utils-bin 
│   └── Intel Vulkan / OpenGL userspace (ANV / Mesa Intel driver) 
│
├── Intel Compute Runtime (GPU Compute Layer)
│   ├── intel-opencl-icd 
│   ├── libze-intel-gpu1 
│   └── libze1 
│
└── VA-API Stack (Media Acceleration)
    └── intel-media-va-driver-non-free 

Kernel Driver Layer
├── i915  (classic Intel DRM driver)
└── xe (new Intel GPU driver path, future/default on newer kernels)
                ↓
Intel Arc GPU (BMG G31 / Pro B70)

Vulkan Stack (Graphics + Compute validation only)
├── vulkan-tools 
│   ├── vulkaninfo (device + driver validation)
│   └── vkcube (visual pipeline test)
│
└── Mesa Intel Vulkan Driver (Mesa ANV)
    └── part of mesa userspace stack (already included above)
```

## 6. General Tips 

### Hardware Setup 

#### Power Supply TDP
> *Note: Be sure your PSU can handle the 230 TDP!*

Dell XPS 500 W Platinum can handle it!

### For a Dell XPS  Build? 
* The card  perfectly fits in the Dell XPS 8940 case , but it requires modification of the front panel to accommodate the Arc Power Jack 
* you need to cut out a hole, which will be invisible when the front panel is installed. 
* Alternatively, use one of these fancy 90-degree adapter from ebay.

#### Cooling 

* If you are using a similar setup, such as the Dell XPS 8940, I recommend installing additional fans or replacing the stock ones. 
* A larger CPU cooler should be also on your list 

 
### Before you start: Check Disk Space

> **Important: Check your Disk Space & partitioning, especially if you setup a brand new ubuntu! Driver, Tools, Models, Docker images can be very large!** 

My partitioning:
```text
 	Filesystem      Size  Used Avail Use%  Mounted on
	/dev/nvme0n1p1   43G     32G   10G   77% /
	/dev/nvme0n1p3   75G     30G   44G   41% /home
	/dev/nvme0n1p4  359G    329G   29G   92% /data
```	
		  
- This setup led to a consistently full `/home` partition. Even after creating a `/data/ai` directory for large files and improving backups
- Do not  underestimated the amount of space required in `/home` for things like APT package installations
- Docker downloads mya fail due to low space, leaving behind hidden, temporary files in the containerd folder that you might not realize are there – and they are huge
- Yes you can "fix it" with symbolic links but it **may cause compatibility issues** with some tools, such as the Openclaw workspace
- 
 **Recommendation: Allocate at least 130GB to the `/home` partition.**
 	
### Driver and Workload Options 
After the base driver setup: [Intel Arc B70 Base Driver Setup Guide](docs/01-base-driver-setup.md)  you need to decide between:

``OpenCL + Level Zero + Ollama (Dockerized)`` 
vs.
 ``Vulkan + llama.cpp`` 
or
``both`` in parallel - I recommend to start with **a) OpenCL + Level Zero + Ollama (Dockerized)**
 
#### a) OpenCL + Level Zero + Ollama (Dockerized)
- **Setup**:
	1. Install Base: -> [Intel Arc B70 Base Driver Setup Guide](docs/01-base-driver-setup.md)
	2.  Install Driver 	  		-> [Intel Arc B70 OpenCL Level Zero Setup Guide](docs/02-aneapi-stack.md)
	3.  Install Workload	-> [Intel Arc B70 Ollama (Dockerized) Oneapi Guide](docs/workloads/ollama-dockerized-oneapi.md)

*   **Pros:**
    *   Easier to set up.
    *   More optimized and stable than Vulkan 
*   **Cons:**
	* 	Outdated model support: The official Intel Docker image/custom Docker images do not support the latest models (e.g., Gemma 3, Gemma 4).
    *   Relies on a custom Docker image that works for me.  I have been unable to get the official Intel Ollama Docker image working with my setup, possibly due to incorrect iGPU or CPU model selection.

>   **Why Dockerized** * The official Ollama native installation currently does not support Intel Arc.
 
#### b) Vulkan + llama.cpp
- **Setup**:
	1. Install Base: -> [Intel Arc B70 Base Driver Setup Guide](docs/01-base-driver-setup.md)
	2. Install Driver 			->  [Intel Arc B70 Vulkan Setup Guide](docs/03-vulkan-stack.md)
	3. Install Workload	-> [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md)

*   **Pros:**
    *   Supports newer models (e.g., Gemma 4).
	*   Faster responses when the model is loaded in VRAM.
*   **Cons:**
    *   More complex setup – requires tools like CMake and other build tools.
    *   Less optimized and stable. 
	    * Loading a model into VRAM takes considerably more time than with Ollama Dockerized. If you frequently switch models, OneAPI is a better choice.
    *   My system sometimes sounds like a jet engine after using LLM with llama.cpp.
	
### Other Drivers and Workload Combinations 

**Vulkan + Ollama (Dockerized)**
*   I haven't been able to get Ollama to detect the Vulkan backend with this setup. I will try this and update the documentation.

>**Important Notes**  Intel Arc support on Linux is evolving rapidly. Hopefully, we will soon get official native support for Ollama, and an updated official Intel Dockerized image.


## 7. Use Cases

### Vulkan
- llama.cpp Vulkan backend
- Ollama
- Vulkan compute
- AI inference
- Multi-GPU inference

### oneAPI / SYCL
- OpenVINO
- Intel optimized AI workloads
- SYCL compute
- HPC workloads
- Intel Extension for PyTorch

## 8 Coming soon

**Currently undocumented**
- Whisper TTS Setup & Config
- CompfyUI Setup & Config
- Openclaw Setup & Config
- n8n Setup & Config


## 9. Contributing

Contributions are welcome.

Please open an issue before large changes.

---

## 10. License

MIT License
