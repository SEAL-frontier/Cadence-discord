# (Blueprint) Slingshot

[English](README_EN.md) | Tiếng Việt

Gọn nhẹ. Tương thích chuẩn OpenAI. Đơn giản. Về cơ bản là một OpenAI wrapper với bảo mật tối thiểu, bộ nhớ tối thiểu, và có hỗ trợ multimodal (hình ảnh).

Dự án thuộc **NeuralSeal**.

---

## Bản chất project

Cadence là bot Discord nằm ở giữa channel chat và endpoint `/v1/chat/completions` tương thích OpenAI. Nó không màu mè thành multi-agent framework hay hệ thống RAG cồng kềnh: nhận tin nhắn, đóng gói payload, stream câu trả lời về lại channel rồi nghỉ.

### Đặc điểm cốt lõi

- **Streaming tương thích OpenAI**: Dùng `aiohttp` bắt stream Server-Sent Events (SSE). Chạy được với OpenAI xịn lẫn các backend local (vLLM, Ollama, llama.cpp server). Code tự đếm dấu backtick (` ``` `) để nhả từng dòng mà không làm vỡ codeblock Markdown giữa chừng.
- **Multimodal (Vision)**: Tự bắt file ảnh đính kèm trên Discord, kiểm tra MIME type, chặn ảnh trên 5MB, cache Base64 trong RAM (tối đa 32 ảnh) và format thành mảng `image_url` chuẩn OpenAI Vision.
- **Bộ nhớ siêu tối giản**: Lưu tối đa 6 tin nhắn gần nhất của mỗi user vào một file JSON phẳng (`chat_memory.json`). Đủ để bot không bị mất trí nhớ tạm thời sau 1 câu, nhưng cũng không phình context làm tốn tiền token hay nghẽn ổ đĩa.
- **Sanitizer chống sập role**: Viết riêng một hàm xử lý role vì các backend local (đặc biệt là `llama.cpp`) sẽ văng lỗi ngay nếu thấy 2 message cùng role đứng cạnh nhau hoặc mở đầu bằng role assistant. Hàm này tự gộp nội dung các role trùng nhau (hỗ trợ cả text thường lẫn object vision) trước khi bắn sang API.
- **Trigger linh hoạt**: Nhận diện khi được tag trực tiếp (`@bot`), khi reply tin nhắn của bot, hoặc khi câu chat chứa từ khoá cấu hình trong `TRIGGER_KEYWORDS` (khớp theo ranh giới từ `\b`).
- **Xoay trạng thái tự động**: Đổi game presence mỗi tiếng một lần dựa theo danh sách trong `games.json`.
- **Dọn dẹp bộ nhớ**: Slash command `/purgememory` (`me`, `user`, `all`) để tự xoá lịch sử chat khi cần.

---

## Yêu cầu

- Python 3.10+
- Discord Bot Token đã bật **Message Content Intent** trong Discord Developer Portal
- API Key OpenAI hoặc URL endpoint local tương thích chuẩn OpenAI

---

## Cài đặt & Triển khai

### 1. Cài dependencies
```bash
pip install -r requirements.txt
```

### 2. Cấu hình file môi trường
Copy `.env.example` thành `.env`:
```bash
cp .env.example .env
```

Điền token và key:
```env
DISCORD_TOKEN=token_bot_cua_ban
OPENAI_API_KEY=key_openai_hoac_local
```

Các biến tuỳ chọn có thể ghi đè:
```env
MODEL_NAME=gpt-4o
API_ENDPOINT=https://api.openai.com/v1/chat/completions
TRIGGER_KEYWORDS=cadence,bot
ADMIN_IDS=123456789012345678
```

### 3. Các file cấu hình

- **`config.json`**: Cấu hình mặc định (fallback khi không set biến môi trường):
  ```json
  {
      "trigger_keywords": ["cadence"],
      "admin_ids": [],
      "system_prompt": "",
      "model_name": "gpt-4o",
      "api_endpoint": "https://api.openai.com/v1/chat/completions"
  }
  ```
  *(Lưu ý: `trigger_keywords` bắt theo từ riêng biệt. Nếu để trống và không ai ping/reply, bot sẽ im lặng).*

- **`system_prompt.txt`**: Định hình tính cách bot. Nếu file này tồn tại trong thư mục gốc, nội dung trong file sẽ được ưu tiên cao nhất, đè lên `SYSTEM_PROMPT` trong `.env` và `config.json`.

- **`games.json`**: Danh sách status bot tự xoay mỗi giờ:
  ```json
  [
      {"title": "Minecraft"},
      {"title": "Half-Life 2"}
  ]
  ```

### 4. Chạy bot
```bash
python bot.py
```

---

## Cấu trúc thư mục

```
Cadence-discord/
├── bot.py             # Logic chính, xử lý streaming, event Discord
├── config.json        # Cấu hình dự phòng mặc định
├── system_prompt.txt  # System prompt (ưu tiên cao nhất)
├── games.json         # Danh sách presence xoay vòng
├── chat_memory.json   # Lưu 6 tin nhắn gần nhất theo user ID
├── .env.example       # Template biến môi trường
└── requirements.txt   # discord.py, python-dotenv, aiohttp
```

---

## Giấy phép & Nguồn gốc

Phát hành theo giấy phép MIT. Dự án duy trì bởi **NeuralSeal**. Khởi nguồn dựa trên kiến trúc bot Discord của Connor.
