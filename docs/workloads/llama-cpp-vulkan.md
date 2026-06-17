# Intel Arc B70 llama.cpp Vulkan Guide

**Important:** Before proceeding, check the README.md file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide.

This guide explains how to run **llama.cpp on Intel Arc GPUs** with **Vulkan driver**:

It is based on a working Vulkan compute stack consisting of:
- Intel Vulkan Driver (Mesa ANV)
- Vulkan Loader (Userspace Runtime)
- vulkan-tools Diagnose-Tools (vulkaninfo, vkcube)

Setup Guide here: [docs/03-vulkan-stack.md]

--- 

## Prerequisites

Ensure your system has:

- Intel Arc driver installed
- Intel Arc GPU detected
- Vulkan stack installed
- Verify Vulkan works

Quick Check
	
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
---

# Install llama.cpp (Prebuilt Binary) [Recommended]

## 1. Create Application Directory

```bash
mkdir -p ~/apps
cd ~/apps
```

## 2. Download and install Release

```bash
wget https://github.com/ggml-org/llama.cpp/releases/download/b9436/llama-b9436-bin-ubuntu-x64.tar.gz
tar -xzvf llama-b9436-bin-ubuntu-x64.tar.gz
mv llama-b9436 llama-cpp
```
## 3. Verify Installation

```bash
cd ~/apps/llama-cpp
./llama-server --version
```

**Expected Output:**
```
version: 9436 (d6588daa8)
built with GNU 11.4.0 for Linux x86_64
```

## 4. Verify Vulkan Arc GPU is detected
```bash
./llama-server --list-devices
```

**Expected Output (similar):**
```
Available devices:
  Vulkan0: Intel(R) Graphics (BMG G31) (32656 MiB, 8 MiB free)
  Vulkan1: Intel(R) Graphics (RKL GT1) (23414 MiB, 19465 MiB free)
```

...

## 5. Download Test Model

```bash
mkdir -p /data/ai/llama-cpp-models
cd /data/ai/llama-cpp-models
```

"Small" example model:

```bash
wget -O Qwen2.5-1.5B-Instruct-Q4_K_M.gguf https://huggingface.co/jc-builds/Qwen2.5-1.5B-Instruct-Q4_K_M-GGUF/resolve/main/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf?download=true
```

*Note* Models can be up to 30GB -> be sure you have enough disk space

Verify model download
```bash
ls -lh /data/ai/llama-cpp-models
```

...



## 6. Run First Model - running llama.cpp (Vulkan Backend)

*Important* If multiple GPUs are present, select the Vulkan device to use, Typical values:

```text
| Value | Device          |
| ----- | --------------- |
| 0     | Intel Arc GPU   |
| 1     | Intel UHD iGPU  |
| 2+    | Additional GPUs |
```
 
Get device number with:
```bash
./llama-server --list-devices
```
**Expected Output (similar):**
```
  Vulkan0: Intel(R) Graphics (BMG G31) (32656 MiB, 8547 MiB free)
```

-> GGML_VK_VISIBLE_DEVICES=0

```bash
export GGML_VK_VISIBLE_DEVICES=0
```

### Start Server 
```bash
./llama-server -m /data/ai/llama-cpp-models/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf \
--host 127.0.0.1 \
--port 8082 \
-ngl 999 \
--ctx-size 16384 \
--jinja
```

**Expected Output (similar, first 4 lines):**
```
0.00.056.780 I log_info: verbosity = 3 (adjust with the `-lv N` CLI arg)
0.00.056.782 I device_info:
0.00.056.949 I   - Vulkan0 : Intel(R) Graphics (BMG G31) (32656 MiB, 29063 MiB free)
0.00.056.952 I   - CPU     : 11th Gen Intel(R) Core(TM) i7-11700 @ 2.50GHz (31219 MiB, 31219 MiB free)
[..}
```
The following line confirms that llama.cpp is using the Intel Arc GPU:
```text
- Vulkan0 : Intel(R) Graphics (BMG G31) (32656 MiB, 29063 MiB free)
```

## 7.Verify 

