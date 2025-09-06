# Voice Patrol Bot (Multi-Bot, Always-On)

This project lets you run **multiple Discord bot accounts at once**, with each bot assigned to a specific **voice channel** where it stays online and **monitors voice chat** for banned words. When a violation is detected, it **logs an embed** in a single moderation log channel.

- Multiple bots, one codebase
- Each bot auto-joins a configured VC on startup
- Speech-to-text engine: **Whisper API** (recommended) — or swap in Vosk (offline) by implementing `sttEngine.transcribe()`
- Uses your `badwords.txt` (one per project, shared by all bots)

> **Important:** A single bot account can only join **one VC per guild at a time**. To patrol multiple channels simultaneously, create multiple bot applications/tokens and list them in `config.json`.

---

## 1) Setup

1. **Create one Discord Application per voice channel you want to patrol:**
   - Go to https://discord.com/developers/applications
   - Create Application → Bot → **Privileged Gateway Intents:** Guild Members (optional), **Server Members Intent not required**, enable **Presence** not needed, **Message Content** not needed.
   - Copy each bot **token**.
   - Invite each bot to your server with a URL including the scopes `bot applications.commands` and permissions for **Connect** and **Speak** (Speak is optional), **View Channels**.

2. **Create `config.json`** (copy from `config.example.json`) and fill in:
   - `logChannelId`: the channel where embeds should be sent
   - For each bot: `token`, `guildId`, `voiceChannelId`

3. **Add your bad words** to `data/badwords.txt` (one per line).

4. **Whisper API (recommended):**
   - Set environment variable: `OPENAI_API_KEY=sk-...`
   - The bot records short clips (5s default), converts to WAV, sends to Whisper, gets text back.
   - *You pay per minute of audio.*

5. Install & run:
   ```bash
   npm i
   npm start
   ```

---

## 2) Config

`config.example.json`:
```json
{
  "logChannelId": "1413623155318198353",
  "clipSeconds": 5,
  "engine": "whisper",
  "whisper": {
    "model": "gpt-4o-transcribe",
    "endpoint": "https://api.openai.com/v1/audio/transcriptions"
  },
  "bots": [
    { "token": "BOT_TOKEN_1", "guildId": "YOUR_GUILD_ID", "voiceChannelId": "VC_ID_1" },
    { "token": "BOT_TOKEN_2", "guildId": "YOUR_GUILD_ID", "voiceChannelId": "VC_ID_2" }
  ]
}
```

- `clipSeconds`: length of each audio capture window per user before sending to STT.
- `engine`: currently `"whisper"` is implemented. If you want **Vosk (offline)**, implement `stt/vosk.js` similar to `stt/whisper.js` and set engine to `"vosk"`.

> Tip: Use **Channel IDs** (right click → Copy ID with Developer Mode on).

---

## 3) What gets logged?

When a transcript contains any word from `badwords.txt` (case-insensitive, whole or partial match), the bot sends an **embed** to your mod log with:
- Offender: user tag + ID
- Channel: voice channel name
- Matched word
- Short transcript excerpt
- Timestamp and bot shard (which client)

---

## 4) Notes & Limitations

- Discord only lets a single bot be **in one VC per guild at a time**. This repo runs **multiple bots** to cover multiple channels.
- Whisper transcription requires uploading audio clips — consider privacy and get server mod approval.
- If you prefer fully offline, use **Vosk**. It requires model files (~50–200MB). You can implement it by replacing the `transcribe` function in `stt/whisper.js` with your Vosk code and setting `engine: "vosk"` in config.

---

## 5) Directory

```
voice-patrol-bot/
  data/
    badwords.txt
  src/
    index.js
    voicePatrol.js
    logger.js
    stt/whisper.js
    utils/wav.js
    utils/loadBadwords.js
  package.json
  config.example.json
  README.md
```

---

## 6) Troubleshooting

- **Bot doesn’t join VC:** Check the `voiceChannelId`, permissions, and that the bot is actually in the guild.
- **No logs appear:** Verify `logChannelId` and that the bot can see/send to that channel.
- **Rate limits / cost:** Reduce `clipSeconds` or only monitor active speakers.
- **Double messages:** This code ensures one embed per detection per clip per user to avoid duplication.
