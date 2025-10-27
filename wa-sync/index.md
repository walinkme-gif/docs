---
layout: default
title: Home
nav_order: 1
---

# WA-Sync Documentation

**Version:** 1.0.0  
**Last Updated:** October 27, 2025

{: .fs-9 }
Transform WhatsApp Web into a real-time webhook API
{: .fs-6 .fw-300 }

[Get Started](#quick-start){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }

---

## What is WA-Sync?

WA-Sync is a powerful Chrome extension that bridges WhatsApp Web with your application by forwarding all WhatsApp events to your configured webhook endpoint in real-time. It captures every interaction, message, status change, and more, sending structured JSON payloads to your server.

### Key Features

✨ **Real-Time Event Streaming**
- 16+ different event types captured instantly
- Message delivery, reads, edits, deletes, reactions, and more
- Connection status, calls, battery updates

📦 **Batch Synchronization**
- Scheduled full chat history sync (hourly/daily)
- Up to 1000 messages per chat
- Configurable batch sizes and parallel processing

🎯 **Flexible Filtering**
- Filter by message type, contact, or chat
- Exclude your own messages
- Custom event type selection

🔄 **Reliable Delivery**
- Automatic retry with configurable attempts
- Custom timeout settings
- Comprehensive error handling

🔒 **Secure & Configurable**
- Custom webhook headers for authentication
- HTTPS support
- Instance-based identification

---

## Quick Start

### 1. Install the Extension

1. Download the extension from Chrome Web Store or load unpacked
2. Pin the extension to your toolbar
3. Open WhatsApp Web

### 2. Configure Your Webhook

1. Click the WA-Sync extension icon
2. Go to Settings
3. Enter your webhook URL: `https://your-api.com/webhook`
4. Add custom headers if needed (e.g., API keys)
5. Click "Save Configuration"

### 3. Test Your Connection

1. Click "Test Webhook" button
2. Check your server receives the test payload
3. Enable "Enable Webhook Sync"

### 4. Start Receiving Events

That's it! You'll now receive real-time events from WhatsApp Web.

---

## Documentation Sections

### [📖 Overview](overview.md)

Complete introduction to WA-Sync, configuration options, and webhook setup. Learn about:
- How WA-Sync works
- Webhook configuration
- Supported events overview
- Rate limiting considerations
- Error handling

### [⚡ Real-Time Events](real-time-events.md)

Detailed documentation of all 16+ real-time webhook events with complete payload structures:
- Message events (text, image, video, audio, document, location, contact)
- Message acknowledgments (delivery, read status)
- Message edits and deletions
- Reactions and poll votes
- Connection status and battery updates
- Incoming calls and chat actions

### [🔄 Batch Sync](batch-sync.md)

Learn about periodic full chat synchronization:
- Configuration and scheduling
- Batch payload structure
- Message filtering
- Parallel processing
- Error handling

### [🚀 Getting Started](getting-started.md)

Step-by-step guide to integrate WA-Sync with your application:
- Installation and setup
- Building your webhook endpoint
- Processing events
- Best practices
- Common patterns

### [💡 Examples](examples.md)

Real-world webhook payload examples and integration patterns:
- Sample payloads for all event types
- Server implementation examples (Node.js, Python, PHP)
- Database schema suggestions
- Use case scenarios

---

## Important: Chat IDs & Phone Numbers

⚠️ **WhatsApp uses two ID formats - This is critical for phone number extraction!**

### LID Format (Newer Accounts)
```json
{
  "id": "1234567890@lid",          // LID identifier
  "phoneNumber": "1234567890@c.us" // Phone number here
}
```
**If `id` ends with `@lid`** → Get phone number from `phoneNumber` field

### Classic Format (Traditional Accounts)
```json
{
  "id": "1234567890@c.us",         // ID IS the phone number
  "phoneNumber": "1234567890@c.us" // Same value
}
```
**If `id` ends with `@c.us`** → The `id` itself is the phone number

### Groups & Channels
- Groups: `123456789@g.us` (no phone number)
- Channels: `1234567890@newsletter` (no phone number)

📖 See [Overview](overview.md) for detailed explanation and code examples.

---

## Use Cases

### Customer Support
- Auto-route messages to support agents
- Track response times
- Archive all conversations
- Monitor sentiment

### Business Automation
- Trigger workflows from WhatsApp messages
- Send automated responses
- Integrate with CRM systems
- Track customer interactions

### Analytics & Monitoring
- Message volume tracking
- Response time analysis
- User engagement metrics
- Export chat history

### Backup & Compliance
- Automatic message archival
- Regulatory compliance
- Audit trails
- Data retention

---

## Event Types at a Glance

| Category | Events | Description |
|----------|--------|-------------|
| **Messages** | `message`, `message_ack`, `message_edit`, `message_revoke` | Core messaging events |
| **Interactions** | `reaction`, `poll_vote` | User reactions and polls |
| **Media** | `media_uploaded`, `message_ciphertext` | Media handling |
| **Status** | `app_state_change`, `battery_change` | Connection and device status |
| **Calls** | `incoming_call` | Voice/video calls |
| **Chat Actions** | `chat_archived`, `chat_removed`, `chat_unread_count` | Chat management |
| **Sync** | `chat_sync` | Batch synchronization |

---

## Sample Webhook Payload

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": {
        "fromMe": false,
        "remote": "1234567890@c.us",
        "id": "3EB0ABC123DEF456",
        "_serialized": "false_1234567890@c.us_3EB0ABC123DEF456"
      },
      "ack": 1,
      "body": "Hello, how are you?",
      "type": "chat",
      "timestamp": 1730000000000,
      "from": "1234567890@c.us",
      "to": "0987654321@c.us",
      "notifyName": "John Doe",
      "fromMe": false,
      "hasMedia": false,
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      },
      "toChat": {
        "id": "0987654321@c.us",
        "phoneNumber": "+0987654321"
      }
    }
  }
}
```

---

## Requirements

- Chrome Browser (version 90+)
- Active WhatsApp Web session
- HTTPS webhook endpoint (recommended for production)
- Server capable of handling POST requests

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Next Steps

1. 📖 Read the [Overview](overview.md) to understand how WA-Sync works
2. ⚡ Explore [Real-Time Events](real-time-events.md) for event payload structures
3. 🚀 Follow the [Getting Started](getting-started.md) guide to build your integration
4. 💡 Check out [Examples](examples.md) for implementation patterns

---

**Ready to get started?** [Configure your webhook →](getting-started.md)