### 7.1 Run llama.cpp inference api in parallel shell

```bash
curl http://127.0.0.1:8082/v1/chat/completions \
	  -H "Content-Type: application/json" \
	  -d '{
		"model": "qwen",
		"messages": [
		  {
			"role": "user",
			"content": "Hallo! Wer bist du?"
		  }
		],
		"temperature": 0.7
	  }'
```

**Expected Output (similar):**
```
{"choices":[{"finish_reason":"stop","index":0,"message":{"role":"assistant","content":"Ich bin Qwen, ein Conversational AI-Assistant, der mit Alibaba Cloud entwickelt wurde. Ich bin hier, um Ihnen zu helfen!"}}],"created":1781699738,"model":"Qwen2.5-1.5B-Instruct-Q4_K_M.gguf","system_fingerprint":"b9436-d6588daa8","object":"chat.completion","usage":{"completion_tokens":30,"prompt_tokens":35,"total_tokens":65,"prompt_tokens_details":{"cached_tokens":0}},"id":"chatcmpl-hlfAk74yDTo9JFTQEMFOAqMsoITBxsQX","timings":{"cache_n":0,"prompt_n":35,"prompt_ms":308.684,"prompt_per_token_ms":8.819542857142858,"prompt_per_second":113.38456155809823,"predicted_n":30,"predicted_ms":354.524,"predicted_per_token_ms":11.817466666666666,"predicted_per_second":84.62050524082996}}
```

You will also notice Updates in the ./llama-server command output

### 7.2 Show installed models

```bash
curl http://127.0.0.1:8082/v1/models
```
**Expected Output (similar):**
```
{"models":[{"name":"Qwen2.5-1.5B-Instruct-Q4_K_M.gguf","model":"Qwen2.5-1.5B-Instruct-Q4_K_M.gguf","modified_at":"","size":"","digest":"","type":"model","description":"","tags":[""],"capabilities":["completion"],"parameters":"","details":{"parent_model":"","format":"gguf","family":"","families":[""],"parameter_size":"","quantization_level":""}}],"object":"list","data":[{"id":"Qwen2.5-1.5B-Instruct-Q4_K_M.gguf","aliases":[],"tags":[],"object":"model","created":1781700575,"owned_by":"llamacpp","meta":{"vocab_type":2,"n_vocab":151936,"n_ctx":16384,"n_ctx_train":32768,"n_embd":1536,"n_params":1777088000,"size":1111370240}}]}
```

### 7.3 Run GPU Monitoring in parallel shell (optional)

```bash
nvtop
```

run llama.cpp inference api increasing memory usage during inference

*nvtop parallel llama.cpp inference api 

**Expected Output  (similar):**
```
Device 0 [Battlemage G31 (Intel Graphics)]     PCIe GEN 3@16x RX: N/A TX: N/A
 GPU 2800MHz MEM N/A MHz TEMP  35fC   FAN N/A   POW   0 W
 GPU[                                       0%] MEM[||                       2.0

 Device 1 [RocketLake-S GT1 (UHD Graphics 750)] Integrated GPU RX: N/A TX: N/A
 GPU 350MHz  MEM N/A MHz TEMP N/AfC   CPU-FAN   POW N/A W
 GPU[                                       0%] MEM[
   lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk    lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
100xGPU0 %                  lqk       x 100xGPU1 %                            x
   xGPU0 mem%               x x       x    xGPU1 mem%                         x
 75x                        x x       x  75x                                  x
   x                        x x       x    x                                  x
 50x                        x x       x  50x                                  x
   x                        x x       x    x                                  x
 25x                        x x       x  25x                                  x
  0xqqqqqqqqqqqqqqqqqqqqqqqqvqvqqqqqqqx   0xqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqx
   m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj    m17sqqqq12sqqqqqq8sqqqqqq4sqqqqqq0sj
    PID  USER DEV     TYPE  GPU        GPU MEM    CPU  HOST MEM Command 
F2Setup   F6Sort    F9Kill    F10Quit    F12Save Config
```



## 8. Convenience Shortcut

