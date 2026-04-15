# Telegram Integration (Personal Account Messaging)

## What it does

Read and write Telegram messages as the user's personal account via Telethon (Python MTProto client). Useful for direct engagement with contacts through Telegram without manual copy-paste. Messages appear as coming from the user's real account, not a bot.

## Auth

Uses the Telegram Client API (MTProto protocol), NOT the Bot API. This means messages come from your real account.

```
API credentials:  .env → TELEGRAM_API_ID, TELEGRAM_API_HASH
Session file:     telegram_session.session (gitignored, persists auth)
```

### Setup

1. Go to https://my.telegram.org → log in with your phone number
2. Click **API development tools** → create a new application
3. Copy the **API ID** (number) and **API Hash** (string)
4. Add to `.env`:
   ```
   TELEGRAM_API_ID=your_api_id
   TELEGRAM_API_HASH=your_api_hash
   ```
5. Install Telethon: `.venv/bin/pip install telethon`
6. Run the one-time auth script (see below). It asks for your phone number and a verification code sent via Telegram.
7. A session file is created locally. After this, all scripts reuse the session without re-authenticating.

### One-time auth script

```python
#!/usr/bin/env python3
import os
from dotenv import load_dotenv
from telethon.sync import TelegramClient

load_dotenv()
client = TelegramClient(
    'telegram_session',
    int(os.environ['TELEGRAM_API_ID']),
    os.environ['TELEGRAM_API_HASH']
)
client.start()
me = client.get_me()
print(f'Authenticated as: {me.first_name} (@{me.username or "no username"})')
client.disconnect()
```

## Common Operations

### List recent chats

```python
from telethon.sync import TelegramClient
client = TelegramClient('telegram_session', API_ID, API_HASH)
client.start()

dialogs = client.get_dialogs(limit=20)
for d in dialogs:
    print(f'{d.name} (id: {d.id})')
```

### Read messages from a chat

```python
messages = client.get_messages(CHAT_ID, limit=10)
for msg in reversed(messages):
    sender = 'You' if msg.out else 'Them'
    print(f'[{msg.date}] {sender}: {msg.text or "[non-text]"}')
```

### Send a message

```python
client.send_message(CHAT_ID, 'Your message here')
```

### Search messages in a chat

```python
messages = client.get_messages(CHAT_ID, limit=50, search='keyword')
```

### Download voice notes

```python
messages = client.get_messages(CHAT_ID, limit=10)
for msg in messages:
    if msg.voice:
        path = client.download_media(msg, file='voice_notes/')
        print(f'Downloaded: {path}')
```

### Find a chat by name

```python
dialogs = client.get_dialogs(limit=200)
for d in dialogs:
    if 'search_term' in d.name.lower():
        print(f'{d.name} (id: {d.id})')
```

## Gotchas

- The session file is your logged-in state. **Treat it like a password.** Add it to `.gitignore`.
- Messages sent via Telethon appear as coming from your real account. The recipient has no idea an API is involved.
- my.telegram.org has aggressive rate limiting on login attempts. If you get "too many tries", wait 24 hours. Do not retry, each attempt resets the timer.
- Telethon uses the MTProto protocol (same as Telegram Desktop). This is NOT the Bot API.
- Rate limits apply to message sending. Don't send hundreds of messages in quick succession.
- Voice notes download as `.ogg` files. Use Whisper or similar for transcription.
- Entity IDs (chat IDs) are stable. Once you have one, it doesn't change.
- The Client API requires a personal phone number. Each phone number can only have one active session per API ID. Running from a second machine may invalidate the first session.
- Always call `client.disconnect()` when done to clean up the connection.
