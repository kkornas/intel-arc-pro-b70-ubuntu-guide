# Intel Arc Pro B70 on Ubuntu
A guide to setting up the Intel Arc Pro B70 graphics card on Ubuntu Linux. Let's unlock the potential of your Arc Pro!

This repository documents:

- Intel Arc Pro B70 driver installation
- Ubuntu kernel and Mesa requirements
- Intel oneAPI / SYCL setup
- Vulkan compute configuration
- Validation and diagnostics
- installation and setup of workloads llama-cpp, ollama, (openclaw, n8n, comfyui, whisper follwoing)

---


## Tested Environment

This guide was tested on the following hardware and software configuration.

### Hardware

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

### Software

| Component       | Version           |
| --------------- | ----------------- |
| OS              | Ubuntu 25.10 LTS  |
| Kernel          | Linux 6.x 6.17.0-35-generic        |

* For Mesa, Vulkan, oneAPI etc refer to Package Overview 

### Notes

This guide reflects a real-world working configuration.

Different CPUs, motherboards, power supplies, Ubuntu versions, kernels, and Mesa versions may behave differently.

Intel Arc support on Linux evolves rapidly, so newer kernels and Mesa releases may improve stability and performance.


### Hardware Installation

Be sure your PSU can handle the 230 TDP!

The card fits the Dell XPS 8940 case perfectly, but it requires modification of the front panel to accommodate the Arc Power Jack – you need to cut out a hole, which will be invisible when the front panel is installed. Alternatively, use one of these fancy 90-degree adapter from ebay.

If you are using a similar setup, such as the Dell XPS 8940, I recommend installing additional fans or replacing the stock ones. A larger CPU cooler should be also on your list."

## Before you start

*Important* Check your Disk Space & partitioning, especcially if you setup a brand new ubuntu!


- models and docker images can be very large, up to 30GB
- necessary drivers and tools consume a significant amount of storage space
- in my case
Filesystem      Size  Used Avail Use%  Mounted on
/dev/nvme0n1p1   43G     32G   10G   77% /
/dev/nvme0n1p3   75G     30G   44G   41% /home
/dev/nvme0n1p4  359G    329G   29G   92% /data
	
- The Problem: this setup led to a consistently full /home partition. Even after creating a /data/ai directory for large files and improved backups, 
I underestimated the amount of space required in /home for things like apt package installations.Configuring this is annoying,
 but the biggest problem is that Docker downloads can fail due to low space, leaving behind hidden, temporary files in the containerd folder that
 you may not realize are there - and they are huge
 As a result, I had to move default locations like the Docker (especially the containerd folder) to the /data partition.

 A smaller home requires managing symbolic links and can cause compatibility issues with some tools, such as the Openclaw workspace!
 
 *Recommendation:* Allocate at least 130GB to the /home partition."
   
 	
## Where to start?

First Step: [Intel Arc B70 Base Driver Setup Guide](docs/01-base-driver-setup.md)
 
If you proceed with this guiede decide between:

a) oneAPI + Ollama (Dockerized)
b) Vulkan + llama.cpp

There are other possible combinations, but I haven’t successfully tested them.

**a) opencl level zero + oneAPI + Ollama (Dockerized)**

*   **Steps:** docs/01-base-driver-setup.md + docs/02-opencl-level-zero.md + docs/workloads/02-ollama-dockered-oneapi.md
*   **Pros:**
    *   Easier to set up.
    *   More optimized and stable than Vulkan 
*   **Cons:**
	* 	outdated model support > The official intel docker image/Custom Docker images do not support the latest models like e.g. gemma3 works, gemma4 not! 
    *   Relies on a custom Docker image that works for me. Could not get the official Intel Ollama Docker image working with my setup.	Possible reasons: the iGPU or CPU models were selected instead of the Arc.

* **Why Dockerized:** The official Ollama native installation does not currently support Intel Arc.




**b) Vulkan + llama.cpp**

