
# Intel Arc B70 Base Driver Setup Guide

> **Important:** Before proceeding, check the README.md file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide

This guide shows the basic configuration for the Arc. After this setup, you should decide between 

**a) OpenCL + Level Zero + Ollama (Dockerized)**
- Install Driver 	  		-> [Intel Arc B70 OpenCL Level Zero Setup Guide](docs/02-aneapi-stack.md)
- Install Workload	-> [Intel Arc B70 Ollama (Dockerized) Oneapi Guide](docs/workloads/ollama-dockerized-oneapi.md)

 
**b) Vulkan + llama.cpp**
- Install Driver 			->  [Intel Arc B70 Vulkan Setup Guide](docs/03-vulkan-stack.md)
- Install Workload	-> [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md)


Setting up the Arc after installation is straightforward – basically, you install the hardware, check the BIOS for errors, run `apt update`, and it should work.

> **Disclaimer:** System configurations vary, and some commands may require adjustments for your specific hardware and software. Commands may fail due to system-specific differences,
Please read the Error messages - they provide valuable clues. 


## 1. Uninstall NVIDIA Drivers (Optional)

If you previously had an NVIDIA card (like me an RTX 3060 Ti), uninstall the drivers *before* installing the new card.

```bash
sudo apt remove --purge 'nvidia-*'
sudo apt autoremove
```

## 2. Install New Card and Connect Display

- Install the Intel Arc graphics card
- You can run the card headless (without connecting a DisplayPort cable), but some tools will not work.
> For the initial setup, I recommend connecting a DisplayPort cable and a monitor
## 3. Open your Bios and Enable Resizable BAR (REBAR)

Enable Resizable BAR (REBAR) in your system's BIOS settings.  
>**REBAR** is important for optimal performance with the Arc card.


## 4. Update System (Base Installation)

After updating your system and rebooting, the Intel Arc drivers *should* be automatically installed. In my case, this worked!

```bash
sudo apt update
sudo apt full-upgrade
sudo reboot
```
## 5. Verify  

Check if the card is recognized and the drivers are loaded. 

### 5.1 Check PCI Devices -> Arc B70 is detected by PCIe Bus

```bash
lspci -v | grep -i VGA
```

**Expected Output:**

```
00:02.0 VGA compatible controller: Intel Corporation RocketLake-S GT1 [UHD Graphics 750] (rev 04) (prog-if 00 [VGA controller])
04:00.0 VGA compatible controller: Intel Corporation Battlemage G31 [Intel Graphics] (prog-if 00 [VGA controller])
```

>**Note:** This output shows both the integrated Intel UHD Graphics and the Intel Arc Battlemage G31.  Yours may vary.*

### 5.2 Check Kernel module

This verifies the `i915` kernel module is loaded.

```bash
lsmod | grep -E "i915|xe"
```

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

### 5.3. Verify Basic Function (Desktop Mode Only)

If you see the screen, it worked! :-)

Optionally, you can run a benchmark or a simple stress test and monitor the load in nvtop.

**Terminal 1**
```bash
sudo apt nvtop
nvtop
```
**Terminal 2**
```bash

sudo apt install stress-ng
stress-ng --gpu 1 --timeout 30s
```

## 6 Comment

THAT'S IT- your system is ready for the next Steps:

**a) OpenCL + Level Zero + Ollama (Dockerized)**
- 1. Install Driver 	  		-> [Intel Arc B70 OpenCL Level Zero Setup Guide](docs/02-aneapi-stack.md)
- 2. Install Workload	-> [Intel Arc B70 Ollama (Dockerized) Oneapi Guide](docs/workloads/ollama-dockerized-oneapi.md)

or
 
**b) Vulkan + llama.cpp**
- 1. Install Driver 			->  [Intel Arc B70 Vulkan Setup Guide](docs/03-vulkan-stack.md)
- 2. Install Workload	-> [Intel Arc B70 llama.cpp Vulkan Guide](docs/workloads/llama-cpp-vulkan.md)

or

**both!** ->  then, I recommend to start with **a) OpenCL + Level Zero + Ollama (Dockerized)**
