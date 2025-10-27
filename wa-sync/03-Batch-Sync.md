# WA-Sync Webhook API - Batch Sync

**Version:** 1.0.0  
**Last Updated:** October 27, 2025

This document details the batch synchronization feature that periodically sends full chat history to your webhook.

---

## Overview

Batch Sync is a scheduled background process that:
1. Retrieves all chats from WhatsApp Web
2. For each chat, fetches up to 1000 messages
3. Sends messages in batches to your webhook
4. Tracks progress and provides status updates

## Configuration

### Settings

```javascript
{
  "autoSync": {
    "enabled": true,                    // Enable/disable batch sync
    "interval": "daily",                // "hourly", "every6hours", "daily"
    "messagesPerBatch": 50,             // Messages per request
    "maxParallelChats": 5,              // Concurrent chat processing
    "syncEndpoint": "/chat",            // Endpoint path
    "lastSyncTime": 1730000000000,      // Last sync timestamp
    "nextSyncTime": 1730086400000       // Next scheduled sync
  }
}
```

### Trigger Methods

1. **Automatic (Scheduled)**
   - Based on `interval` setting
   - Runs in background
   - Updates `lastSyncTime` and `nextSyncTime`

2. **Manual**
   - Triggered from extension popup
   - Immediate execution
   - Doesn't affect automatic schedule

## Endpoint

**URL:** `POST {webhookUrl}/chat`

**Important:** Batch sync uses a different endpoint path than real-time events.

### Full URL Example

If `webhookUrl = "https://api.example.com/whatsapp"` and `syncEndpoint = "/chat"`:
- Real-time events: `https://api.example.com/whatsapp`
- Batch sync: `https://api.example.com/whatsapp/chat`

---

## Payload Structure

### Chat Sync Event

**Event Type:** `chat_sync`

```json
{
  "event": "chat_sync",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "chat": {
    "id": "1234567890@c.us",
    "name": "John Doe",
    "userId": "+1234567890"
  },
  "batch": {
    "number": 1,
    "total": 5,
    "messageCount": 50
  },
  "messages": [ /* array of 50 message objects */ ]
}
```

### Root Fields

| Field | Type | Description |
|-------|------|-------------|
| `event` | string | Always "chat_sync" |
| `timestamp` | number | When batch was created (Unix ms) |
| `instanceId` | string | Extension instance ID |
| `chat` | object | Chat information |
| `batch` | object | Batch metadata |
| `messages` | array | Message objects (up to `messagesPerBatch`) |

### Chat Object

```json
{
  "id": "1234567890@c.us",
  "name": "John Doe",
  "userId": "+1234567890"
}
```

| Field | Type | Description | Examples |
|-------|------|-------------|----------|
| `id` | string | WhatsApp chat ID | `1234567890@c.us` (personal)<br>`123456789@g.us` (group)<br>`123@newsletter` (channel) |
| `name` | string | Chat display name | Contact name, group name, or channel name |
| `userId` | string or null | Phone number | `"+1234567890"` for contacts<br>`null` for groups/channels |

**Note:** `userId` may be a string or an object with `_serialized` property:
```json
"userId": "+1234567890"  // OR
"userId": { "_serialized": "+1234567890" }
```

### Batch Object

```json
{
  "number": 2,
  "total": 5,
  "messageCount": 50
}
```

| Field | Type | Description |
|-------|------|-------------|
| `number` | number | Current batch number (1-indexed) |
| `total` | number | Total batches for this chat |
| `messageCount` | number | Messages in THIS batch |

### Messages Array

Each message has the same structure as real-time `message` events. See [Real-Time Events Documentation](./02-Real-Time-Events.md) for complete message structure.

```json
"messages": [
  {
    "id": {
      "fromMe": false,
      "remote": "1234567890@c.us",
      "id": "3EB0MSG001",
      "_serialized": "false_1234567890@c.us_3EB0MSG001"
    },
    "ack": 3,
    "body": "First message in batch",
    "type": "chat",
    "timestamp": 1729800000000,
    "from": "1234567890@c.us",
    "to": "0987654321@c.us",
    "fromMe": false,
    "hasMedia": false,
    "fromChat": {
      "id": "1234567890@c.us",
      "phoneNumber": "+1234567890"
    },
    "toChat": {
      "id": "0987654321@c.us",
      "phoneNumber": "+0987654321"
    }
  },
  {
    "id": {
      "fromMe": true,
      "remote": "1234567890@c.us",
      "id": "3EB0MSG002",
      "_serialized": "true_1234567890@c.us_3EB0MSG002"
    },
    "ack": 4,
    "body": "This is my reply",
    "type": "chat",
    "timestamp": 1729801000000,
    "from": "0987654321@c.us",
    "to": "1234567890@c.us",
    "fromMe": true,
    "hasMedia": false,
    "fromChat": {
      "id": "0987654321@c.us",
      "phoneNumber": "+0987654321"
    },
    "toChat": {
      "id": "1234567890@c.us",
      "phoneNumber": "+1234567890"
    }
  }
  // ... up to 50 messages total
]
```

