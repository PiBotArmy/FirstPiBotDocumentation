# Raspberry Pi Talking Robot – Setup Summary

This document summarizes the successful steps used to build and run the Raspberry Pi talking robot using Bluetooth audio and OpenAI APIs.

---

## Overview

The robot performs:

1. ENTER to start recording
2. ENTER to stop recording
3. Speech-to-text (gpt-4o-mini-transcribe)
4. Chat response (gpt-4o-mini)
5. Text-to-speech (gpt-4o-mini-tts, voice = sage)
6. Playback via ffplay (with padding to prevent clipped speech)
7. Session memory with balanced summarization

---

## Hardware Used

- Raspberry Pi 5
- USB microphone (USB PnP Audio Device)
- Bluetooth speaker (JBL Charge 4)
- Raspberry Pi OS (Bookworm)

---

## Audio Configuration (Bluetooth + PipeWire)

### Bluetooth Speaker

Confirm connection:

    wpctl status

Ensure Bluetooth sink is default (should have `*`):

    wpctl set-default <SINK_ID>

Test playback using PipeWire (NOT aplay):

    pw-play /usr/share/sounds/alsa/Front_Center.wav

Important: `aplay` was not used because it does not reliably route through PipeWire on this setup.

---

### Microphone (USB)

Confirm detection:

    arecord -l

Preferred test using PipeWire:

    pw-record --channels 1 --rate 16000 test.wav
    pw-play test.wav

Alternative ALSA test (explicit device):

    arecord -D hw:2,0 -f S16_LE -r 16000 -c 1 -d 5 test.wav
    pw-play test.wav

---

## System Dependencies

    sudo apt update
    sudo apt install python3-venv python3-full ffmpeg portaudio19-dev -y

---

## Project Setup

Create project folder:

    mkdir -p /home/kyle/pi_talking_robot
    cd /home/kyle/pi_talking_robot

Create virtual environment:

    python3 -m venv venv
    source venv/bin/activate

Install Python packages:

    pip install --upgrade pip
    pip install openai sounddevice soundfile numpy

---

## API Key Setup

Temporary:

    export OPENAI_API_KEY="sk-NEW-KEY"

Permanent (add to ~/.bashrc):

    export OPENAI_API_KEY="sk-NEW-KEY"

Reload:

    source ~/.bashrc

For desktop launcher reliability, export the key inside run_robot.sh.

---

## Run Script (Desktop Safe)

Create:

    nano /home/kyle/pi_talking_robot/run_robot.sh

Contents:

    #!/bin/bash
    export OPENAI_API_KEY="sk-NEW-KEY"
    cd /home/kyle/pi_talking_robot || exit 1
    source venv/bin/activate
    python robot_push_to_talk.py
    read -p "Press ENTER to close"

Make executable:

    chmod +x run_robot.sh

---

## Core Robot Script Features

- ENTER-based push-to-talk
- Silent recording guard
- Voice = sage
- Playback via:

    ffplay -nodisp -autoexit -loglevel quiet -af apad=pad_dur=0.3

The apad filter prevents the first word from being clipped when using Bluetooth audio.

---

## Memory Optimization (Balanced Mode)

    MAX_MESSAGES = 12
    KEEP_LAST = 5

When conversation exceeds 12 messages:
- Older dialogue is summarized
- Last 5 exchanges remain verbatim
- Summary stored as system message

---

## Verified Working State

- Bluetooth audio playback via PipeWire
- USB microphone capture
- OpenAI STT + Chat + TTS pipeline
- No clipped first word
- Stable ENTER-based push-to-talk
- Session memory with summarization
- Desktop-launchable execution

---

End of document.
