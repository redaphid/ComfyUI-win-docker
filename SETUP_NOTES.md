# ComfyUI Windows Container Setup

## What We're Doing

Setting up **ComfyUI** (a powerful and modular Stable Diffusion GUI) to run inside a **Windows Docker container** with GPU support.

## Why Windows Containers?

- ComfyUI has a Windows portable version that includes all dependencies
- Windows containers allow GPU passthrough for NVIDIA GPUs
- Isolates the ComfyUI environment from the host system
- Makes it easy to update and manage versions

## Repository Details

- **Repo**: https://github.com/redaphid/ComfyUI-win-docker
- **Docker Image**: `eisai/comfy-ui` on Docker Hub
- **Base Image**: `mcr.microsoft.com/windows/server:ltsc2022`

## System Requirements

### Required Windows Features
Windows containers require two Windows features to be enabled:

1. **Hyper-V** (`Microsoft-Hyper-V-All`)
   - Provides virtualization platform
   - Required for container isolation

2. **Containers** (`Containers`)
   - Windows container runtime support
   - Enables Docker to run Windows containers

### Hardware Requirements
- Windows 10/11 Pro, Enterprise, or Education (or Windows Server)
- CPU with virtualization support (Intel VT-x or AMD-V)
- NVIDIA GPU (for GPU acceleration)
- Sufficient RAM (16GB+ recommended for Stable Diffusion)

## What We've Done So Far

### 1. Cloned the Repository
```bash
git clone https://github.com/redaphid/ComfyUI-win-docker.git
cd ComfyUI-win-docker
```

### 2. Checked Docker Installation
- Docker Desktop is installed (version 28.4.0)
- Docker was initially in Linux container mode (needs to be Windows mode)

### 3. Enabled Required Windows Features

**Checked current feature status:**
```powershell
Get-WindowsOptionalFeature -Online | Where FeatureName -match 'Hyper|Container'
```

**Enabled Hyper-V:**
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All -All -NoRestart
```

**Enabled Containers:**
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Containers -All -NoRestart
```

Both features were successfully enabled but require a system restart to take effect.

## What Happens Next

### 1. System Restart (REQUIRED)
**You must restart your computer** for Hyper-V and Containers to become active.

### 2. Switch Docker to Windows Container Mode
After restart, Docker needs to be switched from Linux to Windows containers:
```powershell
& 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon
```

Or right-click the Docker Desktop system tray icon and select "Switch to Windows containers..."

### 3. Verify Windows Container Mode
```bash
docker info --format '{{.OSType}}'
# Should output: windows
```

### 4. Create Models Directory
The docker-compose.yaml maps a local `models` directory to the container:
```bash
cd ComfyUI-win-docker
mkdir models
```

This is where you'll store:
- Stable Diffusion checkpoints/models
- VAE files
- LoRA models
- Embeddings
- Etc.

### 5. Pull the Docker Image
Two options:

**Option A - Pull from Docker Hub (easier):**
```bash
docker pull eisai/comfy-ui:latest
```

**Option B - Build locally (if you want the latest ComfyUI):**
```powershell
./build.ps1
```

### 6. Start ComfyUI
```bash
docker-compose up -d
```

This will:
- Start the ComfyUI container
- Map port 8188 to your host
- Mount the `models` directory
- Pass through your NVIDIA GPU

### 7. Access ComfyUI
Open your browser to:
```
http://localhost:8188
```

## How It Works

### Docker Compose Configuration
The `docker-compose.yaml` file configures:

```yaml
services:
  ComfyUI:
    container_name: ComfyUI
    image: eisai/comfy-ui
    isolation: process           # Windows container isolation mode
    ports:
      - 8188:8188               # Web UI port
    volumes:
      - '.\models:C:\app\ComfyUI\models'  # Model storage
    devices:
      - class/5B45201D-F2F2-4F3B-85BB-30FF1F953599  # GPU passthrough
```

### Container Entry Point (`entry.bat`)
When the container starts:

1. **Syncs models**: Copies default models from `_models` to `models` (first run)
2. **Starts ComfyUI**: Launches Python with ComfyUI's main.py
   - Listens on `0.0.0.0:8188` (accessible from host)
   - Runs in standalone build mode
   - Auto-launch disabled (no browser popup in container)

### Dockerfile Process
The image is built by:

1. Starting from Windows Server LTSC 2022
2. Downloading ComfyUI Windows portable from GitHub releases
3. Extracting the portable version
4. Installing NVIDIA CUDA DLLs and Visual C++ redistributables
5. Setting up the entry point

## Troubleshooting

### Docker Not Running
```bash
# Start Docker Desktop
Start-Process 'C:\Program Files\Docker\Docker\Docker Desktop.exe'
```

### Still in Linux Mode After Restart
Right-click Docker Desktop system tray icon → "Switch to Windows containers..."

### GPU Not Working
Check that your NVIDIA drivers are up to date and compatible with CUDA containers.

### Models Not Loading
Make sure you've placed model files in the `models` subdirectories:
- `models/checkpoints/` - SD checkpoints
- `models/vae/` - VAE files
- `models/loras/` - LoRA files
- etc.

## Useful Commands

```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs -f

# Stop ComfyUI
docker-compose down

# Restart ComfyUI
docker-compose restart

# Access container shell
docker exec -it ComfyUI cmd
```

## Current Status

- [x] Docker installed
- [x] Repository cloned
- [x] Hyper-V enabled
- [x] Containers feature enabled
- [ ] **SYSTEM RESTART REQUIRED**
- [ ] Switch to Windows container mode
- [ ] Create models directory
- [ ] Pull/build image
- [ ] Start ComfyUI
- [ ] Access web UI

**Next action: Restart your computer**
