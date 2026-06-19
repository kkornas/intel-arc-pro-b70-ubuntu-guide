# Intel Arc B70 Ollama (Dockerized) Oneapi Guide

>**Important:** Before proceeding, check the README.md file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide.


This guide explains how to run **Ollama on Intel Arc GPUs** using a Dockerized setup with **OpenCL + Level Zero support**:

It is based on a working Intel GPU compute stack consisting of:
- OpenCL Runtime
- Level Zero Runtime
- Intel Kernel Driver (i915 / xe)
- optional: oneAPI Toolkit (`sycl-ls` for verification)

Setup Guide here: [docs/02-opencl-level-zero.md]

---
## 1. Prerequisites

Ensure your system has:

- Intel Arc GPU detected
- `/dev/dri/renderD*` available
- OpenCL + Level Zero installed on host
- Docker + Docker Compose installed


## 2. Working Docker Image

### Tested and working image
```
uberchuckie/ollama-intel-gpu:latest
```
**Characteristics:**

* Based on Intel IPEX / oneAPI stack
* Includes OpenWebUI
* Small (~2GB)
* Ollama version: 0.9.3 (older but stable)
* Works with Intel Arc + Level Zero + OpenCL


**Docker Images that did notwork in this setup:**

```
intelanalytics/ipex-llm-inference-cpp-xpu:latest (do not detect ARC over DRM device interface /dev/dri/* )
visitsb/ollama-ipex:latest
spaceinvaderone/ollama-intel-gpu:latest
ollama/ollama:latest (no support for oneapi)
```


## 3. Setup Directory

```bash
mkdir -p ~/ollama-arc
cd ~/ollama-arc
```

## 4. Create docker-compose.yml in ~/ollama-arc

```yaml
services:
  ollama:
    image: uberchuckie/ollama-intel-gpu:latest
    container_name: ollama-intel
    devices:
      - /dev/dri
    volumes:
      - ~/.ollama:/root/.ollama
      - /data/ai/models:/models  # model location 
    environment:
      - OLLAMA_MODELS=/models
      - OLLAMA_HOST=0.0.0.0
      - DEVICE=Arc
      - OLLAMA_NUM_GPU=999
      - OLLAMA_NUM_CTX=32768 #16384
      - ZES_ENABLE_SYSMAN=1
      - SYCL_CACHE_PERSISTENT=1
      - OLLAMA_MAX_LOADED_MODELS=1
      - OLLAMA_NUM_PARALLEL=1
      - no_proxy=localhost,127.0.0.1
      - ONEAPI_DEVICE_SELECTOR=level_zero:0     # test 1, 2 ... This number may be different for you
    ports:
      - "11434:11434"
    restart: unless-stopped
    shm_size: '32g'               # for arc b70 '32g'
    privileged: true

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui
    volumes:
      - openwebui:/app/backend/data
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    restart: unless-stopped
    depends_on:
      - ollama

volumes:
  ollama-models:
  openwebui:
```

## 5. Start Containers

```bash
docker compose up -d
```

> The initial run will download the image. If this fails, run ``docker compose pull`` and then ``docker compose up -d``

**Expected Output:**
```
[+] up 3/3
 ✔ Network ollama-arc_default Created                                                                                                                                                    0.1s
 ✔ Container ollama-intel     Started                                                                                                                                                    0.4s
 ✔ Container openwebui        Started      
```


### If startup fails 

```bash
docker compose up -d
sleep 3
docker compose up -d
```



## 6. Convenience Alias

```bash
echo 'alias ollama="docker exec -it ollama-intel ollama"' >> ~/.bashrc
source ~/.bashrc
```

Test:

```bash
ollama --version
```


**Expected Output:**
```
ollama version is 0.9.3
```


## 7. Verify Containers

```bash
docker ps
```

Expected:

* ollama-intel / uberchuckie/ollama-intel-gpu:lates 
* openwebui / ghcr.io/open-webui/open-webui:main

 


## 8. Download a Model

Recommended small test model: llama3.2:1b only 1.3 GB

```bash
ollama pull llama3.2:1b
```

List models:

```bash
ollama list
```

**Expected Output:**
```
NAME                           ID              SIZE      MODIFIED
llama3.2:1b                    baf6a787fdff    1.3 GB    4 minutes ago
```



## 9. Run Model

```bash
ollama run llama3.2:1b
```
Test Promt: Input whatever you like e.g. 'test'

**Expected Output: Ollama Chat**
```
>>> test
Can I help you with something else?

>>> Send a message (/? for help)
```

* Congratulations your Setup is ready*

*Note* "Starting a new chat loads the model into VRAM, causing a short delay."

CTRL+D to exit


## 10. GPU Monitoring (optional)

run in parallel

```bash
nvtop
```
Chat with model, then you should see
* Intel Arc GPU (BMG G31)
* increasing memory usage during inference