---

## Batching Logic

### Example: Chat with 237 Messages

**Configuration:**
- `messagesPerBatch = 50`
- Up to 1000 messages retrieved per chat

**Result: 5 Batch Requests**

#### Batch 1 of 5
```json
{
  "batch": {
    "number": 1,
    "total": 5,
    "messageCount": 50
  },
  "messages": [ /* 50 messages */ ]
}
```

#### Batch 2 of 5
```json
{
  "batch": {
    "number": 2,
    "total": 5,
    "messageCount": 50
  },
  "messages": [ /* 50 messages */ ]
}
```

#### Batch 3 of 5
```json
{
  "batch": {
    "number": 3,
    "total": 5,
    "messageCount": 50
  },
  "messages": [ /* 50 messages */ ]
}
```

#### Batch 4 of 5
```json
{
  "batch": {
    "number": 4,
    "total": 5,
    "messageCount": 50
  },
  "messages": [ /* 50 messages */ ]
}
```

#### Batch 5 of 5 (Final - Partial)
```json
{
  "batch": {
    "number": 5,
    "total": 5,
    "messageCount": 37
  },
  "messages": [ /* 37 messages */ ]
}
```

### Timing Between Batches

- **2-second delay** between batches for the same chat
- Prevents rate limiting
- Allows server processing time

### Example Timeline

```
00:00 - Batch 1/5 sent → 200 OK
00:02 - Batch 2/5 sent → 200 OK (2s delay)
00:04 - Batch 3/5 sent → 200 OK (2s delay)
00:06 - Batch 4/5 sent → 200 OK (2s delay)
00:08 - Batch 5/5 sent → 200 OK (2s delay)
00:10 - Chat complete, move to next chat
```

---

## Parallel Processing

### Multiple Chats Simultaneously

**Configuration:** `maxParallelChats = 5`

Up to 5 chats are processed at the same time:

```
Chat A: [Batch 1] → [Batch 2] → [Batch 3] → Done
Chat B: [Batch 1] → [Batch 2] → Done
Chat C: [Batch 1] → [Batch 2] → [Batch 3] → [Batch 4] → Done
Chat D: [Batch 1] → Done
Chat E: [Batch 1] → [Batch 2] → [Batch 3] → Done
```

All running concurrently, each with 2s delays between their own batches.

---

## Message Filtering

Messages are filtered based on configuration before batching:

### Applied Filters

1. **Message Type Filter**
   ```javascript
   messageTypes: ["chat", "image", "video", "audio", "ptt", "document"]
   ```
   Only messages matching these types are included.

2. **Contact Filter**
   ```javascript
   filterContacts: ["1234567890@c.us", "0987654321@c.us"]
   ```
   If set, only messages from these contacts are synced.

3. **Chat Filter**
   ```javascript
   filterChats: ["123456789@g.us"]
   ```
   If set, only these chats are synchronized.

4. **Exclude Own Messages**
   ```javascript
   excludeOwnMessages: true
   ```
   If true, messages where `fromMe = true` are excluded.

### Filter Application

Filters are applied in `config.shouldJobSyncMessage(message)`:

```javascript
// Pseudocode
for each message in chat {
  if (excludeOwnMessages && message.fromMe) skip;
  if (messageTypes doesn't include message.type) skip;
  if (filterContacts set && message.from not in filterContacts) skip;
  if (filterChats set && message.chatId not in filterChats) skip;
  
  include message in batch;
}
```

---

## Full Sync Workflow

### Step-by-Step Process

1. **Trigger**
   - Automatic (scheduled) or manual trigger
   - Creates new `SyncJob` with unique ID

2. **Get All Chats**
   - Query WhatsApp Web for all chats
   - Returns chat list with IDs and names
   - Update job with total chat count

3. **Process Chats**
   - Up to `maxParallelChats` processed simultaneously
   - For each chat:

4. **Get Chat Messages**
   - Fetch up to 1000 messages from chat
   - Apply filters (type, contacts, own messages)
   - Calculate batch count

5. **Send Batches**
   - Divide messages into batches of `messagesPerBatch`
   - Send each batch to `{webhookUrl}/chat`
   - Wait 2 seconds between batches

6. **Track Progress**
   - Update chat status (pending → in_progress → completed/failed)
   - Update job progress
   - Notify extension popup (if open)

7. **Complete**
   - Mark job as completed or failed
   - Update `lastSyncTime` and `nextSyncTime`
   - Store in job history (last 50 jobs kept)

---

## Error Handling

### Chat-Level Errors

If a single chat fails:
- Marked as `failed` in progress
- Error message recorded
- Sync continues with remaining chats
- Failed chats don't retry automatically

### Webhook Errors

If webhook request fails:
- **No retry logic for batch sync**
- Move to next batch after 2s delay
- Chat marked as failed if too many errors

### Complete Failure

If sync job fails entirely:
- Job marked as `failed`
- Error message recorded
- Next automatic sync will try again

---