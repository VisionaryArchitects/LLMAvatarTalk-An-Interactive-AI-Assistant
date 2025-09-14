# RIVA Setup Guide

NVIDIA RIVA is a GPU-accelerated SDK for building speech AI applications that require real-time performance. It provides state-of-the-art models for automatic speech recognition (ASR), text-to-speech (TTS), and natural language processing (NLP). By leveraging RIVA, developers can create applications with high accuracy and low latency, making it ideal for interactive AI assistants like LLMAvatarTalk.

## System Requirements

### Hardware Requirements
- **GPU**: NVIDIA GPU with compute capability 7.0+ (RTX 20 series or newer)
- **RAM**: Minimum 16GB, recommended 32GB
- **Storage**: 50GB free space for models and containers
- **Network**: Stable internet connection for downloading models

### Software Requirements
- **OS**: Ubuntu 20.04/22.04 LTS (recommended) or Windows 10/11 with WSL2
- **Docker**: Version 20.10+
- **NVIDIA Container Toolkit**: Latest version
- **NVIDIA Driver**: 470+ 

**Note**: This guide covers setup on Ubuntu. WSL2 Ubuntu 20.04.6 LTS is tested and working.

## Step-by-Step Guide

### Step 1: Install Prerequisites

#### 1.1 Install Docker
```bash
# Update system packages
sudo apt update
sudo apt upgrade -y

# Install Docker dependencies
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker $USER

# Restart session or run:
newgrp docker

# Verify Docker installation
docker --version
```

#### 1.2 Install NVIDIA Container Toolkit
```bash
# Add NVIDIA GPG key
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Install NVIDIA Container Toolkit
sudo apt update
sudo apt install -y nvidia-container-toolkit

# Restart Docker service
sudo systemctl restart docker

# Test NVIDIA Docker integration
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

### Step 2: Install and Configure NGC CLI

#### 2.1 Download NGC CLI
```bash
# Create directory for NGC CLI
mkdir -p ~/tools/ngc-cli
cd ~/tools/ngc-cli

# Download NGC CLI (latest version)
wget --content-disposition https://api.ngc.nvidia.com/v2/resources/nvidia/ngc-apps/ngc_cli/versions/3.43.0/files/ngccli_linux.zip -O ngccli_linux.zip

# Extract the archive
unzip ngccli_linux.zip
```

#### 2.2 Verify and Install NGC CLI
```bash
# Verify MD5 hash (optional but recommended)
find ngc-cli/ -type f -exec md5sum {} + | LC_ALL=C sort | md5sum -c ngc-cli.md5

# Make the binary executable
chmod u+x ngc-cli/ngc

# Add to PATH permanently
echo "export PATH=\"\$PATH:$HOME/tools/ngc-cli/ngc-cli\"" >> ~/.bashrc
source ~/.bashrc

# Verify installation
ngc version
```

#### 2.3 Configure NGC CLI
```bash
# Configure NGC CLI
ngc config set
```

**When prompted, enter the following:**
- **API Key**: Enter your NGC API key (get from [NGC Portal](https://ngc.nvidia.com/))
  - Log in to NGC Portal → Setup → API Key → Generate API Key
- **CLI output format**: Choose `ascii` (recommended)
- **org**: Leave blank for personal use
- **team**: Leave blank
- **ace**: Leave blank

**Important Notes:**
- Your NGC API key is different from your NVIDIA NIMs API key
- Keep your API key secure and never share it
- You can regenerate your API key if needed, but it will invalidate the old one

### Step 3: Download RIVA QuickStart

#### 3.1 Create RIVA Directory
```bash
# Create dedicated directory for RIVA
mkdir -p ~/riva-setup
cd ~/riva-setup
```

#### 3.2 Download RIVA QuickStart
```bash
# Download the latest RIVA QuickStart package
ngc registry resource download-version "nvidia/riva/riva_quickstart:2.15.1"

# Navigate to the downloaded directory
cd riva_quickstart_v2.15.1

# List contents to verify download
ls -la
```

**Expected files:**
- `config.sh` - Configuration file
- `riva_init.sh` - Initialization script
- `riva_start.sh` - Server start script
- `riva_stop.sh` - Server stop script
- Various other utility scripts

### Step 4: Configure RIVA Settings

#### 4.1 Edit Configuration File
```bash
# Make a backup of the original config
cp config.sh config.sh.backup

# Edit the configuration file
nano config.sh  # or use vim, gedit, etc.
```

#### 4.2 Key Configuration Settings

**Basic Service Configuration:**
```bash
# Enable required services
enable_asr=true           # Speech recognition
enable_tts=true           # Text-to-speech
enable_nlp=false          # Natural language processing (not needed)
enable_nmt=false          # Neural machine translation (not needed)

# Service networking
service_port_grpc=50051   # Change from default 50050 to match LLMAvatarTalk config
service_host=0.0.0.0      # Allow connections from any interface
```

**GPU Configuration:**
```bash
# GPU settings
gpu_device_ids=0          # Use first GPU (adjust if you have multiple GPUs)
gpu_memory_pool_size=2048 # Adjust based on available GPU memory (MB)
```

**Language and Model Configuration:**
```bash
# For English (default)
asr_language_code=en-US
tts_language_code=en-US
tts_voice_names="English-US.Female-1,English-US.Male-1"