**nvtop parallel with llama3.2:1b chat: **

```
Device 0 [Battlemage G31 (Intel Graphics)]     PCIe GEN 3@16x RX: N/A TX: N/A
 GPU 2800MHz MEM N/A MHz TEMP  30fC FAN    0RPM POW   7 W
 GPU[                                       0%] MEM[||||                     3.1

 Device 1 [RocketLake-S GT1 (UHD Graphics 750)] Integrated GPU RX: N/A TX: N/A
 GPU 350MHz  MEM N/A MHz TEMP N/AfC   CPU-FAN   POW N/A W
 GPU[                                       0%] MEM[
   lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk    lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
100xGPU0 %                            x 100xGPU1 %                            x
   xGPU0 mem%                         x    xGPU1 mem%                         x
 75x                                  x  75x                                  x
   x                                  x    x                                  x
 50x                                  x  50x                                  x
   x                                  x    x                                  x
 25x               lqqqqqqqqqqqqqqqqqqx  25x                                  x
  0xqqqqqqqqqqqqqqqvqqqqqqqqqqqqqqqqqqx   0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj    m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj
    PID  USER DEV     TYPE  GPU        GPU MEM    CPU  HOST MEM Command

F2Setup   F6Sort    F9Kill    F10Quit    F12Save Config
```

## 11. Larger Model Test

For example gemma3:27b (17 GB ) which is a good compromise of speed and disk / vram usage 
```bash
ollama pull gemma3:27b
ollama run gemma3:27b
```

*nvtop parallel with gemma3:27b chat  
```
 Device 0 [Battlemage G31 (Intel Graphics)]     PCIe GEN 3@16x RX: N/A TX: N/A
 GPU 2800MHz MEM N/A MHz TEMP  35fC FAN  655RPM POW  81 W
 GPU[                                       0%] MEM[||||||||||||||||||||||||22.1

 Device 1 [RocketLake-S GT1 (UHD Graphics 750)] Integrated GPU RX: N/A TX: N/A
 GPU 350MHz  MEM N/A MHz TEMP N/AfC   CPU-FAN   POW N/A W
 GPU[                                       0%] MEM[
   lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk    lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
100xGPU0 %                            x 100xGPU1 %                            x
   xGPU0 mem%                         x    xGPU1 mem%                         x
 75x               lqqqqqqqqqqqqqqqqqqx  75x                                  x
   x               x                  x    x                                  x
 50x               x                  x  50x                                  x
   x               x                  x    x                                  x
 25xqqqqqqqqqqqqqqqj                  x  25x                                  x
  0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx   0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj    m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj
    PID  USER DEV     TYPE  GPU        GPU MEM    CPU  HOST MEM Command

F2Setup   F6Sort    F9Kill    F10Quit    F12Save Config

```

## 12. Stop Docker 

```bash
docker compose down
```
**Expected Output:**
```
[+] down 3/3
 ✔ Container openwebui        Removed                                                                                                                                                    2.5s
 ✔ Container ollama-intel     Removed                                                                                                                                                   10.3s
 ✔ Network ollama-arc_default Removed      
```


## 13. Architecture Overview

```
Ollama (Docker)
    ↓
OpenCL / Level Zero Runtime
    ↓
Intel Compute Runtime (NEO + libze)
    ↓
Kernel Driver (i915 / xe)
    ↓
Intel Arc GPU
```



## 14. Key Notes

* Docker does NOT replace Level Zero or OpenCL
* It only uses the host GPU stack
* GPU access is provided via `/dev/dri`


## 15. Optional Verification (inside container)

Verification tools depend on the container image.

### uberchuckie/ollama-intel-gpu:latest

This image is lightweight and did **not include developer verification tools** such as:

```bash
sycl-ls
clinfo
```

In this case, just run it and use real workload to verify the GPU.

1. Open GPU monitoring on the host:

```bash
nvtop
```

2. Run a model inside Ollama:

```bash
ollama run llama3.2:1b
```

3. Observe GPU memory usage (`GPU MEM`) on the Arc GPU.

**Expected behavior:**

- VRAM usage increases
- GPU power consumption rises
- Model responds normally

This confirms:

- `/dev/dri` passthrough works
- Intel GPU runtime is accessible
- Ollama uses the Arc GPU for inference



### Official Intel image (`intelanalytics/ipex-llm-inference-cpp-xpu:latest`)

The official Intel image may include oneAPI tooling.

You can try:

```bash
docker exec -it ollama-intel bash

sycl-ls
clinfo
```

**Expected Output (similar):**

```bash
[level_zero:gpu][level_zero:0]
Intel(R) Arc(TM) Pro B70 Graphics[level_zero:gpu][level_zero:0] Intel(R) oneAPI Unified Runtime over Level-Zero V2, Intel(R) Arc(TM) Pro B70 Graphics 20.2.0 [1.15.38308+1]
```

