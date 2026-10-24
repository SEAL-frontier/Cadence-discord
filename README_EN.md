# Cadence Discord

English | [Tiếng Việt](README.md)

Lightweight. OpenAI-compatible. Simple. Basically an OpenAI wrapper with minimal security and minimal memory, with multimodal capability.

Maintained under **NeuralSeal**.

---

## What This Actually Is

Cadence is a Discord chatbot that sits between your server channels and an OpenAI-compatible `/v1/chat/completions` endpoint. It does not pretend to be an enterprise agent system or a bloated RAG stack. It catches a message, formats the payload, streams the response back into the channel, and gets out of the way.

### Core Architecture

- **OpenAI-Compatible Streaming**: Native Server-Sent Events (SSE) streaming over `aiohttp`. Works with official OpenAI models or local backends (vLLM, Ollama, llama.cpp server). Parses line-by-line while tracking codeblocks (` ``` `) so code formatting does not tear mid-stream.
- **Multimodal (Vision)**: Detects Discord image attachments, enforces a 5MB per-image safety ceiling, caches base64 data URLs in an in-memory LRU cache, and structures them into standard OpenAI Vision payload arrays.
- **Minimal Rolling Memory**: Stores up to 6 messages per user in a flat `chat_memory.json` file. It gives just enough short-term conversational context without bloating tokens or blowing up disk space.
- **Message Sanitizer**: Includes a dedicated role sanitizer built because local backends (notably `llama.cpp`) reject consecutive duplicate roles or orphaned assistant messages. Consecutive same-role messages are merged (handling both plain text and vision objects).
- **Trigger Options**: Responds to direct mentions (`@bot`), replies to bot messages, or regex-delimited keywords (`TRIGGER_KEYWORDS`) matching on word boundaries (`\b`).
- **Presence Rotation**: Cycles through game titles defined in `games.json` every hour.
- **Memory Purge**: Simple `/purgememory` slash command (`me`, `user`, `all`) to reset stored context on demand.

---

## Prerequisites

- Python 3.10+
- Discord Bot Token with **Message Content Intent** enabled in the Discord Developer Portal
- OpenAI API Key or a self-hosted OpenAI-compatible endpoint URL

---

## Setup

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Configure Environment
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

Set your credentials:
```env
DISCORD_TOKEN=your_bot_token_here
OPENAI_API_KEY=your_openai_or_local_api_key
```

Optional environment overrides:
```env
MODEL_NAME=gpt-4o
API_ENDPOINT=https://api.openai.com/v1/chat/completions
TRIGGER_KEYWORDS=cadence,bot
ADMIN_IDS=123456789012345678
```

### 3. Configuration Files

- **`config.json`**: Base fallback configuration:
  ```json
  {
      "trigger_keywords": ["cadence"],
      "admin_ids": [],
      "system_prompt": "",
      "model_name": "gpt-4o",
      "api_endpoint": "https://api.openai.com/v1/chat/completions"
  }
  ```
  *(Note: `trigger_keywords` will match exact word boundaries. If left empty and no mention is used, the bot only responds to pings/replies).*

- **`system_prompt.txt`**: Highest priority for bot personality. If this file exists in the root directory, its content overrides `SYSTEM_PROMPT` in `.env` and `config.json`.

- **`games.json`**: List of presence titles the bot rotates through hourly:
  ```json
  [
      {"title": "Minecraft"},
      {"title": "Half-Life 2"}
  ]
  ```

### 4. Run the Bot
```bash
python bot.py
```

---

## File Structure

```
Cadence-discord/
├── bot.py             # Core Discord bot logic, streaming handlers, and events
├── config.json        # Base fallback configuration
├── system_prompt.txt  # System prompt text (highest priority)
├── games.json         # Hourly rotating presence entries
├── chat_memory.json   # Flat JSON storage for 6-turn rolling history
├── .env.example       # Environment template
└── requirements.txt   # discord.py, python-dotenv, aiohttp
```

---

## License & Attribution

MIT License. Maintained under **NeuralSeal**. Originally inspired by Connor's chatbot structure.
