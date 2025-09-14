# LLMAvatarTalk Troubleshooting Guide

This comprehensive troubleshooting guide covers common issues encountered during setup and operation of the LLMAvatarTalk system.

## Table of Contents
1. [General System Issues](#general-system-issues)
2. [RIVA Server Issues](#riva-server-issues)
3. [Audio2Face Issues](#audio2face-issues)
4. [Unreal Engine Issues](#unreal-engine-issues)
5. [Python/Environment Issues](#python-environment-issues)
6. [Network and Connection Issues](#network-and-connection-issues)
7. [Performance Issues](#performance-issues)
8. [Audio Issues](#audio-issues)
9. [Integration Issues](#integration-issues)
10. [Debugging Tools and Commands](#debugging-tools-and-commands)

---

## General System Issues

### GPU Not Detected or Insufficient VRAM

**Symptoms:**
- "CUDA out of memory" errors
- Applications crash on startup
- Poor performance or freezing

**Solutions:**
```bash
# Check GPU status
nvidia-smi

# Check CUDA installation
nvcc --version

# Monitor GPU memory usage
nvidia-smi -l 1
```

**Actions:**
1. **Update NVIDIA drivers** to latest stable version
2. **Close unnecessary applications** consuming GPU memory
3. **Reduce quality settings** in Unreal Engine and Audio2Face
4. **Check GPU temperature** - ensure adequate cooling

### Insufficient System RAM

**Symptoms:**
- System becomes unresponsive
- Applications crash randomly
- Slow performance

**Solutions:**
1. **Close unnecessary applications**
2. **Increase virtual memory/swap space**
3. **Consider upgrading to 32GB+ RAM**
4. **Monitor RAM usage**: Task Manager (Windows) or `htop` (Linux)

### Port Conflicts

**Symptoms:**
- "Address already in use" errors
- Services fail to start
- Connection refused errors

**Diagnosis:**
```bash
# Check what's using specific ports
netstat -tulpn | grep 50051  # RIVA
netstat -tulpn | grep 12030  # Live Link
netstat -tulpn | grep 50052  # Audio2Face gRPC

# Windows equivalent
netstat -ano | findstr 50051
```

**Solutions:**
1. **Kill conflicting processes**:
   ```bash
   sudo kill -9 <PID>
   ```
2. **Change port numbers** in configuration files
3. **Restart services** in correct order

---

## RIVA Server Issues

### Docker Container Won't Start

**Symptoms:**
- "docker: permission denied" errors
- Containers exit immediately
- RIVA initialization fails

**Solutions:**
```bash
# Fix Docker permissions
sudo usermod -aG docker $USER
newgrp docker

# Check Docker service
sudo systemctl status docker
sudo systemctl start docker

# Verify NVIDIA Container Toolkit
docker run --rm --gpus all nvidia/cuda:11.0.3-base-ubuntu20.04 nvidia-smi
```

### RIVA Models Fail to Download

**Symptoms:**
- "Model download failed" during `riva_init.sh`
- Network timeout errors
- Incomplete model files

**Solutions:**
```bash
# Check NGC authentication
ngc config current

# Re-authenticate if needed
ngc config set

# Check internet connectivity
ping catalog.ngc.nvidia.com

# Restart download
sudo bash riva_init.sh
```

### RIVA Service Not Responding

**Symptoms:**
- Connection timeout to localhost:50051
- gRPC errors in Python client
- Empty response from RIVA

**Diagnosis:**
```bash
# Check container status
docker ps | grep riva

# Check service logs
docker logs riva-speech-api

# Test basic connectivity
curl -X POST http://localhost:50051/v1/health
```

**Solutions:**
1. **Restart RIVA services**:
   ```bash
   sudo bash riva_stop.sh
   sudo bash riva_start.sh
   ```
2. **Check configuration** in `config.sh`
3. **Verify port settings** (should be 50051)
4. **Monitor resource usage** during startup

### Language Model Issues

**Symptoms:**
- Wrong language responses
- Poor recognition accuracy
- Missing voice options

**Solutions:**
```bash
# Check language configuration in config.sh
grep -E "(language_code|voice)" config.sh

# For English:
asr_language_code=en-US
tts_language_code=en-US

# For Chinese:
asr_language_code=zh-CN
tts_language_code=zh-CN

# Reinitialize after changes
sudo bash riva_init.sh
```

---

## Audio2Face Issues

### Audio2Face Won't Launch

**Symptoms:**
- Omniverse Launcher shows error
- Audio2Face crashes on startup
- Black screen or frozen interface

**Solutions:**
1. **Update GPU drivers** to latest version
2. **Check GPU memory availability**:
   ```bash
   nvidia-smi
   ```
3. **Run as administrator** (Windows)
4. **Clear Omniverse cache**:
   - Windows: `%LOCALAPPDATA%\ov\`
   - Linux: `~/.local/share/ov/`
5. **Reinstall Audio2Face** through Omniverse Launcher

### Sample Projects Won't Load

**Symptoms:**
- "Failed to open USD file" errors
- Empty viewport
- Missing character models

**Solutions:**
1. **Check file paths** for special characters
2. **Try different sample projects**
3. **Verify Audio2Face installation** is complete
4. **Refresh the project browser** (F5)
5. **Check disk space** and permissions

### Streaming Player Not Working

**Symptoms:**
- No streaming player option
- Can't connect streaming player in graph
- Audio data not received

**Diagnosis:**
```python
# Test gRPC connection
import grpc
try:
    channel = grpc.insecure_channel('localhost:50051')
    # If no exception, connection is possible
    print("gRPC connection available")
except Exception as e:
    print(f"gRPC error: {e}")
```

**Solutions:**
1. **Recreate streaming player**:
   - Delete existing player
   - Add new Streaming Audio Player
   - Reconnect in graph view
2. **Check graph connections**:
   - Ensure time outputs are connected
   - Verify node compatibility
3. **Restart Audio2Face** and reload project

### Face Animation Not Working

**Symptoms:**
- Character face doesn't move
- No blendshape data
- Frozen expressions

**Solutions:**
1. **Verify audio input** is being received
2. **Check character compatibility** (ARKit face support)
3. **Test with sample audio**:
   ```bash
   python test_client.py sample_audio.wav /World/audio2face/PlayerStreaming
   ```
4. **Reset character pose** to default
5. **Check animation quality settings**

---

## Unreal Engine Issues

### Plugin Installation Problems

**Symptoms:**
- Audio2Face plugin not appearing
- Plugin compilation errors
- UE crashes when enabling plugin

**Solutions:**
1. **Verify UE version** (must be 5.3.x, not 5.4+)
2. **Check plugin location**:
   ```
   [ProjectPath]/Plugins/ACE/ACE.uplugin
   ```
3. **Verify plugin file integrity**
4. **Delete and reinstall plugin**
5. **Check UE logs** for detailed error messages

### Live Link Connection Issues

**Symptoms:**
- Live Link shows "Disconnected"
- No Audio2Face subject appears
- Data not streaming

**Diagnosis:**
1. **Check Live Link window** status indicators
2. **Verify Audio2Face Live Link plugin** is enabled
3. **Test network connectivity** between applications

**Solutions:**
1. **Restart both applications** in correct order:
   - Start Audio2Face first
   - Configure streaming player
   - Start Unreal Engine
   - Configure Live Link
2. **Check firewall settings**:
   - Allow UE and Audio2Face through firewall
   - Verify ports 12030/12031 are open
3. **Verify IP and port settings**:
   - IP: `localhost` or actual IP address
   - Port: `12030` (Live Link), `12031` (Audio)

### MetaHuman Import Issues

**Symptoms:**
- Character doesn't appear in Content Browser
- Missing textures or materials
- Animation blueprint errors

**Solutions:**
1. **Check MetaHuman version compatibility** (UE 5.3)
2. **Verify all files imported** correctly
3. **Reimport missing assets**
4. **Check file paths** and permissions
5. **Update project to latest MetaHuman plugin**

### Performance Issues in UE

**Symptoms:**
- Low frame rate
- Stuttering animation
- System freezes

**Solutions:**
1. **Reduce rendering quality**:
   - Edit → Project Settings → Rendering
   - Lower anti-aliasing, shadows, post-processing
2. **Adjust MetaHuman LOD settings**
3. **Monitor GPU usage** with `nvidia-smi`
4. **Close unnecessary background applications**

---

## Python Environment Issues

### Package Installation Failures

**Symptoms:**
- pip install errors
- Missing dependencies
- Import errors in Python

**Solutions:**
```bash
# Update pip
pip install --upgrade pip

# Install build tools (Windows)
# Download Visual Studio Build Tools

# Install system dependencies (Ubuntu)
sudo apt update
sudo apt install python3-dev build-essential

# Create fresh virtual environment
python -m venv venv_new
source venv_new/bin/activate
pip install -r requirements.txt
```

### NVIDIA RIVA Client Issues

**Symptoms:**
- "No module named 'riva'" errors
- gRPC connection failures
- Audio device errors

**Solutions:**
```bash
# Install specific versions
pip install nvidia-riva-client==2.14.0
pip install grpcio==1.51.3

# Check audio device permissions (Linux)
sudo usermod -a -G audio $USER

# Test basic RIVA connection
python -c "import riva.client; print('RIVA client imported successfully')"
```

### LangChain and NVIDIA NIMs Issues

**Symptoms:**
- Authentication errors
- API rate limiting
- Model not available errors

**Solutions:**
```bash
# Verify API key in .env file
cat .env | grep NVIDIA_API_KEY

# Test API connection
python -c "
from langchain_nvidia_ai_endpoints import ChatNVIDIA
from dotenv import load_dotenv
load_dotenv()
llm = ChatNVIDIA(model='meta/llama3-70b-instruct')
print('API connection successful')
"

# Check credit balance at build.nvidia.com
```

---

## Network and Connection Issues

### Firewall Blocking Connections

**Symptoms:**
- Connection timeout errors
- Services can't communicate
- Random disconnections

**Windows Solutions:**
1. **Windows Defender Firewall**:
   - Control Panel → System and Security → Windows Defender Firewall
   - Allow apps through firewall
   - Add Unreal Engine, Audio2Face, Python

2. **Advanced firewall rules**:
   ```cmd
   # Allow specific ports
   netsh advfirewall firewall add rule name="RIVA" dir=in action=allow protocol=TCP localport=50051
   netsh advfirewall firewall add rule name="LiveLink" dir=in action=allow protocol=TCP localport=12030
   ```

**Linux Solutions:**
```bash
# Check firewall status
sudo ufw status

# Allow specific ports
sudo ufw allow 50051
sudo ufw allow 12030
sudo ufw allow 12031

# Or disable firewall temporarily for testing
sudo ufw disable
```

### Network Interface Issues

**Symptoms:**
- Services can't find each other
- Localhost connections fail
- IP address conflicts

**Solutions:**
```bash
# Check network interfaces
ip addr show  # Linux
ipconfig /all  # Windows

# Test localhost connectivity
ping localhost
telnet localhost 50051

# Check if services are listening
netstat -tlnp | grep 50051
```

---

## Performance Issues

### High GPU Memory Usage

**Symptoms:**
- CUDA out of memory errors
- System becomes unresponsive
- Applications crash randomly

**Monitoring:**
```bash
# Continuous GPU monitoring
nvidia-smi -l 1

# Detailed GPU memory info
nvidia-smi --query-gpu=memory.total,memory.used,memory.free --format=csv
```

**Solutions:**
1. **Reduce model batch sizes** in RIVA config
2. **Lower quality settings** in UE and Audio2Face
3. **Use GPU memory management**:
   ```python
   # In Python scripts, add:
   import torch
   torch.cuda.empty_cache()
   ```
4. **Close other GPU applications**

### High CPU Usage

**Symptoms:**
- System becomes slow
- Audio stuttering
- High fan noise

**Solutions:**
1. **Monitor CPU usage**:
   ```bash
   htop  # Linux
   # Task Manager on Windows
   ```
2. **Optimize process priorities**
3. **Reduce concurrent operations**
4. **Check for background processes**

### Network Latency

**Symptoms:**
- Delayed responses
- Audio-visual sync issues
- Stuttering animation

**Solutions:**
1. **Use localhost** for all services when possible
2. **Check network bandwidth** and stability
3. **Optimize buffer sizes** in streaming components
4. **Use wired network** instead of WiFi

---

## Audio Issues

### Microphone Not Working

**Symptoms:**
- No speech recognition
- Silent audio input
- Permission denied errors

**Solutions:**
```python
# Test audio devices
import sounddevice as sd
print(sd.query_devices())

# Check default input device
print(sd.default.device)
```

**Windows:**
1. **Check microphone permissions** in Windows Settings
2. **Set correct default device** in Sound Control Panel
3. **Update audio drivers**

**Linux:**
```bash
# Check audio system
pulseaudio --check -v

# List audio devices
arecord -l
pacmd list-sources

# Test microphone
arecord -d 5 test.wav
aplay test.wav
```

### Audio Output Issues

**Symptoms:**
- No sound from TTS
- Distorted audio
- Wrong audio device

**Solutions:**
1. **Check audio device settings** in system
2. **Test with simple audio playback**:
   ```python
   import sounddevice as sd
   import numpy as np
   
   # Generate test tone
   fs = 22050
   duration = 2
   frequency = 440
   t = np.linspace(0, duration, int(fs * duration))
   audio = 0.5 * np.sin(2 * np.pi * frequency * t)
   sd.play(audio, fs)
   ```
3. **Update audio drivers**
4. **Check volume levels** and mute settings

### Sample Rate Mismatches

**Symptoms:**
- Audio plays too fast/slow
- Poor quality synthesis
- Synchronization issues

**Solutions:**
1. **Standardize sample rates** across all components:
   - RIVA: 22050 Hz
   - Audio2Face: 22050 Hz
   - Python audio: 22050 Hz
2. **Check configuration files**:
   ```bash
   # In RIVA config.sh
   audio_sample_rate=22050
   
   # In Audio2Face player settings
   # Set sample rate to 22050 Hz
   ```

---

## Integration Issues

### End-to-End Pipeline Failures

**Symptoms:**
- Speech recognition works but no animation
- TTS generates audio but face doesn't move
- Partial system functionality

**Debugging Steps:**
1. **Test each component individually**:
   ```bash
   python test_asr.py
   python test_llm.py
   python test_tts.py
   python test_audio2face.py
   ```

2. **Check data flow**:
   - ASR transcript appears in console
   - LLM generates response chunks
   - TTS creates audio bytes
   - Audio2Face receives audio stream

3. **Verify configurations match**:
   - Server URLs and ports
   - API keys and authentication
   - File paths and prim paths

### Configuration Mismatches

**Symptoms:**
- Services can't connect
- Wrong language responses
- Animation data not received

**Common Mismatches:**
```python
# Check these common configuration issues:

# config.py
URI = 'localhost:50051'  # Should match RIVA server
LANGUAGE = 'en-US'       # Should match RIVA and voices

# modules/audio2face.py
self.a2f_url = 'localhost:50051'  # Should match Audio2Face gRPC
self.avatar_instance = '/World/audio2face/PlayerStreaming'  # Should match exact prim path

# Live Link settings in UE
# IP: localhost, Port: 12030, Audio Port: 12031
```

### Timing and Synchronization Issues

**Symptoms:**
- Audio and animation out of sync
- Delayed responses
- Choppy animation

**Solutions:**
1. **Optimize chunk sizes** in LLM text splitting
2. **Adjust audio buffer sizes**
3. **Use faster hardware** or reduce quality settings
4. **Check system clock synchronization**

---

## Debugging Tools and Commands

### System Information
```bash
# GPU information
nvidia-smi
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv

# System information
uname -a  # Linux
systeminfo  # Windows

# Python environment
pip list
python --version
```

### Network Debugging
```bash
# Check listening ports
netstat -tlnp | grep :50051
netstat -tlnp | grep :12030

# Test connectivity
telnet localhost 50051
curl -X POST http://localhost:50051/v1/health

# DNS resolution
nslookup localhost
```

### Application Logs
```bash
# Docker logs
docker logs riva-speech-api
docker logs --follow riva-speech-api

# Unreal Engine logs
# Location: [ProjectFolder]/Saved/Logs/

# Python debugging
python -u main.py > debug.log 2>&1
```

### Performance Monitoring
```bash
# GPU monitoring
nvidia-smi -l 1
nvidia-smi --query-gpu=utilization.gpu,memory.used --format=csv -l 1

# System resources
htop  # Linux
# Task Manager or Resource Monitor on Windows

# Network monitoring
netstat -i  # Network interface stats
iftop       # Real-time network usage
```

### Audio Debugging
```python
# Test audio system
import sounddevice as sd
import numpy as np

# List devices
print(sd.query_devices())

# Test recording
recording = sd.rec(int(5 * 22050), samplerate=22050, channels=1)
sd.wait()
print("Recording complete")

# Test playback
sd.play(recording, samplerate=22050)
sd.wait()
```

---

## Getting Help

### Before Asking for Help
1. **Check all configuration files** match the documentation
2. **Test individual components** separately
3. **Review log files** for specific error messages
4. **Try the suggested solutions** in this guide
5. **Document your exact setup** (OS, hardware, versions)

### Information to Include
- **Operating system** and version
- **Hardware specifications** (GPU, RAM, CPU)
- **Software versions** (UE, Audio2Face, RIVA, Python)
- **Exact error messages** or symptoms
- **Configuration files** content (remove API keys)
- **Steps to reproduce** the issue

### Community Resources
- **GitHub Issues**: Report bugs and get help from the community
- **NVIDIA Developer Forums**: For RIVA, Audio2Face, and Omniverse issues
- **Epic Games Forums**: For Unreal Engine and MetaHuman issues
- **Discord/Slack**: Real-time help from community members

This troubleshooting guide should help resolve most common issues. If you encounter a problem not covered here, please document it and consider contributing to the guide to help other users.