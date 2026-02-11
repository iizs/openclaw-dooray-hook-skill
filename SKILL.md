---
name: dooray-hook
description: Send messages to Dooray chat rooms via webhook.
homepage: https://dooray.com
metadata:
  {
    "openclaw":
      { "emoji": "📨", "requires": { "bins": ["python3"] } },
  }
---

# Dooray Webhook Skill

Send messages to Dooray chat rooms using incoming webhooks.

## Overview

This skill enables sending text messages to configured Dooray chat rooms. Each room has a unique webhook URL stored in the OpenClaw config.

## Configuration

Dooray room webhook URLs are stored in the OpenClaw config at `skills.entries.dooray.config.rooms`:

```json
{
  "skills": {
    "entries": {
      "dooray": {
        "enabled": true,
        "config": {
          "rooms": {
            "RoomName": "https://hook.dooray.com/services/...",
            "AnotherRoom": "https://hook.dooray.com/services/..."
          },
          "botName": "N.I.C.K.",
          "botIconImage": "https://static.dooray.com/static_images/dooray-bot.png"
        }
      }
    }
  }
}
```

### Adding a New Room

To add a new Dooray chat room:

1. Get the webhook URL from Dooray (Settings → Incoming Webhook)
2. Use the gateway config.patch tool to add the room:

```python
from gateway import config.patch

gateway.config.patch({
  "skills": {
    "entries": {
      "dooray": {
        "config": {
          "rooms": {
            "NewRoomName": "https://hook.dooray.com/services/YOUR_WEBHOOK_TOKEN"
          }
        }
      }
    }
  }
})
```

## Usage

### Sending a Message

Run the send script with the room name and message:

```bash
python scripts/send_dooray.py "RoomName" "Your message here"
```

The script will:
1. Load the room webhook URL from OpenClaw config
2. Send a POST request to the Dooray webhook
3. Return success/failure status

### Examples

**Example 1: Simple notification**
```bash
python scripts/send_dooray.py "Dev Team" "Deployment completed successfully ✅"
```

**Example 2: Status update**
```bash
python scripts/send_dooray.py "Operations" "Server CPU usage is high (85%)"
```

## Webhook Payload

The script sends a JSON payload with the following structure:

```json
{
  "botName": "N.I.C.K.",
  "botIconImage": "https://static.dooray.com/static_images/dooray-bot.png",
  "text": "Your message here"
}
```

- `botName`: Name displayed in Dooray (from config, default: "OpenClaw")
- `botIconImage`: Bot avatar URL (from config, optional)
- `text`: Plain text message (markdown not supported by Dooray)

## Implementation

The skill uses only Python standard library modules:
- `urllib.request` for HTTP POST
- `json` for payload serialization
- `os`, `sys` for config path resolution

No external dependencies or venv required.

## Reference

For detailed Dooray webhook API specification, see [references/dooray-api.md](references/dooray-api.md).

## Troubleshooting

**Room not found:**
- Verify the room name exists in `skills.entries.dooray.config.rooms`
- Run `/config` command to inspect current config

**Webhook fails:**
- Check webhook URL is valid (starts with `https://hook.dooray.com/services/`)
- Test the webhook URL directly with curl
- Ensure the Dooray incoming webhook is still active

**Config not found:**
- Ensure OpenClaw config exists at `~/.openclaw/openclaw.json`
- Verify `skills.entries.dooray` section is present