# For Chinese (uncomment if needed)
# asr_language_code=zh-CN
# tts_language_code=zh-CN
# tts_voice_names="Mandarin-CN.Female-1,Mandarin-CN.Male-1"
```

**Model Configuration (Advanced):**
```bash
# ASR model settings
use_streaming_asr_audio_setup=true
asr_acoustic_model="riva_asr_citrinet_1024_asrset_1.0.0"

# TTS model settings
tts_enable_ssml=true      # Enable SSML support
```

#### 4.3 Save Configuration
- Save the file (Ctrl+O in nano, then Enter, then Ctrl+X to exit)
- Verify your changes:
```bash
grep -E "(enable_|service_port|gpu_device)" config.sh
```

### Step 5: Initialize and Start RIVA

#### 5.1 Initialize RIVA
```bash
# Initialize RIVA (downloads models and sets up containers)
# This step takes 15-45 minutes depending on internet speed
sudo bash riva_init.sh

# Monitor progress
# The script will download several GB of models
# You'll see progress indicators for each model download
```

**What happens during initialization:**
- Downloads ASR and TTS models
- Sets up Docker containers
- Configures model pipelines
- Prepares the RIVA service

#### 5.2 Start RIVA Server
```bash
# Start the RIVA server
sudo bash riva_start.sh

# Monitor startup process
# The server takes 1-3 minutes to fully start
```

#### 5.3 Verify RIVA is Running
```bash
# Check Docker containers
docker ps

# You should see containers like:
# - riva-speech-api (main service)
# - tritonserver (inference engine)

# Check service logs
docker logs riva-speech-api

# Test basic connectivity
curl -X POST http://localhost:50051/v1/health
```

### Step 6: Test RIVA Installation

#### 6.1 Test ASR (Speech Recognition)
```bash
# Navigate to RIVA directory
cd ~/riva-setup/riva_quickstart_v2.15.1

# Test ASR with sample audio
python scripts/asr/transcribe_file.py \
    --server=localhost:50051 \
    --input-file=data/samples/en-US_sample.wav \
    --language-code=en-US
```

#### 6.2 Test TTS (Text-to-Speech)
```bash
# Test TTS with sample text
python scripts/tts/talk.py \
    --server=localhost:50051 \
    --text="Hello, this is a test of RIVA text to speech." \
    --voice="English-US.Female-1" \
    --output=test_output.wav

# Play the generated audio (if audio system is available)
# aplay test_output.wav  # On Linux with ALSA
# Or copy to Windows and play there
```

#### 6.3 Test Python Client Connection
```bash
# Test Python client connectivity
python3 -c "
import riva.client
auth = riva.client.Auth(uri='localhost:50051')
asr_service = riva.client.ASRService(auth)
print('✓ RIVA ASR connection successful')

tts_service = riva.client.SpeechSynthesisService(auth)
print('✓ RIVA TTS connection successful')
print('✓ RIVA setup complete and ready for LLMAvatarTalk!')
"
```

### Step 7: Server Management

#### 7.1 Server Control Commands
```bash
# Stop RIVA server
sudo bash riva_stop.sh

# Start RIVA server
sudo bash riva_start.sh

# Restart RIVA server
sudo bash riva_stop.sh && sudo bash riva_start.sh

# Check server status
docker ps | grep riva
```

#### 7.2 Server Logs and Monitoring
```bash
# View real-time logs
docker logs -f riva-speech-api

# Check resource usage
nvidia-smi  # GPU usage
docker stats  # Container resource usage
```

#### 7.3 Update Configuration
```bash
# After changing config.sh, you must reinitialize:
sudo bash riva_stop.sh
sudo bash riva_init.sh  # Re-download/setup if needed
sudo bash riva_start.sh
```

### Troubleshooting

#### Common Issues and Solutions

**1. Docker Permission Denied**
```bash
# Add user to docker group and restart
sudo usermod -aG docker $USER
newgrp docker
# Or logout and login again
```

**2. GPU Not Detected**
```bash
# Verify GPU access in Docker
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi

# If this fails, reinstall NVIDIA Container Toolkit
sudo apt remove --purge nvidia-container-toolkit
# Then reinstall following Step 1.2
```

**3. Port Already in Use**
```bash
# Check what's using port 50051
sudo netstat -tulpn | grep 50051

# Kill process if needed
sudo kill -9 <PID>

# Or change port in config.sh
```

**4. Model Download Issues**
```bash
# Check internet connectivity
ping catalog.ngc.nvidia.com

# Verify NGC CLI authentication
ngc registry model list nvidia/riva/rmir*

# Re-run initialization if download failed
sudo bash riva_init.sh
```

**5. Service Won't Start**
```bash
# Check detailed logs
docker logs riva-speech-api

# Common fixes:
# - Ensure sufficient GPU memory
# - Check config.sh for syntax errors
# - Verify all required models downloaded
```

### Performance Optimization

#### For Different Hardware Configurations:

**High-end Systems (RTX 4090, A6000):**
```bash
# In config.sh:
gpu_memory_pool_size=4096
max_batch_size=16
```

**Mid-range Systems (RTX 3070, RTX 4070):**
```bash
# In config.sh:
gpu_memory_pool_size=2048
max_batch_size=8
```

**Entry-level Systems (RTX 3060):**
```bash
# In config.sh:
gpu_memory_pool_size=1024
max_batch_size=4
```

Your RIVA server is now ready for integration with LLMAvatarTalk!
