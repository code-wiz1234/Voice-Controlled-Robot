# 🏥 Robot Nurse — Voice Clone Edition

A voice-controlled robot nurse simulation where the nurse speaks back **in your own cloned voice** using [Chatterbox TTS](https://github.com/resemble-ai/chatterbox) (free, zero-shot, no training required).

## Stack

| Layer | Tool |
|---|---|
| Speech-to-Text | Faster-Whisper |
| Intent parsing | Ollama (Llama 3.2) → keyword fallback |
| Text-to-Speech | Chatterbox TTS (your voice clone) |
| Map server | Flask |
| Control panel | Gradio |

## Setup

### 1. Prerequisites

```bash
# Python 3.10+
pip install -r requirements.txt

# ffmpeg (required for .m4a conversion)
brew install ffmpeg      # Mac
# sudo apt install ffmpeg  # Linux
```

### 2. Record your voice

Record yourself speaking naturally for **15–30 seconds**. Any content is fine — just talk clearly.

Save the file as `my_voice.m4a` (or `my_voice.wav`) **in the same folder as the notebook**.

```
nursebot/
├── nursebot.ipynb
├── my_voice.m4a       ← your recording goes here
├── requirements.txt
└── README.md
```

### 3. Run

Open `nursebot.ipynb` in VSCode and run all cells top to bottom.

Two browser tabs will open:
- `http://localhost:7860` — Gradio control panel
- `http://localhost:7861` — Animated hospital map

### 4. Optional: Smarter intent (Ollama)

```bash
brew install ollama
ollama serve          # keep running in a separate terminal
ollama pull llama3.2
```

If Ollama isn't running, the system falls back to a built-in keyword parser automatically.

## How to use

Select your role (Doctor / Patient) in the UI, then type or speak a command:

| Command | What happens |
|---|---|
| `Check blood pressure room 302` | Robot moves to Room 302 |
| `Administer medication room 305` | Robot goes to Medicine Cabinet |
| `Urgent — patient in room 301 has chest pains` | Emergency interrupt, robot rushes |
| `Cancel the blood pressure check` | Removes task from queue |
| `What tasks are active?` | Status report |

## Voice cloning notes

- **Chatterbox** is completely free and open-source (Apache 2.0)
- Zero-shot: no training, just a reference audio clip
- 15–30 sec reference gives best results; 6 sec minimum
- `.m4a` files are auto-converted to `.wav`
- Device auto-detected: MPS (Mac M4) → CUDA → CPU
