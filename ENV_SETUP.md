# Environment Configuration Guide

## Overview

The docker-compose.yaml now uses environment variables to avoid committing personal directory paths to git.

## Files

- **`.env`** - Your personal configuration (NOT committed to git)
- **`.env.example`** - Template for new setups (committed to git)
- **`.gitignore`** - Excludes .env from git

## Setup

### First Time Setup

1. Copy the example file:
   ```bash
   cp .env.example .env
   ```

2. Edit `.env` with your actual paths:
   ```bash
   # Main instance directories
   COMFY_MODELS_DIR=D:\your\path\to\models
   COMFY_CUSTOM_NODES_DIR=D:\your\path\to\custom_nodes
   COMFY_OUTPUT_DIR=D:\your\path\to\output

   # Second instance (optional)
   COMFY2_MODELS_DIR=D:\your\path\to\models
   COMFY2_CUSTOM_NODES_DIR=D:\your\path\to\custom_nodes
   COMFY2_OUTPUT_DIR=D:\your\path\to\output2
   COMFY2_PORT=8189
   ```

3. Start ComfyUI:
   ```bash
   docker-compose up -d
   ```

## Running Multiple Instances

### Single Instance (Default)
```bash
docker-compose up -d
```
- Runs only ComfyUI on port 8188
- Access at: http://localhost:8188

### Two Instances
```bash
docker-compose --profile multi-instance up -d
```
- Runs both ComfyUI and ComfyUI2
- ComfyUI: http://localhost:8188
- ComfyUI2: http://localhost:8189 (or custom port from COMFY2_PORT)

### Managing Individual Instances
```bash
# Start only main instance
docker-compose up -d ComfyUI

# Start only second instance
docker-compose --profile multi-instance up -d ComfyUI2

# Stop specific instance
docker-compose stop ComfyUI2

# View logs for specific instance
docker-compose logs -f ComfyUI2
```

## Environment Variables

### Main Instance
| Variable | Description | Example |
|----------|-------------|---------|
| `COMFY_MODELS_DIR` | Path to models directory | `D:\comfy\models` |
| `COMFY_CUSTOM_NODES_DIR` | Path to custom nodes | `D:\comfy\custom_nodes` |
| `COMFY_OUTPUT_DIR` | Path to output directory | `D:\comfy\output` |

### Second Instance (Optional)
| Variable | Description | Example |
|----------|-------------|---------|
| `COMFY2_MODELS_DIR` | Path to models directory | `D:\comfy\models` |
| `COMFY2_CUSTOM_NODES_DIR` | Path to custom nodes | `D:\comfy\custom_nodes` |
| `COMFY2_OUTPUT_DIR` | Path to output directory | `D:\comfy\output2` |
| `COMFY2_PORT` | Host port mapping | `8189` |

## Common Configurations

### Shared Models, Separate Outputs
```env
# .env
COMFY_MODELS_DIR=D:\comfy\models
COMFY_CUSTOM_NODES_DIR=D:\comfy\custom_nodes
COMFY_OUTPUT_DIR=D:\projects\project-a\output

COMFY2_MODELS_DIR=D:\comfy\models              # Same models
COMFY2_CUSTOM_NODES_DIR=D:\comfy\custom_nodes  # Same nodes
COMFY2_OUTPUT_DIR=D:\projects\project-b\output # Different output
COMFY2_PORT=8189
```

### Completely Isolated Instances
```env
# .env
COMFY_MODELS_DIR=D:\comfy\instance1\models
COMFY_CUSTOM_NODES_DIR=D:\comfy\instance1\custom_nodes
COMFY_OUTPUT_DIR=D:\comfy\instance1\output

COMFY2_MODELS_DIR=D:\comfy\instance2\models
COMFY2_CUSTOM_NODES_DIR=D:\comfy\instance2\custom_nodes
COMFY2_OUTPUT_DIR=D:\comfy\instance2\output
COMFY2_PORT=8189
```

## Troubleshooting

### "bind source path does not exist"
The directory specified in .env doesn't exist. Create it first:
```bash
mkdir -p "D:\path\to\directory"
```

### Changes to .env not taking effect
Restart the containers:
```bash
docker-compose down
docker-compose up -d
```

### Second instance won't start
Make sure you're using the `--profile multi-instance` flag:
```bash
docker-compose --profile multi-instance up -d
```

### Port conflict
Change `COMFY2_PORT` in your `.env` file to an available port.

## Git Best Practices

✅ **DO** commit:
- `.env.example`
- `docker-compose.yaml`
- `.gitignore`

❌ **DON'T** commit:
- `.env` (contains your personal paths)
- `models/` directory
- `.claude/` directory

The `.gitignore` file is configured to prevent accidentally committing these files.
