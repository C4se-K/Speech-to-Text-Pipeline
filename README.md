# Discord Real-Time Transcription Bot

A real-time audio capture and transcription pipeline for Discord voice channels, built with **Pycord**, **faster-whisper**, and a threaded message-passing architecture. The system intercepts raw Opus voice packets, decodes them into PCM frames, streams them into a transcription manager, and posts transcribed text back to Discord in real time.

---

## Overview

This project is composed of three main subsystems:

1. **Discord Interface Layer**  
   Custom extensions of Pycord’s `VoiceClient` and `DecodeManager` that intercept audio packets and push decoded PCM audio into a shared queue.

2. **Transcription Pipeline**  
   A `Transcription_Manager` built on faster-whisper that collects audio frames, uses VAD logic, performs transcription, and outputs recognized text.

3. **Coordinator (main.py)**  
   Launches the Discord bot on a separate thread, runs the transcription manager, and uses a messenger thread to move data between components.

Audio, commands, and outbound messages all move through producer/consumer queues for isolation and responsiveness.

---

## Features

- Real-time Discord voice capture
- Custom Opus → PCM decode path using overridden Pycord internals
- Streaming transcription powered by faster-whisper
- Multi-threaded pipeline built around queues
- Automatic posting of transcribed text into Discord
- Non-blocking architecture that keeps the bot responsive

---

## System Flow

### 1. Bot Startup
`main.py` loads environment variables, initializes queues, starts the Discord bot in a background thread, and waits for a ready signal.

### 2. Joining a Voice Channel
The `!join` command connects the bot to the caller’s voice channel using a custom `ExtendVoiceClient`, which:

- Starts recording
- Decodes Opus packets
- Pushes PCM frames into `raw_audio_queue`

### 3. Transcription Pipeline
A messenger thread continuously:

- Feeds raw audio into the transcription manager
- Retrieves transcription output
- Sends messages back to Discord

A dedicated processing loop in `main.py` calls:

```python
transcription.process_buffer()
```
to drive model inference.

4. Leaving a Voice Channel

`!leave `stops audio capture, disconnects the bot, and removes the voice connection.

# Installation
## Requirements

- Python 3.10+
- Opus installed on the system
- FFmpeg
- GPU support recommended for faster-whisper

## Install Dependencies
Run the command:
```bash
pip install -r requirements.txt
```

### dependencies:
- py-cord
- faster-whisper
- numpy
- scipy
- webrtcvad
- python-dotenv

## Environment Setup
1. Create a .env file:
```bash
ACCESS_TOKEN=your_discord_bot_token_here
```
2. Running the Bot
```bash
python main.py
```

3. In Discord:

Join any voice channel.
In a channel the discord bot is participating in, run the command:
```bash
!join
```
The bot will start capturing audio and transcribing into the text channel where the command was issued.

To stop:
```bash
!leave
```
## Concurrency Model

The system uses multiple threads:
- Discord event loop thread — Runs Pycord (asyncio).
- Decode thread — Converts Opus packets to PCM frames.
- Messenger thread — Moves audio and messages between queues.
- Main loop — Continuously drives the transcription engine.
  
This separation ensures that Whisper inference never blocks Discord’s event loop.
