# LLMAvatarTalk: Complete Setup Guide

## Table of Contents
1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Prerequisites and Account Setup](#prerequisites-and-account-setup)
4. [Phase 1: Environment Setup](#phase-1-environment-setup)
5. [Phase 2: NVIDIA RIVA Server Installation](#phase-2-nvidia-riva-server-installation)
6. [Phase 3: NVIDIA Omniverse and Audio2Face Setup](#phase-3-nvidia-omniverse-and-audio2face-setup)
7. [Phase 4: Unreal Engine and MetaHuman Setup](#phase-4-unreal-engine-and-metahuman-setup)
8. [Phase 5: LLMAvatarTalk Project Configuration](#phase-5-llmavatartalk-project-configuration)
9. [Phase 6: Integration and Testing](#phase-6-integration-and-testing)
10. [Troubleshooting](#troubleshooting)
11. [Advanced Configuration](#advanced-configuration)

---

## Overview

LLMAvatarTalk is an interactive AI assistant that combines multiple cutting-edge technologies:
- **ASR (Automatic Speech Recognition)**: NVIDIA RIVA for real-time speech-to-text
- **LLM (Large Language Model)**: NVIDIA NIMs API for intelligent conversation
- **TTS (Text-to-Speech)**: NVIDIA RIVA for natural voice synthesis  
- **Audio2Face**: NVIDIA Omniverse for audio-driven facial animation
- **MetaHuman**: Unreal Engine for photorealistic 3D avatar rendering

This guide provides a complete, step-by-step process to set up the entire system from scratch.

---

## System Requirements

### Minimum Hardware Requirements
- **CPU**: Intel i7-8700K / AMD Ryzen 7 2700X or better
- **GPU**: NVIDIA RTX 3070 / RTX A4000 or better (minimum 8GB VRAM)
- **RAM**: 32GB DDR4 (recommended 64GB for optimal performance)
- **Storage**: 500GB free SSD space (1TB recommended)
- **Network**: Stable internet connection (minimum 100 Mbps for streaming)

### Software Requirements
- **Operating System**: 
  - Windows 10/11 (64-bit) - Primary recommended
  - Ubuntu 20.04/22.04 LTS - Alternative for RIVA server
- **Python**: 3.9.x (tested and verified)
- **NVIDIA Driver**: 535.xx or newer
- **CUDA Toolkit**: 12.0 or newer

### Network Requirements
- Multiple applications will run on different ports
- Ensure firewall allows communication between components
- Default ports used: 50050/50051 (RIVA), 50052 (Audio2Face gRPC)

---

## Prerequisites and Account Setup

### 1. NVIDIA Developer Account Setup

#### Step 1.1: Create NVIDIA Developer Account
1. Go to [NVIDIA Developer Portal](https://developer.nvidia.com/)
2. Click "Join Now" in the top right corner
3. Fill out the registration form with your details
4. Verify your email address
5. Complete your developer profile

#### Step 1.2: Get NVIDIA NIMs API Key
1. Navigate to [NVIDIA NIMs](https://build.nvidia.com/explore/discover)
2. Sign in with your NVIDIA Developer account
3. Click "Get API Key" or "Sign Up for Free Credits"
4. You'll receive 1000 free credits for testing
5. Copy your API key (format: `nvapi-xxxxxxxxx`)
6. **Important**: Save this key securely - you'll need it later

#### Step 1.3: NGC (NVIDIA GPU Cloud) Setup
1. Go to [NGC Portal](https://ngc.nvidia.com/)
2. Sign in with your NVIDIA Developer account
3. Click "Setup" in the left menu
4. Click "API Key" 
5. Click "Generate API Key"
6. **Warning**: This will invalidate any existing NGC API key
7. Click "CONTINUE" to generate
8. Copy and save your NGC API key

### 2. Epic Games Account Setup

#### Step 2.1: Create Epic Games Account
1. Go to [Epic Games](https://www.epicgames.com/)
2. Click "Sign In" then "Sign Up"
3. Create your account and verify email
4. Accept the terms of service

#### Step 2.2: Get Unreal Engine Access
1. Sign in to your Epic Games account
2. Go to [Unreal Engine](https://www.unrealengine.com/)
3. Click "Download Now"
4. Accept the Unreal Engine EULA

---

## Phase 1: Environment Setup

### Step 1.1: Install Python 3.9

#### On Windows:
1. Download Python 3.9.x from [python.org](https://www.python.org/downloads/release/python-3913/)
2. **Important**: Check "Add Python to PATH" during installation
3. Choose "Customize installation"
4. Check all optional features
5. Check "Add Python to environment variables"
6. Complete installation

#### On Ubuntu:
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.9 python3.9-venv python3.9-dev
sudo apt install python3-pip
```

#### Verify Installation:
```bash
python --version  # Should show Python 3.9.x
pip --version     # Should show pip version
```

### Step 1.2: Install Git
#### On Windows:
1. Download from [git-scm.com](https://git-scm.com/download/win)
2. Install with default settings
3. Choose "Git from the command line and also from 3rd-party software"

#### On Ubuntu:
```bash
sudo apt install git
```

### Step 1.3: Install Visual Studio Build Tools (Windows Only)
1. Download [Visual Studio Build Tools](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2022)
2. Install "C++ build tools" workload
3. This is required for compiling some Python packages

### Step 1.4: Install NVIDIA Drivers and CUDA

#### Install Latest NVIDIA Driver:
1. Go to [NVIDIA Driver Downloads](https://www.nvidia.com/drivers/)
2. Select your GPU model
3. Download and install the latest driver
4. Restart your computer

#### Install CUDA Toolkit:
1. Go to [CUDA Toolkit Downloads](https://developer.nvidia.com/cuda-downloads)
2. Select your OS and download CUDA 12.0+
3. Install with default settings
4. Add CUDA to PATH if not automatically added

#### Verify CUDA Installation:
```bash
nvidia-smi    # Should show GPU info and CUDA version
nvcc --version # Should show CUDA compiler version
```

---

## Phase 2: NVIDIA RIVA Server Installation

### Step 2.1: Install NGC CLI

#### On Windows:
1. Download NGC CLI from [NGC CLI Downloads](https://org.ngc.nvidia.com/setup/installers/cli)
2. Extract the ZIP file to `C:\ngc-cli\`
3. Add `C:\ngc-cli\` to your system PATH
4. Open new Command Prompt and verify: `ngc version`

#### On Ubuntu:
```bash
# Download and install NGC CLI
wget --content-disposition https://api.ngc.nvidia.com/v2/resources/nvidia/ngc-apps/ngc_cli/versions/3.43.0/files/ngccli_linux.zip -O ngccli_linux.zip
unzip ngccli_linux.zip

# Verify MD5 hash
find ngc-cli/ -type f -exec md5sum {} + | LC_ALL=C sort | md5sum -c ngc-cli.md5

# Make executable and add to PATH
chmod u+x ngc-cli/ngc
echo "export PATH=\"\$PATH:$(pwd)/ngc-cli\"" >> ~/.bashrc
source ~/.bashrc
```

### Step 2.2: Configure NGC CLI
```bash
ngc config set
```
When prompted, enter:
- **API Key**: Your NGC API key from Prerequisites
- **CLI output format**: ascii
- **org**: (leave blank for personal use)
- **team**: (leave blank)
- **ace**: (leave blank)

### Step 2.3: Download RIVA QuickStart

```bash
# Create RIVA directory
mkdir ~/riva-setup
cd ~/riva-setup

# Download RIVA QuickStart (latest version)
ngc registry resource download-version "nvidia/riva/riva_quickstart:2.15.1"

# Navigate to the downloaded directory
cd riva_quickstart_v2.15.1
```

### Step 2.4: Configure RIVA Settings

#### Edit config.sh:
```bash
# Open config.sh in your preferred editor
nano config.sh  # or vim config.sh on Linux
# notepad config.sh on Windows
```

#### Key configurations to modify:
```bash
# Set RIVA service port (default is 50050, you can change to 50051)
service_port_grpc=50051

# Enable ASR and TTS services
enable_asr=true
enable_tts=true
enable_nlp=false  # Not needed for this project

# Set GPU configuration
gpu_device_ids=0  # Use first GPU, adjust if you have multiple GPUs

# Set language models (adjust based on your needs)
# For English:
asr_language_code=en-US
tts_language_code=en-US

# For Chinese (if needed):
# asr_language_code=zh-CN
# tts_language_code=zh-CN
```

### Step 2.5: Initialize and Start RIVA

#### On Linux (Recommended):
```bash
# Initialize RIVA (downloads and sets up models)
sudo bash riva_init.sh
# This will take 10-30 minutes depending on your internet speed

# Start RIVA server
sudo bash riva_start.sh
```

#### On Windows with Docker:
1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Enable WSL 2 integration
3. Run in PowerShell as Administrator:
```powershell
# Initialize RIVA
.\riva_init.sh

# Start RIVA server  
.\riva_start.sh
```

### Step 2.6: Verify RIVA Installation

```bash
# Check if RIVA is running
docker ps

# You should see containers like:
# - riva-speech-api
# - riva-speech-client

# Test RIVA connection
python -c "import riva.client; auth = riva.client.Auth(uri='localhost:50051'); print('RIVA connection successful')"
```

---

## Phase 3: NVIDIA Omniverse and Audio2Face Setup

### Step 3.1: Install NVIDIA Omniverse Launcher

1. Go to [NVIDIA Omniverse](https://www.nvidia.com/omniverse/download/)
2. Click "Download Omniverse"
3. Fill out the form and download the installer
4. Run the installer and follow the setup wizard
5. Launch Omniverse Launcher after installation

### Step 3.2: Install Audio2Face

#### In Omniverse Launcher:
1. Click on the "EXCHANGE" tab
2. Search for "Audio2Face"
3. Click on "Audio2Face" application
4. Click "Install" (this will download ~2-3GB)
5. Wait for installation to complete
6. Click "Launch" to verify installation

### Step 3.3: Set Up Audio2Face Project

#### Launch Audio2Face:
1. Open Omniverse Launcher
2. Click "APPS" tab
3. Click "Launch" on Audio2Face
4. Wait for Audio2Face to start (first launch takes longer)

#### Load Sample Project:
1. In Audio2Face main menu, click "Open"
2. Navigate to sample scenes
3. Select `solved_arkit.usd` or similar sample
4. Click "Open"

#### Configure Streaming Audio Player:
1. In the Audio2Face interface, find the "Audio2Face" tool panel
2. Scroll down to find "Audio Player" section
3. Click the "+" button next to "Audio Player"
4. Select "Streaming Audio Player" from the dropdown
5. A new streaming player instance will be created

#### Connect Audio Player in Graph:
1. Click on the "Graph" tab at the bottom
2. You'll see the Audio2Face graph with nodes
3. Find the newly created "player_streaming_instance" node
4. Connect its "time" output to the "Audio2Face Core" node's "time" input
5. This enables the streaming audio to drive the facial animation

#### Note the Prim Path:
1. Click on the "player_streaming_instance" in the stage
2. In the Property panel, find the "Prim Path"
3. Copy this path (typically `/World/audio2face/PlayerStreaming`)
4. **Save this path** - you'll need it for configuration

### Step 3.4: Test Audio2Face Streaming

#### Find Test Script:
```bash
# The test script is located in your Audio2Face installation
# Typical path on Windows:
# C:\Users\[Username]\AppData\Local\ov\pkg\audio2face-[version]\exts\omni.audio2face.player\omni\audio2face\player\scripts\streaming_server\test_client.py

# On Linux:
# ~/.local/share/ov/pkg/audio2face-[version]/exts/omni.audio2face.player/omni/audio2face/player/scripts/streaming_server/test_client.py
```

#### Test the Connection:
1. Keep Audio2Face running with the streaming player configured
2. Navigate to the test script location
3. Run the test:
```bash
python test_client.py sample_audio.wav /World/audio2face/PlayerStreaming
```
4. You should see the avatar's face animate based on the audio

---

## Phase 4: Unreal Engine and MetaHuman Setup

### Step 4.1: Install Epic Games Launcher

1. Go to [Epic Games](https://www.epicgames.com/store/)
2. Download and install Epic Games Launcher
3. Sign in with your Epic Games account
4. Accept any required agreements

### Step 4.2: Install Unreal Engine 5.3

#### In Epic Games Launcher:
1. Click "Unreal Engine" tab
2. Click "Library" on the left
3. Click the "+" next to "Engine Versions"
4. **Important**: Select version "5.3.x" (NOT 5.4+)
5. Choose installation location (requires ~15GB)
6. Click "Install"
7. Wait for download and installation

### Step 4.3: Create MetaHuman Character (Optional)

#### Access MetaHuman Creator:
1. Go to [MetaHuman Creator](https://metahuman.unrealengine.com/)
2. Sign in with your Epic Games account
3. **Important**: Set Unreal Engine version to 5.3
4. Create your custom character:
   - Choose base template
   - Customize facial features
   - Adjust hair, clothing, etc.
   - Preview your character

#### Download Character:
1. Once satisfied with your character, click "Download"
2. Choose Unreal Engine 5.3 format
3. Download will include all necessary assets
4. Extract to a convenient location

### Step 4.4: Set Up Unreal Engine Project

#### Create New Project:
1. Launch Unreal Engine 5.3 from Epic Games Launcher
2. Click "Games" category
3. Select "Third Person" template
4. Choose "Blueprint" (not C++)
5. Set project location and name (e.g., "LLMAvatarTalk_UE")
6. Click "Create"

#### Import MetaHuman:
1. In Unreal Engine, open the Content Browser
2. Create a new folder called "MetaHumans"
3. Import your downloaded MetaHuman assets
4. Follow the import wizard prompts

### Step 4.5: Install Audio2Face Plugin

#### Locate Plugin Files:
The Audio2Face plugins are typically located at:
- Windows: `C:\Users\[Username]\AppData\Local\ov\pkg\audio2face-[version]\ue-plugins\audio2face-ue-plugins\ACEUnrealPlugin-5.3\ACE`
- Linux: `~/.local/share/ov/pkg/audio2face-[version]/ue-plugins/audio2face-ue-plugins/ACEUnrealPlugin-5.3/ACE`

#### Install Plugin:
1. Navigate to your Unreal Engine project directory
2. Create a "Plugins" folder if it doesn't exist
3. Copy the entire "ACE" folder to your project's "Plugins" directory
4. Restart Unreal Engine
5. Open your project
6. Go to Edit > Plugins
7. Search for "Audio2Face" or "ACE"
8. Enable the plugin
9. Restart Unreal Engine when prompted

### Step 4.6: Configure Live Link

#### Set Up Live Link:
1. In Unreal Engine, go to Window > Virtual Production > Live Link
2. Click the "Source" dropdown (+ icon)
3. Select "Audio2Face LiveLink"
4. Configure the connection:
   - **IP Address**: `localhost` (if Audio2Face is on same machine)
   - **Port**: `12030` (default)
   - **Audio Port**: `12031`
   - **Audio Sample Rate**: Match your Audio2Face settings (typically 22050 Hz)
5. Click "Ok"

#### Configure Live Link Subject:
1. In the Live Link window, you should see "Audio2Face" subject
2. Select it and check "Use ARKit Face"
3. Ensure the status shows "Connected" (green indicator)

#### Test the Connection:
1. Play some audio in Audio2Face
2. In Unreal Engine Live Link, you should see data streaming
3. The face blendshapes should update in real-time

---

## Phase 5: LLMAvatarTalk Project Configuration

### Step 5.1: Clone and Set Up Project

#### Clone Repository:
```bash
# Clone the project
git clone https://github.com/VisionaryArchitects/LLMAvatarTalk-An-Interactive-AI-Assistant.git
cd LLMAvatarTalk-An-Interactive-AI-Assistant
```

#### Create Virtual Environment:
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On Linux/Mac:
source venv/bin/activate
```

#### Install Dependencies:
```bash
# Upgrade pip
pip install --upgrade pip

# Install requirements
pip install -r requirements.txt
```

### Step 5.2: Configure Environment Variables

#### Create .env File:
```bash
# Copy the sample environment file
cp .env.sample .env
```

#### Edit .env File:
```bash
# Open .env in your preferred editor
# On Windows: notepad .env
# On Linux: nano .env
```

Add your NVIDIA API key:
```
NVIDIA_API_KEY=nvapi-your-actual-api-key-here
```

### Step 5.3: Configure Project Settings

#### Edit config.py:
```python
# Open config.py and modify settings

# RIVA server configuration
URI = 'localhost:50051'  # Adjust if RIVA is on different machine/port

# Voice configuration for different languages
voice_config = {
    'en-US': "English-US.Female-1",
    'zh-CN': "Mandarin-CN.Female-1"
}

# Set your preferred language
LANGUAGE = 'en-US'  # Change to 'zh-CN' for Chinese
```

#### Configure Audio2Face Settings:
Open `modules/audio2face.py` and verify settings:
```python
# In the Audio2FaceService __init__ method:
self.a2f_url = 'localhost:50051'  # Audio2Face gRPC port
self.avatar_instance = '/World/audio2face/PlayerStreaming'  # Your prim path from Step 3.3
```

### Step 5.4: Verify All Configurations

#### Check RIVA Connection:
```python
# Test RIVA connection
python -c "
import riva.client
from config import URI
auth = riva.client.Auth(uri=URI)
asr_service = riva.client.ASRService(auth)
print('RIVA ASR connection successful')
tts_service = riva.client.SpeechSynthesisService(auth)
print('RIVA TTS connection successful')
"
```

#### Check NVIDIA NIMs API:
```python
# Test NVIDIA NIMs connection
python -c "
from langchain_nvidia_ai_endpoints import ChatNVIDIA
from dotenv import load_dotenv
import os
load_dotenv()
llm = ChatNVIDIA(model='meta/llama3-70b-instruct')
response = llm.invoke('Hello')
print('NVIDIA NIMs connection successful')
print(f'Response: {response.content}')
"
```

---

## Phase 6: Integration and Testing

### Step 6.1: Start All Services

#### Start Services in Order:
1. **RIVA Server** (should already be running from Phase 2)
   ```bash
   # Verify RIVA is running
   docker ps | grep riva
   ```

2. **Audio2Face** (from Omniverse Launcher)
   - Launch Audio2Face
   - Open your configured project with streaming player
   - Ensure the streaming player is connected in the graph

3. **Unreal Engine** (with Live Link configured)
   - Open your UE project
   - Ensure Live Link shows Audio2Face connection
   - Load your MetaHuman character in the scene

### Step 6.2: Test Individual Components

#### Test ASR (Speech Recognition):
```python
# Create test file: test_asr.py
from modules.asr import ASRService
from config import URI

print(f"Testing ASR with RIVA server at: {URI}")
asr = ASRService()
print("Speak something (press Ctrl+C to stop)...")
try:
    asr.run()
    print(f"Transcript: {asr.transcript}")
except KeyboardInterrupt:
    print("ASR test completed")
```

#### Test LLM:
```python
# Create test file: test_llm.py
from modules.llm import LLMService

print("Testing LLM...")
llm = LLMService()
response = llm.invoke_conversation("Hello, how are you?")
print(f"LLM Response: {response}")
```

#### Test TTS:
```python
# Create test file: test_tts.py
from modules.tts import TTSService
import soundfile as sf

print("Testing TTS...")
tts = TTSService()
audio_bytes = tts.get_audio_bytes("Hello, this is a test of text to speech.")
print("TTS audio generated successfully")
# Optionally save to file for testing
# sf.write("test_output.wav", audio_bytes, 22050)
```

#### Test Audio2Face Integration:
```python
# Create test file: test_audio2face.py
from modules.audio2face import Audio2FaceService
from modules.tts import TTSService

print("Testing Audio2Face integration...")
tts = TTSService()
a2f = Audio2FaceService()

# Generate speech
audio_bytes = tts.get_audio_bytes("This is a test of audio to face animation.")

# Send to Audio2Face
a2f.make_avatar_speaks(audio_bytes)
print("Audio sent to Audio2Face - check your Unreal Engine for animation")
```

### Step 6.3: Full System Test

#### Run Complete Pipeline:
```bash
# Ensure all services are running, then:
python main.py
```

#### Testing Workflow:
1. The system will wait for speech input
2. Speak clearly into your microphone
3. The system will:
   - Convert speech to text (ASR)
   - Process with LLM for response
   - Convert response to speech (TTS)
   - Send audio to Audio2Face for animation
   - Display animation in Unreal Engine

#### Expected Behavior:
- Console shows recognized speech text
- Console shows LLM response chunks
- Audio2Face receives audio stream
- Unreal Engine shows MetaHuman facial animation
- Audio plays through speakers/headphones

### Step 6.4: Performance Optimization

#### Monitor System Resources:
```bash
# Monitor GPU usage
nvidia-smi -l 1

# Monitor CPU and RAM
htop  # Linux
# Task Manager on Windows
```

#### Optimize Settings:
1. **RIVA**: Adjust batch sizes in config.sh if needed
2. **Audio2Face**: Reduce quality settings if performance is poor
3. **Unreal Engine**: Adjust rendering quality based on hardware
4. **Python**: Consider using faster audio processing if latency is high

---

## Troubleshooting

### Common Issues and Solutions

#### 1. RIVA Connection Issues

**Problem**: `Connection refused` or `RIVA server not responding`
**Solutions**:
```bash
# Check if RIVA is running
docker ps | grep riva

# Restart RIVA if needed
cd ~/riva-setup/riva_quickstart_v2.15.1
sudo bash riva_stop.sh
sudo bash riva_start.sh

# Check port conflicts
netstat -tulpn | grep 50051
```

#### 2. NVIDIA NIMs API Issues

**Problem**: `Authentication failed` or `Rate limit exceeded`
**Solutions**:
- Verify API key in .env file
- Check credit balance at [NVIDIA NIMs](https://build.nvidia.com/)
- Ensure API key has proper permissions
- Try different model if current one is unavailable

#### 3. Audio2Face Connection Issues

**Problem**: No facial animation or connection timeout
**Solutions**:
```python
# Test gRPC connection
import grpc
channel = grpc.insecure_channel('localhost:50051')
# Should not raise exception
```

- Verify prim path in Audio2Face matches configuration
- Check if streaming player is properly connected in Audio2Face graph
- Restart Audio2Face and reconfigure streaming player

#### 4. Unreal Engine Live Link Issues

**Problem**: Live Link shows "Disconnected" status
**Solutions**:
- Verify Audio2Face plugin is installed and enabled
- Check IP address and port settings in Live Link
- Restart both Audio2Face and Unreal Engine
- Ensure firewall allows communication on Live Link ports

#### 5. Audio Issues

**Problem**: No audio output or poor quality
**Solutions**:
```python
# Test audio devices
import sounddevice as sd
print(sd.query_devices())  # List available audio devices
```

- Check microphone permissions
- Verify audio device settings in Windows/Linux
- Test with different audio devices
- Adjust sample rates to match across all components

#### 6. Performance Issues

**Problem**: System runs slowly or freezes
**Solutions**:
- Check GPU memory usage: `nvidia-smi`
- Reduce batch sizes in RIVA config
- Lower quality settings in Unreal Engine
- Close unnecessary applications
- Ensure adequate cooling for GPU

### Debug Mode Configuration

#### Enable Detailed Logging:
```python
# Add to main.py for debugging
import logging
logging.basicConfig(level=logging.DEBUG)

# Enable verbose mode in conversation
llm_service = LLMService()
llm_service.conversation.verbose = True
```

#### Test Each Service Independently:
```bash
# Test services one at a time
python test_asr.py
python test_llm.py  
python test_tts.py
python test_audio2face.py
```

---

## Advanced Configuration

### Multi-Language Support

#### Configure for Chinese:
```python
# In config.py
LANGUAGE = 'zh-CN'

# In RIVA config.sh
asr_language_code=zh-CN
tts_language_code=zh-CN

# Reinitialize RIVA after changes
sudo bash riva_init.sh
```

### Custom Voice Configuration

#### Add Custom Voices:
```python
# In config.py, extend voice_config
voice_config = {
    'en-US': "English-US.Female-1",
    'en-US-male': "English-US.Male-1", 
    'zh-CN': "Mandarin-CN.Female-1",
    'zh-CN-male': "Mandarin-CN.Male-1"
}
```

### Performance Tuning

#### GPU Memory Optimization:
```bash
# In RIVA config.sh
gpu_memory_pool_size=2048  # Adjust based on available GPU memory
```

#### Reduce Latency:
```python
# In modules/llm.py, adjust chunk size
def split_text_by_period(self, text, max_length=100):  # Reduced from 200
```

### Security Considerations

#### Network Security:
```bash
# Restrict RIVA access to localhost only
service_host=localhost  # In RIVA config.sh

# Use VPN if running services on different machines
# Configure firewall rules for required ports only
```

#### API Key Security:
```bash
# Ensure .env file is in .gitignore
echo ".env" >> .gitignore

# Use environment variables in production
export NVIDIA_API_KEY="your-key-here"
```

---

## Conclusion

This comprehensive guide covers the complete setup process for LLMAvatarTalk. The system integrates multiple complex technologies, so patience during setup is important. Each phase builds on the previous ones, so ensure each component is working before proceeding to the next.

For additional support:
- Check the individual tutorial files in the `docs/` directory
- Review the troubleshooting section for common issues
- Monitor system resources during operation
- Test components individually before running the full system

The final result is a sophisticated AI assistant that can engage in natural conversation while displaying realistic facial animations through a MetaHuman avatar.