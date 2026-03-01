# Raspberry Pi Talking Robot – Software Architecture

This document describes the final working software architecture of the Raspberry Pi Talking Robot.

---

# High-Level Architecture

User Speech
   ↓
Audio Capture (sounddevice)
   ↓
Speech-to-Text (OpenAI gpt-4o-mini-transcribe)
   ↓
Conversation Memory (Session + Summarization)
   ↓
Chat Model (OpenAI gpt-4o-mini)
   ↓
Text-to-Speech (OpenAI gpt-4o-mini-tts, voice = sage)
   ↓
Audio Playback (ffplay via PipeWire → Bluetooth Speaker)

---

# Core Components

## 1. Audio Capture Layer

Library:
- sounddevice

Configuration:
- SAMPLE_RATE = 16000
- Mono channel
- int16 audio format

Behavior:
- ENTER starts recording
- ENTER stops recording
- Audio buffered in memory
- Guard prevents empty / too-short recordings (<200ms)

Purpose:
Provides stable push-to-talk without GPIO hardware.

---

## 2. Speech-to-Text (STT)

Model:
- gpt-4o-mini-transcribe

Input:
- Temporary WAV file (16 kHz mono)

Output:
- Plain text transcription

Design Notes:
- Empty transcription is ignored
- Prevents "Audio file corrupted" errors by skipping silent clips

---

## 3. Conversation Memory System

Structure:
conversation = list of messages

Initial state:
- System prompt defines robot personality

Memory Optimization (Balanced Mode):

MAX_MESSAGES = 12
KEEP_LAST = 5

When conversation length exceeds 12:
1. Older messages are summarized
2. Summary stored as a system message
3. Most recent 5 exchanges remain intact

Purpose:
- Prevents token explosion
- Reduces latency
- Controls API cost
- Maintains conversational continuity

---

## 4. Language Model (LLM)

Model:
- gpt-4o-mini

Input:
- Entire conversation array (system + summary + recent dialogue)

Output:
- Assistant reply text

Design:
- Stateless per API call
- Stateful via conversation array

---

## 5. Text-to-Speech (TTS)

Model:
- gpt-4o-mini-tts

Voice:
- sage

Output:
- MP3 binary stream

Playback:
- ffplay
- -nodisp
- -autoexit
- -loglevel quiet
- -af apad=pad_dur=0.3

Important:
The apad filter prevents Bluetooth startup latency from clipping the first word.

---

## 6. Audio Playback Layer

Engine:
- ffplay (from ffmpeg)

Reason:
- Works reliably with PipeWire
- Avoids ALSA routing issues
- Stable with Bluetooth speakers

Audio Routing:
ffplay → PipeWire → BlueZ → Bluetooth Speaker

---

# Environment Configuration

Virtual Environment:
- Python venv used
- Isolates dependencies

Required Python Packages:
- openai
- sounddevice
- soundfile
- numpy

System Packages:
- ffmpeg
- portaudio19-dev
- python3-venv

---

# Execution Flow

Main loop:

1. Wait for ENTER
2. Record audio
3. Transcribe audio
4. Append user message to conversation
5. Generate assistant reply
6. Append reply to conversation
7. Summarize if needed
8. Speak reply
9. Repeat

Exit:
- Ctrl + C

---

# Stability Enhancements Implemented

- Silent recording guard
- Balanced memory summarization
- Bluetooth audio padding
- Environment variable based API key
- Desktop-launch-safe run script

---

# Current Verified State

✔ USB microphone capture  
✔ Bluetooth playback via PipeWire  
✔ OpenAI STT → LLM → TTS pipeline  
✔ Session memory with summarization  
✔ No clipped first word  
✔ Desktop-launchable execution  

---

End of document.
