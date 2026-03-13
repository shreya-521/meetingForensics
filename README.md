  Multi-Modal Meeting Assistant

> An AI-powered backend engine that turns raw meeting recordings into actionable intelligence — transcripts with speaker labels, executive summaries, structured action items, and risk analysis.

---

##  What It Does

Upload a recorded meeting file (`.mp3` or `.mp4`) and the system will:

1. **Transcribe** the audio with speaker diarization (who said what)
2. **Summarize** the meeting into a concise executive overview
3. **Extract action items** as structured JSON (task, assignee, deadline)
4. **Analyze risks** — flagging unresolved questions, blockers, and disagreements

---

##  Architecture Overview

```
┌──────────────┐       ┌────────────────────┐       ┌──────────────────────┐
│  Audio/Video │──────▶│   transcriber.py   │──────▶│ langchain_workflow.py│
│  (.mp3/.mp4) │       │  (AssemblyAI SDK)  │       │  (LangChain + Groq)  │
└──────────────┘       └────────────────────┘       └──────────┬───────────┘
                              │                                │
                              ▼                                ▼
                     Diarized Transcript             3 Parallel LLM Chains
                     (Speaker A, B, …)              ┌─────────────────────┐
                                                    │ • Executive Summary │
                                                    │ • Action Items JSON │
                                                    │ • Risk Analysis     │
                                                    └─────────────────────┘
```

### Module Breakdown

| Module | Role | Tech |
|---|---|---|
| `transcriber.py` | Speech-to-text + speaker diarization | AssemblyAI SDK |
| `langchain_workflow.py` | Meeting intelligence extraction | LangChain + Groq (Llama 4 Maverick) |

---

##  End-to-End Workflow (Example)

### Step 1 — Transcription

**Input:** A 10-minute team standup recording (`standup.mp3`)

**Process:** The file is sent to AssemblyAI, which returns a transcript with speaker labels.

**Output:**
```
Speaker A: Good morning everyone. Let's go over the sprint status.
Speaker B: The auth module is done. I'll start on the dashboard today.
Speaker A: Great. What about the payment integration?
Speaker B: I'm blocked — still waiting on the Stripe API keys from DevOps.
Speaker A: I'll follow up with them. Can we get it resolved by Wednesday?
Speaker B: If I get the keys by tomorrow, yes.
Speaker A: Alright. Any other concerns?
Speaker B: We haven't finalized the database schema for user profiles. That could be a risk.
Speaker A: Noted. Let's schedule a design review for Thursday.
```

### Step 2 — Executive Summary

The transcript is fed into the **Summary Chain** (Groq LLM).

**Output:**
> The team conducted a sprint standup reviewing current progress. The authentication
> module has been completed and work on the dashboard will begin immediately. A key
> blocker was identified around the Stripe API keys needed for payment integration,
> with a follow-up planned with DevOps. The team also flagged an unresolved database
> schema decision for user profiles, and a design review has been scheduled for Thursday
> to address this gap.

### Step 3 — Action Items (JSON)

The transcript is fed into the **Action Items Chain**.

**Output:**
```json
[
  {
    "task": "Follow up with DevOps for Stripe API keys",
    "assignee": "Speaker A",
    "deadline": "Tomorrow"
  },
  {
    "task": "Start working on the dashboard",
    "assignee": "Speaker B",
    "deadline": "Today"
  },
  {
    "task": "Complete payment integration",
    "assignee": "Speaker B",
    "deadline": "Wednesday"
  },
  {
    "task": "Schedule and conduct database schema design review",
    "assignee": "Speaker A",
    "deadline": "Thursday"
  }
]
```

### Step 4 — Risk Analysis

The transcript is fed into the **Risk Analysis Chain**.

**Output:**
> **Blockers**
> - Speaker B is blocked on Stripe API keys from DevOps — payment integration cannot proceed.
>
> **Unresolved Decisions**
> - Database schema for user profiles has not been finalized.
>
> **Dependencies**
> - Payment integration deadline (Wednesday) depends on receiving API keys by tomorrow.
>
> **Potential Risks**
> - If the design review on Thursday surfaces major schema changes, it could impact the sprint timeline.

---

##  Quick Start

### 1. Clone & Set Up

```bash
# Navigate to the project
cd CodeApex_wave3.0

# Create virtual environment (already done if you cloned this repo)
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Configure API Keys

Copy the example env file and add your keys:

```bash
cp .env.example .env
```

Edit `.env`:
```
ASSEMBLYAI_API_KEY=your_assemblyai_key_here
GROQ_API_KEY=your_groq_key_here
```

| Key | Where to Get It |
|---|---|
| `ASSEMBLYAI_API_KEY` | [assemblyai.com/dashboard](https://www.assemblyai.com/dashboard) |
| `GROQ_API_KEY` | [console.groq.com/keys](https://console.groq.com/keys) |

### 3. Run It

```python
from transcriber import transcribe_audio
from langchain_workflow import get_executive_summary, get_action_items, get_risk_analysis

# Step 1: Transcribe
transcript = transcribe_audio("meeting.mp3")
print(transcript)

# Step 2: Analyze
print(get_executive_summary(transcript))
print(get_action_items(transcript))
print(get_risk_analysis(transcript))
```

Or from the CLI:
```bash
# Transcribe only
python transcriber.py meeting.mp3

# Full pipeline with sample data
python langchain_workflow.py
```

---

##  Project Structure

```
CodeApex_wave3.0/
├── transcriber.py           # Speech-to-text + speaker diarization
├── langchain_workflow.py    # LangChain intelligence engine (3 chains)
├── requirements.txt         # Python dependencies
├── .env.example             # API key template
├── .env                     # Actual API keys (gitignored)
├── README.md                # This file
└── venv/                    # Virtual environment
```

---

##  Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Transcription | **AssemblyAI** | Industry-grade STT with speaker diarization |
| LLM Inference | **Groq** (Llama 4 Maverick) | Ultra-fast inference for meeting analysis |
| Orchestration | **LangChain** (LCEL) | Prompt management & LLM chaining |
| Config | **python-dotenv** | Secure API key management |

---

##  For Evaluators

### Why This Architecture?

1. **Modular Design** — Transcription and intelligence are completely decoupled. Swap AssemblyAI for Whisper, or Groq for OpenAI, with zero changes to the other module.
2. **Production-Ready Prompts** — Each LLM chain uses a carefully engineered system prompt with structured output instructions (JSON for action items, markdown for analysis).
3. **Speaker-Aware Intelligence** — Action items are attributed to specific speakers, not just extracted generically. This is possible because we pipe the diarized transcript directly into the LLM.
4. **Scalable to UI** — The backend is UI-agnostic by design. It can be plugged into Streamlit, a React dashboard, a Slack bot, or a REST API.

### Key Talking Points

- "We use **AssemblyAI's speaker diarization** so the AI knows *who* said what — this is critical for assigning action items to the right person."
- "The intelligence layer uses **three specialized LangChain chains**, each with a tailored system prompt, rather than one generic prompt — this gives much higher quality results."
- "We chose **Groq** for inference because it provides the fastest available LLM inference, making the entire pipeline feel near-real-time."
- "The architecture is **modular and UI-agnostic** — we can plug in any frontend without touching the core logic."

---

##  License

Built for CodeApex Wave 3.0 Hackathon.
