# LLMAvatarTalk: Quick Start Guide

This is a condensed setup guide for experienced users. For detailed step-by-step instructions, see the [Complete Setup Guide](COMPLETE_SETUP_GUIDE.md).

## Prerequisites Checklist

### Hardware Requirements ✓
- [ ] NVIDIA RTX 3070+ (8GB+ VRAM)
- [ ] 32GB+ RAM  
- [ ] 500GB+ free SSD space
- [ ] Stable internet connection

### Accounts Required ✓
- [ ] NVIDIA Developer Account ([register here](https://developer.nvidia.com/))
- [ ] NVIDIA NIMs API Key ([get here](https://build.nvidia.com/))
- [ ] NGC API Key ([get here](https://ngc.nvidia.com/))
- [ ] Epic Games Account ([register here](https://www.epicgames.com/))

## Quick Installation Steps

### 1. Environment Setup (15 minutes)
```bash
# Install Python 3.9
# Windows: Download from python.org
# Ubuntu: sudo apt install python3.9 python3.9-venv python3.9-dev

# Install NVIDIA drivers and CUDA 12.0+
# Verify: nvidia-smi && nvcc --version

# Clone project
git clone https://github.com/VisionaryArchitects/LLMAvatarTalk-An-Interactive-AI-Assistant.git
cd LLMAvatarTalk-An-Interactive-AI-Assistant

# Setup Python environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

### 2. RIVA Server Setup (30-60 minutes)
```bash
# Install Docker and NVIDIA Container Toolkit
# Ubuntu example:
sudo apt install docker.io
sudo apt install nvidia-container-toolkit

# Install NGC CLI
wget https://api.ngc.nvidia.com/v2/resources/nvidia/ngc-apps/ngc_cli/versions/3.43.0/files/ngccli_linux.zip
unzip ngccli_linux.zip && chmod +x ngc-cli/ngc
export PATH="$PATH:$(pwd)/ngc-cli"

# Configure NGC
ngc config set  # Enter your NGC API key

# Download and setup RIVA
mkdir ~/riva-setup && cd ~/riva-setup
ngc registry resource download-version "nvidia/riva/riva_quickstart:2.15.1"
cd riva_quickstart_v2.15.1

# Edit config.sh:
# - service_port_grpc=50051
# - enable_asr=true, enable_tts=true
# - Set your language (en-US or zh-CN)

# Initialize and start (takes 15-45 minutes)
sudo bash riva_init.sh
sudo bash riva_start.sh

# Verify: docker ps | grep riva
```

### 3. Audio2Face Setup (20 minutes)
```bash
# Download and install Omniverse Launcher
# From: https://www.nvidia.com/omniverse/download/

# In Omniverse Launcher:
# 1. Go to EXCHANGE → Search "Audio2Face" → Install
# 2. Launch Audio2Face
# 3. Open sample project: solved_arkit.usd
# 4. Add Streaming Audio Player: Audio2Face Tool → Audio Player → + → Streaming Audio Player
# 5. Connect in Graph: player_streaming_instance.time → Audio2Face Core.time
# 6. Note the Prim Path: /World/audio2face/PlayerStreaming
```

### 4. Unreal Engine Setup (30 minutes)
```bash
# Install Epic Games Launcher from https://www.epicgames.com/store/

# In Epic Games Launcher:
# 1. Unreal Engine → Library → + → Install UE 5.3.x (NOT 5.4+)
# 2. Create MetaHuman at https://metahuman.unrealengine.com/ (set to UE 5.3)
# 3. Create new UE project: Games → Third Person → Blueprint
# 4. Import MetaHuman character to project

# Install Audio2Face Plugin:
# 1. Copy ACE plugin from: [Audio2Face-Install]/ue-plugins/audio2face-ue-plugins/ACEUnrealPlugin-5.3/ACE
# 2. Paste to: [UE-Project]/Plugins/ACE/
# 3. Restart UE → Edit → Plugins → Enable Audio2Face plugin

# Configure Live Link:
# 1. Window → Virtual Production → Live Link
# 2. + → Audio2Face LiveLink
# 3. Configure: localhost:12030, Audio Port: 12031, Sample Rate: 22050 Hz
# 4. Select Audio2Face subject → Use ARKit Face
```

### 5. LLMAvatarTalk Configuration (5 minutes)
```bash
# Configure environment
cp .env.sample .env
# Edit .env: NVIDIA_API_KEY=nvapi-your-key-here

# Configure settings in config.py:
URI = 'localhost:50051'  # RIVA server
LANGUAGE = 'en-US'       # or 'zh-CN'

# Configure Audio2Face connection in modules/audio2face.py:
self.a2f_url = 'localhost:50051'
self.avatar_instance = '/World/audio2face/PlayerStreaming'  # Your prim path
```

## Quick Test Procedure

### Test Individual Components:
```bash
# Test RIVA connection
python -c "import riva.client; riva.client.Auth(uri='localhost:50051'); print('RIVA OK')"

# Test NVIDIA NIMs
python -c "from langchain_nvidia_ai_endpoints import ChatNVIDIA; from dotenv import load_dotenv; load_dotenv(); ChatNVIDIA(model='meta/llama3-70b-instruct').invoke('Hello'); print('NIMs OK')"

# Test Audio2Face (with Audio2Face running)
python -c "from modules.audio2face import Audio2FaceService; Audio2FaceService(); print('Audio2Face OK')"
```

### Full System Test:
```bash
# 1. Start RIVA: sudo bash riva_start.sh
# 2. Start Audio2Face with streaming player configured
# 3. Start Unreal Engine with MetaHuman and Live Link connected
# 4. Run: python main.py
# 5. Speak into microphone → Should see facial animation in UE
```

## Troubleshooting Quick Fixes

### RIVA Issues:
```bash
# Check status: docker ps | grep riva
# Restart: sudo bash riva_stop.sh && sudo bash riva_start.sh
# Check logs: docker logs riva-speech-api
```

### Audio2Face Issues:
```bash
# Verify gRPC: python -c "import grpc; grpc.insecure_channel('localhost:50051')"
# Check prim path matches configuration
# Restart Audio2Face and reconfigure streaming player
```

### UE Live Link Issues:
```bash
# Verify plugin enabled: Edit → Plugins → Search "Audio2Face"
# Check connection: Live Link window should show "Connected" status
# Restart both Audio2Face and UE if connection fails
```

### Performance Issues:
```bash
# Monitor GPU: nvidia-smi -l 1
# Reduce UE quality: Edit → Project Settings → Rendering
# Check Audio2Face quality settings
```

## Expected Workflow

1. **User speaks** → Microphone captures audio
2. **ASR (RIVA)** → Converts speech to text
3. **LLM (NVIDIA NIMs)** → Generates response
4. **TTS (RIVA)** → Converts response to audio
5. **Audio2Face** → Creates facial animation from audio
6. **Unreal Engine** → Displays animated MetaHuman
7. **Audio playback** → User hears response

## Performance Benchmarks

### Expected Latency:
- **ASR**: 100-500ms
- **LLM**: 1-3 seconds
- **TTS**: 200-800ms  
- **Audio2Face**: 50-100ms
- **Total**: 2-5 seconds end-to-end

### Resource Usage:
- **GPU Memory**: 6-12GB (depends on models and quality)
- **System RAM**: 16-32GB
- **CPU**: Moderate usage across all cores

## Success Criteria

✅ **RIVA**: Docker containers running, can transcribe speech and synthesize voice  
✅ **Audio2Face**: Loads sample project, streaming player configured and connected  
✅ **Unreal Engine**: MetaHuman character loaded, Live Link shows "Connected"  
✅ **Integration**: Speaking results in real-time facial animation  
✅ **Quality**: Lip sync is accurate, expressions are natural  

## Support Resources

- **Detailed Guide**: [Complete Setup Guide](COMPLETE_SETUP_GUIDE.md)
- **Component Guides**: [RIVA](RIVA/RIVA_Tutorial.md) | [Audio2Face](Audio2Face/Audio2Face_Tutorial_Enhanced.md) | [Unreal Engine](UE/UE_Tutorial_Enhanced.md)
- **Troubleshooting**: See individual component guides
- **Performance**: Monitor with `nvidia-smi`, UE stats, and Audio2Face metrics

**Estimated total setup time**: 2-4 hours for first-time setup, 30-60 minutes for experienced users.