```bash
sudo ln -sf ~/apps/llama-cpp/llama-server /usr/local/bin/llama-server
```
-> This should work anywhere!

```bash
cd ~
llama-server --version
```




# Additional Information

## Download Models

### Small Test Model 

(Qwen2.5-1.5B) ~ 1.1GB

```bash
mkdir -p /data/ai/llama-cpp-models

cd /data/ai/llama-cpp-models

wget -O Qwen2.5-1.5B-Instruct-Q4_K_M.gguf \
https://huggingface.co/jc-builds/Qwen2.5-1.5B-Instruct-Q4_K_M-GGUF/resolve/main/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf?download=true
```

### Larger Model 

(Gemma 4 26B) ~ 16GB

```bash
wget -O gemma-4-26B-A4B-it-MXFP4_MOE.gguf \
https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/gemma-4-26B-A4B-it-MXFP4_MOE.gguf?download=true
```

(Qwen 3 30B) ~ 21GB

```bash
wget -O Qwen3-Coder-30B-A3B-Instruct-Q5_K_M.gguf \
https://huggingface.co/unsloth/gemma-4-26B-A4B-it-GGUF/resolve/main/Qwen3-Coder-30B-A3B-Instruct-Q5_K_M.gguf?download=true
```


Note: The Gemma and Qwen model is large and may require tens of gigabytes of disk space.

## llama-server run command Parameters

### Basic command

llama-server -m /data/ai/llama-cpp-models/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf \
--host 127.0.0.1 \--port 8082 -ngl 999 --ctx-size 16384 --jinja

### Optimized command
llama-server -m /data/ai/llama-cpp-models/gemma-4-26B-A4B-it-MXFP4_MOE.gguf --host 127.0.0.1 --port 8082 \
-ngl 999 \
--ctx-size 8192 \
-b 1024 -ub 512  \          
--flash-attn 1 \             
--jinja \                    
--cache-type-k q8_0 --cache-type-v q8_0 \   
-t 8 -tb 4                  

| Parameter        | Example                                                       | Purpose                                                                                                                                      |
| ---------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `-m`             | `/data/ai/llama-cpp-models/gemma-4-26B-A4B-it-MXFP4_MOE.gguf` | Path to the GGUF model file.                                                                                                                 |
| `--host`         | `127.0.0.1`                                                   | Bind the server to a network interface. `127.0.0.1` = localhost only. Use `0.0.0.0` for remote access.                                       |
| `--port`         | `8082`                                                        | HTTP API port used by llama-server.                                                                                                          |
| `-ngl`           | `999`                                                         | Number of layers offloaded to the GPU. `999` means "offload as many layers as possible".                                                     |
| `--ctx-size`     | `8192`                                                        | Context window size (tokens). Larger values improve long conversations but require more VRAM.                                                |
| `-b`             | `1024`                                                        | Logical batch size used during prompt processing. Larger values may improve throughput but increase VRAM usage.                              |
| `-ub`            | `512`                                                         | Physical micro-batch size. Controls how the batch is split internally. Useful when tuning memory usage and stability.                        |
| `--flash-attn`   | `1`                                                           | Enables Flash Attention. Usually improves performance and reduces memory usage on supported backends.                                        |
| `--jinja`        | enabled                                                       | Enables Jinja chat template support embedded in many modern GGUF models (Qwen, Gemma, Llama, etc.). Recommended for instruction/chat models. |
| `--cache-type-k` | `q8_0`                                                        | Quantization used for the Key KV cache. Higher precision improves quality but uses more VRAM.                                                |
| `--cache-type-v` | `q8_0`                                                        | Quantization used for the Value KV cache. Higher precision improves quality but uses more VRAM.                                              |
| `-t`             | `8`                                                           | Number of CPU threads used by llama.cpp. Usually set close to available CPU cores.                                                           |
| `-tb`            | `4`                                                           | Number of CPU threads used for batch processing. Fine-tuning parameter for CPU workload distribution.                                        |

Tuning Notes

*More VRAM available?
	*Increase --ctx-size
	*Increase -b
	*Use q8_0 KV cache
