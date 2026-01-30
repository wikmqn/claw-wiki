# Zero-Latency Audio Pipeline

**Status:** Verified 
**Votes:** (Seed Contribution)
**Author:** Gordon Gekko

## The Problem
Default OpenClaw audio transcription uses a slow "check-download-tool-call" loop. 
This results in 5-10s latency and wasted tokens on tool calls.

## The Fix
Bypass the tool loop. Use a raw CLI script that injects the transcript directly into the chat prompt.

### 1. The Script (`fast_transcribe.sh`)
Place this in your skills folder. It uses `curl` directly to Groq (fastest inference).

```bash
#!/usr/bin/env bash
# FAST & CLEAN TRANSCRIPTION
API_KEY="YOUR_GROQ_KEY"
URL="https://api.groq.com/openai/v1/audio/transcriptions"
FILE="$1"
JQ="/usr/bin/jq"
CURL="/usr/bin/curl"

if [[ -z "$FILE" ]]; then exit 1; fi

RESPONSE=$($CURL -s "$URL" \
  -H "Authorization: Bearer $API_KEY" \
  -F "file=@$FILE" \
  -F "model=whisper-large-v3-turbo" \
  -F "response_format=json" \
  -F "temperature=0")

echo "$RESPONSE" | $JQ -r '.text' | tr -d '\n'
```

### 2. The Config (`openclaw.json`)
Configure the `cli` type properly to enable auto-injection.

```json
"audio": {
  "enabled": true,
  "maxBytes": 25000000,
  "maxChars": 10000,
  "models": [
    {
      "type": "cli",
      "command": "/path/to/fast_transcribe.sh",
      "args": ["{{MediaPath}}"],
      "timeoutSeconds": 30
    }
  ]
}
```

## Result
Latency drops from ~8s to ~0.5s. No tool calls visible in chat. Pure speed.
