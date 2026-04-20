# Chatwoot Integration (Customer Messaging Inbox)

## What it does

Self-hosted customer messaging platform. Handles WhatsApp, email, and other channel conversations via a unified inbox. Can be accessed via web and iOS/Android apps.

## Auth

### Web/API login

```
POST https://YOUR_CHATWOOT_URL/auth/sign_in
Content-Type: application/json

{"email": "admin@example.com", "password": "..."}
```

Returns `access_token` in `data.access_token`. Use it as:

```
Header: api_access_token: [TOKEN]
```

## Common Operations

### List conversations

```bash
curl -s -H "api_access_token: $TOKEN" \
  "https://YOUR_CHATWOOT_URL/api/v1/accounts/1/conversations"
```

### Get messages in a conversation

```bash
curl -s -H "api_access_token: $TOKEN" \
  "https://YOUR_CHATWOOT_URL/api/v1/accounts/1/conversations/CONV_ID/messages"
```

### Send a reply

```bash
curl -s -X POST -H "api_access_token: $TOKEN" \
  -H "Content-Type: application/json" \
  "https://YOUR_CHATWOOT_URL/api/v1/accounts/1/conversations/CONV_ID/messages" \
  -d '{"content": "Hello!", "message_type": "outgoing"}'
```

### Create a WhatsApp inbox

```bash
curl -s -X POST -H "api_access_token: $TOKEN" \
  -H "Content-Type: application/json" \
  "https://YOUR_CHATWOOT_URL/api/v1/accounts/1/inboxes" \
  -d '{
    "name": "WhatsApp",
    "channel": {
      "type": "whatsapp",
      "phone_number": "+1XXXXXXXXXX",
      "provider": "whatsapp_cloud",
      "provider_config": {
        "api_key": "META_ACCESS_TOKEN",
        "phone_number_id": "PHONE_NUMBER_ID",
        "business_account_id": "WABA_ID"
      }
    }
  }'
```

## Rails Console

For operations not available via API:

```bash
ssh user@server
cd /opt/chatwoot
docker compose exec rails bundle exec rails console
```

### Fix inactive WhatsApp channel

```ruby
channel = Channel::Whatsapp.find_by(phone_number: '+1XXXXXXXXXX')
channel.reauthorized! if channel.reauthorization_required?
```

### Create a new user

```ruby
user = User.new(name: 'Name', email: 'email@example.com', password: 'password')
user.skip_reconfirmation!
user.save!
AccountUser.create!(account_id: 1, user_id: user.id, role: :agent)
```

## Gotchas

- **Reauthorization flag is silent.** When Chatwoot creates a WhatsApp inbox, it tries to auto-register the webhook via Meta API. If that fails, it marks the channel as "inactive" and silently drops incoming messages. Fix with `channel.reauthorized!` in Rails console.
- **The reauthorization flag lives in Redis**, not Postgres. Restarting containers doesn't clear it.
- **Conversations don't appear in "Mine" until assigned.** New agents must be added as inbox members and conversations must be assigned. Enable `enable_auto_assignment` on the inbox.
- **Devise email confirmation blocks email changes.** Use `user.skip_reconfirmation!` before saving when SMTP isn't configured.
- **Chatwoot has active CVE history.** Stay current — patch within a week of releases.
- **First boot is slow.** Rails takes 30-60 seconds on small VPS. Expect 502s during boot.
- **Webhook verify token is per-inbox.** If you recreate the inbox, you get a new token and must update the webhook config.
