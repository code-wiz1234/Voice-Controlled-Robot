# NurseBot

Voice-controlled robot nurse simulation for medical environments. Doctors and patients issue commands by voice or text — the robot navigates a hospital floor plan in real time, runs nursing tasks in priority order, and handles emergency interrupts mid-task.

---

## Stack

| Layer | Library | Notes |
|-------|---------|-------|
| STT | `faster-whisper` | Runs fully offline, CPU |
| TTS | `kokoro` | `af_heart` voice, local neural |
| Intent | `llama3.2` via Ollama | Falls back to rule parser if unavailable |
| Map server | Flask | Serves hospital map + `/state` JSON |
| Control panel | Gradio | Text, voice, quick-action buttons |

---

## Quickstart

```bash
git clone https://github.com/yourname/nursebot.git
cd nursebot
pip install -r requirements.txt
jupyter notebook nursebot.ipynb
```

Run all cells top to bottom. Two browser tabs open automatically:
- `http://localhost:7861` — animated hospital map
- `http://localhost:7860` — control panel

### Optional: Ollama (smarter intent parsing)

```bash
brew install ollama
ollama serve          # in a terminal, keep running
ollama pull llama3.2  # ~2 GB, one time
```

If Ollama isn't running, NurseBot falls back to the built-in keyword parser.

---

## How it works

```
Voice / Text input
      ↓
Faster-Whisper (speech → text)
      ↓
IntentParser (text → action JSON)
      ↓
NurseSystem → TaskManager
      ↓                    ↓
RobotController     Kokoro TTS speaks response
      ↓
Flask /state endpoint
      ↓
Map JS polls every 300ms → robot moves
```

**Task lifecycle:**
```
QUEUED → RUNNING → COMPLETED
             ↓
         INTERRUPTED (emergency came in)
             ↓
         QUEUED again (auto-resumed after urgent done)
```

---

## Commands

| Say | What happens |
|-----|-------------|
| `check blood pressure room 302` | Robot goes to Room 302 |
| `administer medication` | Robot goes to Medicine Cabinet |
| `draw blood` | Robot goes to Laboratory |
| `urgent — patient in room 301 has chest pains` | Pauses current task, rushes to Room 301 |
| `cancel the blood pressure check` | Removes task from queue |
| `what tasks are active` | Status report |
| `I'm in pain, help me` _(patient)_ | Emergency interrupt |

---

## Room layout

```
┌─────────────────────────────────────────────┐
│  Medicine Cabinet │   Laboratory  │    ICU   │
├───────────────────┼───────────────┼──────────┤
│     Room 301      │    Room 302   │  Room 303│
├───────────────────┼───────────────┼──────────┤
│     Room 304      │    Room 305   │  Room 306│
├───────────────────┴───────────────┴──────────┤
│               Nurse Station                  │
└──────────────────────────────────────────────┘
```

---

## Accessibility

- **Blind users** — voice input only; nurse speaks every response via Kokoro TTS
- **Deaf users** — text input; visual task board and animated map show everything
- **Emergency button** — large one-tap shortcut always visible for patients

---

## Files

```
nursebot/
├── nursebot.ipynb          main notebook — run this
├── demo_scenarios.ipynb    pre-built demo without microphone
├── requirements.txt
├── .gitignore
├── LICENSE
└── docs/
    ├── architecture.md
    └── commands.md
```

---

## License

MIT