*Out of memory errors?
	*Reduce --ctx-size
	*Reduce -b
	*Reduce -ub
	*Use lower KV cache precision (q4_0, q5_0)
*GPU not used?
	*Check -ngl 999
	*Verify with:	
		```bash 
		llama-server --list-devices
		```
*Multi-GPU systems (Arc + UHD)
		```bash 
		export GGML_VK_VISIBLE_DEVICES=0
		```

# Connecting llama.cpp to External Tools

You can connect external Tools with the OpenAI-compatible API which is configured in the llama-server command:
```bash
 --host 127.0.0.1 --port 8082
```
OpenAI-compatible API Example/Verification:

```bash
curl http://127.0.0.1:8082/v1/chat/completions \
	  -H "Content-Type: application/json" \
	  -d '{
		"model": "qwen",
		"messages": [
		  {
			"role": "user",
			"content": "Hallo! Wer bist du?"
		  }
		],
		"temperature": 0.7
	  }'
```
-> if you successfully chat with you model the OpenAI-compatible API should work in external tools.

## OpenClaw + llama.cpp

Adapt your OpenClaw configuration file (for example`~/.openclaw/openclaw.json`)

Add a local llama.cpp "local-llama" provider:

```json
  "models": {
    "mode": "merge",
    "providers": {
	"local-llama": {
			"baseUrl": "http://127.0.0.1:8082/v1",
			"apiKey": "dummy",
			"api": "openai-completions",
			"models": [
			  {
				"id": "gemma-4-26B-A4B-it-MXFP4_MOE",
				"name": "gemma-4-26B-A4B-it-MXFP4_MOE",
				"reasoning": false,
				"contextWindow": 8192,
				"maxTokens": 8192
			  },
			  {
				"id": "Qwen3-Coder-30B-A3B-Instruct-Q5_K_M",
				"name": "Qwen3-Coder-30B-A3B-Instruct-Q5_K_M",,
				"reasoning": false,
				"contextWindow": 16384,
				"maxTokens": 8192
			  }
			]
		  },	
		},
	},		  
[...]
```
And change the primary model!
```json
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "model": {
        "primary": "local-llama/gemma-4-26B-A4B-it-MXFP4_MOE"
      },
      "models": {
        "local-llama/gemma-4-26B-A4B-it-MXFP4_MOE": {},   
      }
    }
  },
```

## Open WebUI + llama.cpp

Install Open WebUI using your preferred deployment method
(for example Docker).

Then navigate to:

Settings
 → Connections
 → OpenAI API

and add:

Base URL:
http://127.0.0.1:8082/v1

API Key:
dummy

*Note* When using the Dockerized Open WebUI, the URL may change to http://host.docker.internal:8082, depending on your system and Docker network settings!


# Systemd Service for Permanent Use (Optional)

*Note* if you switch between Ollama and llama.cpp you have to disable the systemd service

## 1. Create llama-server command script

Create a script file for the Systemd Service. Adapt the llama-server command to your preferred configuration and model (do not forget to download it).

Example: ```~/scripts/start-llama.sh```  

```
#!/bin/bash

export GGML_VK_VISIBLE_DEVICES=0

llama-server -m /data/ai/llama-cpp-models/gemma-4-26B-A4B-it-MXFP4_MOE.gguf --host 127.0.0.1 --port 8082 \
-ngl 999 \
--ctx-size 8192 \
-b 1024  \
-ub 512  \          
--flash-attn 1 \             
--jinja \                    
--cache-type-k q8_0 --cache-type-v q8_0 \  
-t 8 -tb 4                 
```
*Important!*

```chmod +x ~/scripts/start-llama.sh```

## 2. Create Systemd Service file: ```/etc/systemd/system/llama.service```

and add:

```
[Unit]
Description=llama.cpp server
After=network.target

[Service]
Type=simple
User='user name'
Group='group'
WorkingDirectory=/home/'user name'

Environment=GGML_VK_VISIBLE_DEVICES=0

ExecStart=/home/'user name'/scripts/start-llama.sh

Restart=always
RestartSec=5

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

## 3. Finish Systemd Setup
```bash
sudo systemctl daemon-reload
sudo systemctl enable llama
sudo systemctl start llama
sudo systemctl status llama
```
## 4. Stop

*Requiered when changing the llama-server command parameters in ```~/scripts/start-llama.sh```  

```bash
sudo systemctl stop llama
sudo systemctl restart llama
```

# Troubleshooting

## Vulkan Device Not Found

Verify Vulkan installation:

```bash
vulkaninfo | grep deviceName
```

Install missing Vulkan packages:

```bash
sudo apt install -y mesa-vulkan-drivers libvulkan-dev libvulkan1 vulkan-tools
```

-> Full Guide here: [docs/03-vulkan-stack.md]

### CPU Only Execution

Check startup logs.

If only:

```text
CPU
```

appears and no:

```text
Vulkan0 : Intel(R) Graphics (BMG G31) # or similiar
```

is listed, Vulkan acceleration is not being used.

Verify available devices:

```bash
llama-server --list-devices
```

Select the Arc GPU:

```bash
export GGML_VK_VISIBLE_DEVICES=0
```

### Rebuild Cache

Delete the build directory and rebuild:

```bash
cd ~/apps/llama-cpp

rm -rf build

cmake -B build \
    -DGGML_VULKAN=ON \
    -DCMAKE_BUILD_TYPE=Release

cmake --build build -j
```

---

# Optional: Build llama.cpp from Source

*Note* I recommend starting with the prebuilt binary. My prebuilt binary installation was broken before I realized I had already built llama.cpp from source

Use this section if you want:
* newest development version
* Vulkan backend compiled locally
* custom build options
* latest fixes not yet available in prebuilt releases


## 1. Install Build Dependencies

```bash
sudo apt update

sudo apt install -y \
    git \
    cmake \
    build-essential \
    libvulkan-dev \
    vulkan-tools \
    glslang-tools
```

## 2. Verify Vulkan Compiler (glslc)

Some Ubuntu releases may not provide a working `glslc` binary through `glslang-tools` and i had to download and install it!

Check:

```bash
which glslc
glslc --version
```

If `glslc` is missing, install the Vulkan SDK from LunarG and source its environment:

```bash
mkdir -p ~/vulkan
cd ~

wget https://sdk.lunarg.com/sdk/download/1.4.350.1/linux/vulkansdk-linux-x86_64-1.4.350.1.tar.xz

tar xf vulkansdk-linux-x86_64-1.4.350.1.tar.xz -C ~/vulkan

source ~/vulkan/1.4.350.1/setup-env.sh
```

Verify:

```bash
which glslc
glslc --version
```

Optional: make the environment permanent

```bash
echo 'source ~/vulkan/1.4.350.1/setup-env.sh' >> ~/.bashrc
source ~/.bashrc
```
* source ~/.bashrc requieres a restart of your current session to take effect

## 3. Clone Repository

```bash
cd ~/apps

rm -rf llama.cpp

git clone https://github.com/ggml-org/llama.cpp.git

cd llama.cpp
```

## 4. Build with Vulkan

```bash
rm -rf build

cmake -B build -DGGML_VULKAN=ON -DCMAKE_BUILD_TYPE=Release

cmake --build build -j
```

## 5. Verify Build

```bash
./build/bin/llama-server --version
```

**Expected Output  (similar):**
```text
version: 9436 (d6588daa8)
built with GNU 15.2.0 for Linux x86_64
```

### 5.1 Verify Vulkan Device Detection

```bash
./build/bin/llama-server --list-devices
```

**Expected Output (similar):**

```text
Available devices:
  Vulkan0: Intel(R) Graphics (BMG G31)
  Vulkan1: Intel(R) Graphics (RKL GT1)
```

## 6. Create Symlink (Convenience Shortcut)

```bash
sudo ln -sf ~/apps/llama-cpp/build/bin/llama-server /usr/local/bin/llama-server
```

Verify:

```bash
cd ~

llama-server --version

which llama-server
```

**Expected Output (similar):**

```text
version: 9436 (d6588daa8)
built with GNU 15.2.0 for Linux x86_64

/usr/local/bin/llama-server
```

---