If the Arc GPU appears through `level_zero`, Level Zero / SYCL is working correctly inside the container.

Anyway.. because my drm is not working correctly intelanalytics/ipex-llm-inference-cpp-xpu:latest didn't work for me, maybe you have more luck!




## 16. Docker Compose Parameters Explained

Below is a short explanation of the most important parameters used in the `docker-compose.yml` 

### GPU Device Passthrough

```yaml
devices:
  - /dev/dri
```

This exposes Intel GPU render devices from the host into the container.

Without `/dev/dri`, Ollama inside the container cannot access the Intel Arc GPU.

*Info* with uberchuckie/ollama-intel-gpu:latest changing this had no effect! 

You can verify available GPU devices on the host:

```bash
ls /dev/dri
```

**Example Output:**

```bash
by-path  card1  card2  renderD128  renderD129
```

---

### Model Storage Path

```yaml
volumes:
  - /data/ai/models:/models
```

This defines where Ollama models are stored.

**Format:**

```text
HOST_PATH:CONTAINER_PATH
```

Example:

```yaml
- /data/ai/models:/models
```

means:

* `/data/ai/models` → folder on your host system
* `/models` → folder inside the container

You should adjust this path to a location with enough free disk space.

Examples:

```yaml
- /mnt/ssd/ollama-models:/models
```

or

```yaml
- ~/ollama-models:/models
```

The following environment variable tells Ollama to use this path:

```yaml
environment:
  - OLLAMA_MODELS=/models
```

---

### GPU Selection (`ONEAPI_DEVICE_SELECTOR`)

```yaml
environment:
  - ONEAPI_DEVICE_SELECTOR=level_zero:0
```

This selects which Intel GPU should be used by oneAPI / SYCL.

The device index depends on your system configuration.

Examples:

```yaml
level_zero:0
level_zero:1
level_zero:2
```

Typical setups:

| Configuration       |                      Suggested Value | 
| ------------------- | -----------------------------------: |
| Arc GPU only        |                       `level_zero:0` |
| iGPU + Arc GPU      | try `level_zero:0` or `level_zero:1` |
| Multiple Intel GPUs |              test `0`, `1`, `2`, ... |

You can determine the correct index using:

```bash
sycl-ls
```

**Example Output:**

```bash
[level_zero:gpu][level_zero:0] Intel(R) oneAPI Unified Runtime over Level-Zero V2, Intel(R) Arc(TM) Pro B70 Graphics 20.2.0 [1.15.38308+1]
[level_zero:gpu][level_zero:1] Intel(R) oneAPI Unified Runtime over Level-Zero V2, Intel(R) UHD Graphics 750 12.1.0 [1.15.38308+1]
[opencl:cpu][opencl:0] Intel(R) OpenCL, 11th Gen Intel(R) Core(TM) i7-11700 @ 2.50GHz OpenCL 3.0 (Build 0) [2026.21.3.0.31_160000]
[opencl:gpu][opencl:1] Intel(R) OpenCL Graphics, Intel(R) Arc(TM) Pro B70 Graphics OpenCL 3.0 NEO  [26.18.38308.1]
[opencl:gpu][opencl:2] Intel(R) OpenCL Graphics, Intel(R) UHD Graphics 750 OpenCL 3.0 NEO  [26.18.38308.1]
```

In this example:

* `level_zero:0` → Intel Arc B70
* `level_zero:1` → Intel UHD 750 (iGPU)

If inference unexpectedly runs on the iGPU, try changing the selector and restart the container:

```bash
docker compose down
docker compose up -d
```

---

### Shared Memory (`shm_size`)

```yaml
shm_size: '32g'
```

Sets the container shared memory size. 
Larger models may require more shared memory to avoid runtime issues.

Recommendations:

| GPU VRAM               | Suggested `shm_size` |
| ---------------------- | -------------------: |
| 8–12 GB                |             `8g–16g` |
| 16–24 GB               |            `16g–24g` |
| 24–32 GB (Arc Pro B70) |                `32g` |

---

### GPU Count

```yaml
- OLLAMA_NUM_GPU=999
```

Allows Ollama to use all available GPU resources.

For Intel Arc this is commonly left at:

```yaml
OLLAMA_NUM_GPU=999
```

---

### Context Window

```yaml
- OLLAMA_NUM_CTX=32768
```

Defines the maximum context length.

Higher values require more VRAM.

Examples:

| Context Size | VRAM Usage |
| ------------ | ---------: |
| `8192`       |      lower |
| `16384`      |     medium |
| `32768`      |       high |

If you experience memory issues, reduce this value.

---

### Model Parallelism

```yaml
- OLLAMA_MAX_LOADED_MODELS=1
- OLLAMA_NUM_PARALLEL=1
```

Recommended for stability on Intel Arc setups.

This prevents:

* multiple models being loaded simultaneously
* excessive VRAM pressure
* unstable inference behavior
