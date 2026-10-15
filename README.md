# AssemblyAI Voice Agent — Voice Clinic Booking

Speak to book your clinic visit: listen → confirm aloud → book, with per-stage latency in ms.

## Demo

- Live demo: `https://voicecare-scheduler-lmzkozkdf4rssh4w6vz8p6.streamlit.app/`
- Backup demo (Hugging Face): run `python app.py` (Gradio on `:7860`)

## Video

- 90-second script: `demo/demo-script.md`

## Architecture

```text
User voice
   ↓
AssemblyAI (batch STT / streaming WS / Voice Agent API)
   ↓
Agent harness (src/agent/loop.py — 5-state FSM)
   ↓
Tools (check_availability / book_appointment, YES-gated)
   ↓
Voice + text response (.ics on booking)
```

States: `LISTENING → UNDERSTANDING → CHECKING → CONFIRMATION → BOOKED / ESCALATED`.
The model never books by itself — code writes only after a strict YES.

## Features

- Real-time streaming STT (`u3-rt-pro`, `min_latency`, diarization, keyterms)
- Batch STT fallback for audio files
- Managed Voice Agent API path (one-time token, direct voice WS, TTS, tool loop)
- Gemini NLU with labeled regex fallback (`nlu_source` always visible)
- Barge-in (newer utterance wins) + escalation to nurse hotline
- Per-stage latency in ms, audit log, atomic booking writes

## Tech Stack

- AssemblyAI (STT batch + streaming + voice agents)
- Gemini (`gemini-2.0-flash`, NLU/confirmation/readback)
- Python, Streamlit (main demo), Gradio (backup demo)

## Run locally

```bash
git clone https://github.com/pham-canh/assemblyai-voice-agent.git assemblyai-voice-agent
cd assemblyai-voice-agent
pip install -r requirements.txt
cp .env.example .env   # then fill your keys
echo "[]" > data/bookings.json

python src/main.py "Book a cardiology checkup tomorrow morning at 9"
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests/ -q
```

With a microphone/file upload use the Streamlit app:

```bash
streamlit run app_streamlit.py
```

## API keys needed

- `ASSEMBLYAI_API_KEY` — real STT for `.wav/.mp3/.m4a` (free at https://www.assemblyai.com/). Without it the app still runs as labeled typed try-out.
- `GEMINI_API_KEY` — LLM understanding. Without it a labeled regex fallback is used.

Never commit real keys. See `.env.example`.

## Structure

```text
src/main.py          CLI entry (thin wrapper over agent harness)
src/agent_api.py     managed Voice Agent API path
src/realtime.py      streaming WS STT path
src/tts.py           lightweight TTS (never crashes demo)
src/agent/           harness: loop, llm_client, stt_client, tools, store, config
app_streamlit.py     main demo (Streamlit)
app.py               backup demo (Gradio)
tests/test_agent.py  25 pytest scenarios
demo/                demo script + sample audio + screenshots
```

## License

MIT — see `LICENSE`.
