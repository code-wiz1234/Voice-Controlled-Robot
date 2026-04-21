# Command Reference

## Doctor commands

Doctors have full control: add, interrupt, delete any task, set urgent priority.

| Intent | Example phrases |
|--------|----------------|
| Add normal task | `check blood pressure room 302` |
| Add low priority | `take temperature whenever you can` |
| Add urgent (interrupts) | `urgent — patient in room 301 has chest pains` |
| Delete task | `cancel the blood pressure check` |
| Status | `what tasks are currently active` |

## Patient commands

Patients can request tasks and trigger emergencies, but cannot delete doctor-assigned tasks or set urgent priority directly.

| Intent | Example phrases |
|--------|----------------|
| Request task | `can you bring my medication` |
| Emergency | `I'm in a lot of pain, I need help` |
| Emergency | `help me, chest hurts` |

Patient emergency phrases auto-trigger an interrupt regardless of wording — the parser watches for: `pain`, `help`, `emergency`, `chest`, `bleeding`, `severe`.

## Room routing

The robot resolves a destination from the task name:

| Task contains | Destination |
|---------------|-------------|
| `room 301–306` | That patient room |
| `medication`, `medicine`, `administer` | Medicine Cabinet |
| `draw blood`, `lab`, `specimen` | Laboratory |
| `icu`, `critical`, `intensive` | ICU |
| Anything else | Nurse Station |

## Task names recognised by the rule parser

The rule parser maps these keywords to standardised task names:

| Keyword | Task name |
|---------|-----------|
| `blood pressure`, `bp` | Check Blood Pressure |
| `medication`, `medicine` | Administer Medication |
| `temperature` | Take Temperature |
| `vital`, `vitals` | Vital Signs Check |
| `draw blood` | Draw Blood Sample |
| `iv`, `drip` | Change IV |
| `dressing`, `wound` | Wound Dressing |
| `assessment` | Patient Assessment |
| `oxygen`, `o2` | Check Oxygen Level |
| `ecg` | ECG Recording |
| `emergency` | Emergency Response |

If none match, the parser strips filler words and title-cases whatever remains.