*   **Steps:** docs/03-vulkan-stack.md + docs/workloads/03-llama-cpp-vulkan.md
*   **Pros:**
    *   You can run newer modles -> e.g. gemma4
	*   When model is loaded in the Vram -> faster responses than ollama
*   **Cons:**
    *   More complex setup – requires tools like CMake and other build tools.
    *   Less optimized and stable. 
	*       Loading a model in the Vram took definetly more time than with ollama (same models tested) -> so if you want to switch models, oneapi is the way to go instead of vulkan
	*       When Running my Setup starts to listen like a jet engine, and it didnt stop until i killed llama.cpp
	

**Other Possible Combinations**

**Vulkan + Ollama (Dockerized)**
*   I haven't been able to get Ollama to detect the Vulkan backend with this setup. I will try this and update the documentation.

**Important Notes**
Intel Arc support on Linux is evolving rapidly. Hopefully, we will soon get official native support for Ollama, and an updated official Intel Dockerized image.


# Intel Arc Stack (Compute + Graphics + AI Workload)

Full Stack after installing both Paths:  *Vulkan + llama.cpp & OpenCL + Level Zero + Ollama Oneapi*


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
	
	
## 4. Package Overview 

Full Stack after installing both Paths:  *Vulkan + llama.cpp & OpenCL + Level Zero + Ollama Oneapi*

## OpenCL + Level Zero (Compute / AI)

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

## Graphics / Rendering

| Layer                            | Package            | Installed Version | Purpose |
| -------------------------------- | ------------------ | ----------------: | -------- |
| OpenGL Tools                     | `mesa-utils`      | `9.0.0-2`         | `glxinfo`, `glxgears` |
| OpenGL Helper Binaries           | `mesa-utils-bin`  | `9.0.0-2`         | additional Mesa utilities |
| Vulkan Tools (verification only) | `vulkan-tools`    | `1.4.304.0+dfsg1-1` | `vulkaninfo`, `vkcube` | 
| Vulkan Loader (Mesa / system)    | `libvulkan1`      | `1.4.321.0-1`   | Vulkan runtime loader |
| Intel Vulkan Driver (Mesa ANV)   | `mesa-vulkan-drivers` | `25.2.8-0ubuntu0.25.10.2` | Intel Arc Vulkan backend |
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

## LLM Runtime (llama.cpp)

| Layer                          | Component        | Installed Version | Purpose |
| ----------------------------- | ---------------- | ----------------: | -------- |
| llama.cpp (source build)*     | compiled from source | `9436 (d6588daa8, gcc 11.4)` | local LLM inference engine |
| GGUF Model Support            | built-in        | -                 | model format support |
| Vulkan Backend (optional)     | compile flag     | enabled/disabled  | GPU acceleration via Vulkan |
| OpenCL Backend (optional)     | compile flag     | enabled/disabled  | Intel GPU compute backend |
* compiled from source. Installation steps-> [docs/llama-cpp-vulkan??.md][ ?? ] https://github.com/ggml-org/llama.cpp/releases/download/b9436/llama-b9436-bin-ubuntu-x64.tar.gz
---
## Developer / Tooling Layer (Intel oneAPI)
| Layer                     | Package           | Installed Version | Purpose                     |
| ------------------------- | ----------------- | ----------------: | --------------------------- |
| oneAPI Base Toolkit (sycl-ls)*    | offline installer    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |
| SYCL Device Inspector     | oneAPI Base Toolkit (sycl-ls)    |             `2026.0.0.198_` | to verify Level Zero with SYCL    |

* oneAPI Base Toolkit Installed via offline installer (not apt) Installation steps-> [docs/02-opencl-level-zero.md][3.4 Verfiy Level Zero with SYCL (final compute test)]
or https://www.intel.com/content/www/us/en/developer/tools/oneapi/oneapi-toolkit-download.html?packages=oneapi-toolkit&oneapi-toolkit-os=linux&oneapi-lin=offline*

---




