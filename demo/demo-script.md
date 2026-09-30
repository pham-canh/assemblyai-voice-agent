# Demo script — 90 seconds (judge self-run + filming guide)

Video is hosted on YouTube (unlisted), never committed to git.
Target: 60–120s. Show the running product, not code.

## Timeline

```text
00:00–00:10  Problem: elderly patients give up on tiny-text apps.
00:10–00:25  Patient speaks (mic or sample file demo/audio/sample_en.wav).
00:25–00:50  AssemblyAI hears → harness parses → availability tool runs.
00:50–01:15  Agent reads back aloud → patient says YES → BOOKED + .ics.
01:15–01:30  Architecture + why AssemblyAI is load-bearing (remove it: demo is deaf).
```

## Judge self-run (2 minutes, no setup)

```bash
echo "[]" > data/bookings.json
python src/main.py "Book a cardiology checkup tomorrow morning at 9"
```

Browser path:

1. Open the live demo URL, type `Book a cardiology checkup tomorrow morning` → Listen.
2. Type `9 o'clock` → confirmation card appears.
3. Press `YES, BOOK IT` → `APPOINTMENT BOOKED` + download `.ics`.
4. Open the Technical expander: backend, entities, per-stage latency.

## Edge cases to show (optional, 30s extra)

```bash
python src/main.py "Book a checkup tomorrow morning at 9" --confirm=""
# → asks specialty first, never guesses

python src/main.py "uhh hmm ..."
# → hands off to the nurse hotline, never books

python src/main.py "Book cardiology day after tomorrow at 9" \
  --barge-in="No, 10 o'clock instead" --confirm="yes"
# → barge-in: newer utterance wins, books 10:00
```

## Voice paths (need API key)

```bash
python src/realtime.py --src demo/audio/sample_en.wav
python src/agent_api.py --print-session   # offline OK
python src/agent_api.py --src demo/audio/sample_ba.wav   # live voice, needs key
```

## Subtitle cues (EN)

1. Speak to book your clinic visit
2. Real-time speech recognition (AssemblyAI) + entities + confidence
3. Barge-in: agent stops, uses the newer utterance
4. Low confidence → human hotline, never guesses
5. Books only on YES + `.ics` calendar file
6. Remove AssemblyAI → demo is deaf (load-bearing)
