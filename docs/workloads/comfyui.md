# ComfyUI Installation Guide

## Step-by-Step Setup

>**Important:** Before proceeding, check the README.md file for details on the specific Ubuntu distribution, kernel versions and other prerequisites used in this guide.

## Prerequisites

a) Base Driver
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

b) **OpenCL + Level Zero support**:

It is based on a working Intel GPU compute stack consisting of:
- OpenCL Runtime
- Level Zero Runtime
- Intel Kernel Driver (i915 / xe)
- optional: oneAPI Toolkit (`sycl-ls` for verification)

Setup Guide here: [docs/02-opencl-level-zero.md]


### 1. Create Project Directory and Activate Virtual Environment

```bash
mkdir ~/comfyui-arc && cd ~/comfyui-arc
python3 -m venv venv
source venv/bin/activate
```

### 2. Install Required Packages

```bash
pip install comfy-cli
comfy install --fast-deps
```

### 3. Launch ComfyUI with Optimized Settings

```bash
comfy launch -- --force-fp16 --highvram --use-pytorch-cross-attention
```
* Launches ComfyUI with settings optimized for high VRAM usage and performance .

### 4. Install Custom PyTorch Versions

If you already installed PyTorch uninstall existing PyTorch packages 
```bash
pip uninstall -y torch torchvision torchaudio intel-extension-for-pytorch
```

Installs Intel-optimized versions for Arc Pro
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu
pip install intel-extension-for-pytorch --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/
```


### 5. Install Remaining Dependencies

```bash
pip install -r ~/comfy/ComfyUI/requirements.txt
```

### 6. Set Up Intel OneAPI Environment

```bash
source /opt/intel/oneapi/setvars.sh 2>/dev/null || true
```

### 7. Launch ComfyUI 

```bash
comfy launch -- --force-fp16 --highvram --use-pytorch-cross-attention
```

or with --use-xpu
```bash
comfy launch -- --use-xpu --force-fp16 --highvram --use-pytorch-cross-attention
```

### 7.1 Open ComfyUI

http://yourIP:8188/ or http://127.0.0.1:8188/ or localhost:8188/


# Convenience Tips & Troubleshooting 

## Accessing ComfyUI Remotely via SSH Tunnel (optional)

a) Configure SSH Tunnel in PuTTY

* Go to: `Connection → SSH → Tunnels`
* Enter:
    - Source port: `8188`
    - Destination: `127.0.0.1:8188`

b) Adapt ComfyUI command to listen on all network interfaces

```bash
comfy launch -- --listen 0.0.0.0 --force-fp16 --highvram --use-pytorch-cross-attention
```

Access via: `http://yourIP:8188/` (replace IP with your server's IP address).

c) Opening Port for External Access
Allow Port 8188 in UFW (Uncomplicated Firewall)

```bash
sudo ufw allow 8188
```

## Moving ComfyUI Directory


>  Note: Models can be huge -> follow the Guide to move the model directory

### Stop ComfyUI, Move Directory and Symlink

```bash
# Stop ComfyUI
comfy launch --stop # or CTRL+C

# Move directory
mv ~/comfyui-arc /data/comfyui-arc

# Create symlink for convenience (optional)
ln -s /data/comfyui-arc ~/comfyui-arc
```

### Move Model Files

> Note: Checkpoints Directory in only one of many subfolders, where comfyui model files are saved!

```bash
mv ~/Downloads/v1-5-pruned-emaonly-fp16.safetensors ~/comfy/ComfyUI/models/checkpoints/
```


## Setting Up as a Systemd Service

### Systemd Service (ComfyUI standalone)

Create a service file:

```bash
sudo nano /etc/systemd/system/comfyui.service
```

Add the following content to the file:

```ini
[Unit]
Description=ComfyUI
After=network.target

[Service]
Type=simple
User=youruser  # Replace with your actual username
WorkingDirectory=/home/youruser/comfyui-arc
Environment="PATH=/home/youruser/comfyui-arc/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
ExecStart=/home/youruser/comfyui-arc/venv/bin/comfy launch -- --listen 0.0.0.0 --force-fp16 --highvram --use-pytorch-cross-attention
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Replace `youruser` with your actual username.

### Reload Systemd and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable comfyui.service
sudo systemctl start comfyui.service
```

### Systemd Service (ComfyUI in parallel with Ollama)
> Important: if you use comfyui and ollama in parallel, you might run into vram issues. This is why you should switch beetween both tools

**Create ComfyUI service**
```bash
sudo nano /etc/systemd/system/comfyui.service
```

Add the following content to the file:

```ini
[Unit]
Description=ComfyUI
After=network.target

Conflicts=ollama-stack.service		# IMPORTANT TO SWITCH WITH OLLAMA
AssertInactive=ollama-stack.service	# IMPORTANT TO SWITCH WITH OLLAMA

[Service]
Type=simple
User=youruser  # Replace with your actual username
WorkingDirectory=/home/youruser/comfyui-arc
Environment="PATH=/home/youruser/comfyui-arc/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"
ExecStart=/home/youruser/comfyui-arc/venv/bin/comfy launch -- --listen 0.0.0.0 --force-fp16 --highvram --use-pytorch-cross-attention
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Replace `youruser` with your actual username.

** Ollama service **
add
`Conflicts=comfyui.service`
`AssertInactive=comfyui.service`
to your ollama Service file

Example:
```bash
cat /etc/systemd/system/ollama-stack.service
```

```ini
[Unit]
Description=AI Ollama Stack (Docker Compose)
After=docker.service
Requires=docker.service

Conflicts=comfyui.service
AssertInactive=comfyui.service

[Service]
Type=oneshot
RemainAfterExit=yes

WorkingDirectory=/home/youruser/ollama-arc

ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

