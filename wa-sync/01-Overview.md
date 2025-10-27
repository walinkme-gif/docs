# WA-Sync Webhook API - Overview

**Version:** 1.0.0  
**Last Updated:** October 27, 2025

## Introduction

WA-Sync is a Chrome extension that forwards WhatsApp Web events to your configured webhook endpoint in real-time. It captures every interaction happening in WhatsApp Web and sends structured JSON payloads to your server.

## How It Works

### Architecture

```
WhatsApp Web → Injected Scripts → Content Bridge → Service Worker → Your Webhook
```

1. **Injected Scripts** - Run in WhatsApp Web's page context, capture events
2. **Content Bridge** - Forwards events from page to extension
3. **Service Worker** - Processes events, applies filters, sends to webhook
4. **Your Webhook** - Receives POST requests with JSON payloads

### Two Modes of Operation

#### 1. Real-Time Event Streaming (Auto-Send)
- Events are captured **immediately** as they occur
- Each event is sent individually to your webhook
- Supports 16+ different event types
- Configurable filtering and retry logic

#### 2. Periodic Batch Sync
- Scheduled full synchronization (hourly/daily)
- Retrieves up to 1000 messages per chat
- Sends in batches of 50 messages (configurable)
- Processes multiple chats in parallel

## Webhook Configuration

### Endpoints

| Type | Endpoint | Purpose |
|------|----------|---------|
| Real-Time Events | `{webhookUrl}` | Individual events as they happen |
| Batch Sync | `{webhookUrl}/chat` | Periodic full chat synchronization |

### HTTP Request Format

**Method:** `POST`

**Headers:**
```json
{
  "Content-Type": "application/json",
  "User-Agent": "WhatsApp-Webhook-Sync/1.0",
  "X-Instance-Id": "wa-sync-1730000000000-abc123xyz"
}
```

You can configure custom headers in the extension settings.

### Common Payload Structure

All payloads follow this base structure:

```json
{
  "event": "string",        // Event type (e.g., "message", "reaction")
  "timestamp": 1730000000000,  // Unix timestamp in milliseconds
  "instanceId": "string",   // Unique extension instance identifier
  "data": {}                // Event-specific data (varies by event type)
}
```

## Supported Events

### Real-Time Events (16 types)

| Event | Description | Trigger |
|-------|-------------|---------|
| `message` | New message | Message received/sent |
| `message_ack` | Status change | Sent → Delivered → Read |
| `message_edit` | Message edited | User edits message (15min window) |
| `message_revoke` | Message deleted | User deletes message |
| `message_ciphertext` | Encrypted message | Before decryption |
| `message_change` | Message updated | Any message property changes |
| `message_type_change` | Type changed | Message type changes (rare) |
| `media_uploaded` | Media ready | Media upload completes |
| `reaction` | Emoji reaction | User reacts to message |
| `poll_vote` | Poll response | User votes on poll |
| `app_state_change` | Connection status | WA connects/disconnects |
| `battery_change` | Phone battery | Battery status changes |
| `incoming_call` | Call received | Voice/video call incoming |
| `chat_removed` | Chat deleted | User deletes chat |
| `chat_archived` | Chat archived | User archives/unarchives chat |
| `chat_unread_count` | Unread count | Unread messages change |

### Batch Sync Event

| Event | Description |
|-------|-------------|
| `chat_sync` | Full chat synchronization with message history |

## Configuration Options

The extension provides extensive configuration through the settings page:

### Basic Settings
- **Webhook URL** - Your endpoint URL
- **Enable/Disable** - Master switch for syncing
- **Instance ID** - Unique identifier for this installation

### Filtering
- **Message Types** - Which types to sync (chat, image, video, etc.)
- **Event Types** - Which events to capture
- **Contact Filter** - Sync specific contacts only
- **Chat Filter** - Sync specific chats only
- **Exclude Own Messages** - Don't sync messages you send

### Retry & Performance
- **Retry Attempts** - How many times to retry failed webhooks (default: 3)
- **Retry Delay** - Delay between retries (default: 1000ms)
- **Timeout** - Request timeout (default: 30000ms)

### Custom Headers
- Add authentication headers, API keys, etc.

### Auto-Sync (Batch Sync)
- **Enabled** - Turn on periodic sync
- **Interval** - hourly / every 6 hours / daily
- **Messages Per Batch** - How many messages per request (default: 50)
- **Max Parallel Chats** - Concurrent chat processing (default: 5)
- **Sync Endpoint** - Custom endpoint path (default: `/chat`)

## Important Data Points

### Message Identification

Every message has a unique ID:
```json
{
  "id": {
    "_serialized": "false_1234567890@c.us_3EB0ABC123DEF456"
  }
}
```

or 

```json
{
  "id": "false_1234567890@c.us_3EB0ABC123DEF456"
}
```

Use `id._serialized` || `id` to uniquely identify messages across all events.

### Chat ID Formats

- **Personal Chat:** `1234567890@c.us`
- **Group Chat:** `123456789-1234567890@g.us`
- **Channel:** `1234567890@newsletter`

You can identify chat types by the domain suffix.

### Phone Number Extraction

From the code analysis, phone numbers are available in:
```json
{
  "fromChat": {
    "id": "1234567890@c.us",
    "phoneNumber": "+1234567890"  // May be serialized as object
  },
  "toChat": {
    "id": "0987654321@c.us",
    "phoneNumber": "+0987654321"
  }
}
```

Note: Phone numbers can be in format `"+1234567890"` or as an object `{"_serialized": "+1234567890"}`

### Media Handling

**Important:** The webhook does NOT include media files (images, videos, documents). 

What you receive:
- `mediaKey` - Encryption key
- `size` - File size in bytes
- `mimetype` - MIME type (e.g., "image/jpeg")
- `filename` - Original filename
- `caption` - Media caption text
- `body` - Base64 data is included

## Rate Limiting Considerations

### Real-Time Events
- Events sent immediately (no batching)

### Batch Sync
- Default: 50 messages per request
- 2-second delay between batches
- 5 chats processed in parallel
- A full sync can generate **hundreds of requests**

### Recommendations
- Implement queue-based async processing
- Return 200 OK immediately, process in background
- Use `messageId` for idempotency
- Implement rate limiting on your end

## Error Handling

### HTTP Response Codes

Your webhook should respond with appropriate codes:

- **200-299** - Success (event processed)
- **400-499** - Client error (extension won't retry)
- **500-599** - Server error (extension will retry)

### Retry Logic

The extension implements automatic retry:
1. First attempt fails
2. Wait `retryDelay` milliseconds (default: 1000ms)
3. Retry up to `retryAttempts` times (default: 3)
4. If all attempts fail, event is lost

**Note:** Failed events are NOT queued persistently. If all retries fail, the event is discarded.

## Testing Your Integration

### Test Webhook Feature

The extension includes a test button that sends:

```json
{
  "event": "test",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": "This is a test webhook from WhatsApp Webhook Sync extension"
  }
}
```

Use this to verify:
- Your endpoint is reachable
- Headers are correctly configured
- Authentication is working
- Response handling is correct

## Next Steps

- **[Real-Time Events Documentation](./02-Real-Time-Events.md)** - Detailed payload structures for each event
- **[Batch Sync Documentation](./03-Batch-Sync.md)** - Full chat synchronization details
- **[Data Models](./04-Data-Models.md)** - Complete field reference
- **[Examples](./05-Examples.md)** - Real-world payload examples

---

**Need Help?**
- Check the extension console logs for debugging
- Verify webhook URL is accessible
- Test with the built-in test webhook feature
- Ensure HTTPS is used for production
