# Architecture

## Overview

NurseBot is split across two servers that share Python state:

- **Flask (port 7861)** serves a standalone HTML page with the hospital map. The map's JavaScript calls `/state` every 300ms and moves the robot based on the response. No Gradio is involved in the animation loop — this is intentional. Gradio's Svelte framework doesn't reliably expose component values to vanilla JS, so the map runs independently.

- **Gradio (port 7860)** handles user input (text and voice) and shows the task queue. It has no role in moving the robot; it only updates the Python `RobotController` object which Flask reads.

## Data flow

```
User types / speaks
    → STTEngine (if voice)         faster-whisper, CPU, int8
    → IntentParser                 Ollama → rule fallback
    → NurseSystem.process()
    → TaskManager (background thread)
        → RobotController.assign_task()
        → worker executes steps
        → RobotController.arrive() / update_progress() / complete() / idle()

Flask /state endpoint
    ← RobotController.get_state()  called every 300ms by map JS
    → JSON { target, status, action, carrying, progress }

Map JS
    → lerp(current_pos, target_pos, 0.07) each animation frame
    → robot glides smoothly between rooms
```

## Task queue mechanics

The `TaskManager` runs a single background daemon thread (`NurseWorker`). It pops the highest-priority task from the queue and runs it step by step. Each step sleeps for a fraction of the estimated duration, so progress advances at a realistic pace.

**Interrupt:**
When an urgent task arrives mid-execution, `_interrupt_evt` is set. The worker detects this between sub-steps, marks the current task as `INTERRUPTED`, pushes it onto `interrupted_stack`, and starts the urgent task instead. When the urgent task completes, the last interrupted task is popped and re-queued at the front.

**Priority ordering:**
```
URGENT (3) > NORMAL (2) > LOW (1)
```
Within the same priority level, FIFO by creation time.

## Intent parsing

Two-tier system:

1. **Ollama / llama3.2** — if the server is running and the model is pulled, a structured JSON prompt is sent and the response parsed. More accurate on complex free-form speech.

2. **RuleParser** — keyword matching + regex room extraction. Handles all standard nursing commands reliably without any model. Used when Ollama is unavailable.

The parser never crashes the system — any exception falls back to the rule parser.

## TTS

Kokoro `af_heart` voice. Synthesis happens on a background thread (`speak_async`) so it never blocks the task worker. The audio is both played via `sounddevice` and displayed as an `IPython.display.Audio` widget in the notebook cell for confirmation.

## Why Flask + iframe instead of pure Gradio

Gradio's component updates go through Svelte's reactive system. When Python updates a `gr.Textbox`, Svelte may batch or defer writing the new value to the actual DOM `textarea.value`. Vanilla JS reading `element.value` therefore gets stale data.

The iframe approach bypasses this entirely: the Flask page is its own document with its own JS runtime. `fetch("/state")` goes directly to Flask over HTTP — no Svelte, no component lifecycle, always fresh data.
