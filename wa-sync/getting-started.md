---
layout: default
title: Getting Started
nav_order: 5
---

# Getting Started with WA-Sync
{: .no_toc }

This guide will walk you through setting up WA-Sync and building your first webhook integration.

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

Before you begin, ensure you have:

- ✅ Chrome Browser (version 90 or higher)
- ✅ Active WhatsApp account with WhatsApp Web access
- ✅ A server or hosting environment for your webhook
- ✅ Basic knowledge of REST APIs and JSON
- ✅ HTTPS endpoint (recommended for production)

---

## Step 1: Install WA-Sync Extension

### Option A: Chrome Web Store (Recommended)

1. Visit the [WA-Sync Chrome Web Store page](#)
2. Click "Add to Chrome"
3. Confirm the installation
4. Pin the extension to your toolbar for easy access

### Option B: Load Unpacked (Development)

1. Download the extension source code
2. Open Chrome and navigate to `chrome://extensions/`
3. Enable "Developer mode" (top right)
4. Click "Load unpacked"
5. Select the extension folder
6. Pin the extension to your toolbar

---

## Step 2: Set Up WhatsApp Web

1. Open [WhatsApp Web](https://web.whatsapp.com) in Chrome
2. Scan the QR code with your phone to log in
3. Wait for all chats to load completely
4. You should see the WA-Sync extension icon turn active

---

## Step 3: Create Your Webhook Endpoint

Your webhook endpoint will receive POST requests with JSON payloads. Here are examples in different languages:

### Node.js + Express

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/webhook', (req, res) => {
  const { event, timestamp, instanceId, data } = req.body;
  
  console.log(`Received event: ${event}`);
  console.log('Payload:', JSON.stringify(data, null, 2));
  
  // Process the event
  switch(event) {
    case 'message':
      handleMessage(data.message);
      break;
    case 'message_ack':
      handleAck(data.ackInfo);
      break;
    case 'reaction':
      handleReaction(data.reaction);
      break;
    // ... handle other events
  }
  
  // Always return 200 OK quickly
  res.status(200).json({ success: true });
});

app.post('/webhook/chat', (req, res) => {
  const { event, chat, batch, messages } = req.body;
  
  console.log(`Batch ${batch.number}/${batch.total} for chat ${chat.name}`);
  console.log(`Processing ${batch.messageCount} messages`);
  
  // Store messages in database
  messages.forEach(msg => {
    saveMessage(msg);
  });
  
  res.status(200).json({ success: true });
});

function handleMessage(message) {
  console.log(`New message from ${message.fromChat.phoneNumber}: ${message.body}`);
  // Store in database, trigger workflows, etc.
}

function extractPhone(chatObj) {
  if (!chatObj) return null;
  
  // Handle different ID formats
  const chatId = chatObj.id || '';
  const phoneNumber = chatObj.phoneNumber;
  
  // If ID is @lid format, use phoneNumber field
  if (chatId.includes('@lid')) {
    return extractPhoneValue(phoneNumber);
  }
  
  // If ID is @c.us format, it IS the phone number
  if (chatId.includes('@c.us')) {
    return chatId.replace('@c.us', '');
  }
  
  // Fallback to phoneNumber field
  return extractPhoneValue(phoneNumber);
}

function extractPhoneValue(phone) {
  if (typeof phone === 'string') {
    return phone.replace('@c.us', '').replace('@lid', '');
  }
  if (phone && phone._serialized) {
    return phone._serialized.replace('@c.us', '').replace('@lid', '');
  }
  return null;
}

function handleAck(ackInfo) {
  console.log(`Message ${ackInfo.messageId} ack changed to ${ackInfo.ack}`);
  // Update message status in database
}

function handleReaction(reaction) {
  console.log(`Reaction ${reaction.reactionText} on message ${reaction.parentMsgKey._serialized}`);
  // Update message reactions in database
}

app.listen(3000, () => {
  console.log('Webhook server running on port 3000');
});
```

### Python + Flask

```python
from flask import Flask, request, jsonify
import json

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    event = data.get('event')
    timestamp = data.get('timestamp')
    instance_id = data.get('instanceId')
    event_data = data.get('data')
    
    print(f"Received event: {event}")
    print(f"Payload: {json.dumps(event_data, indent=2)}")
    
    # Process the event
    if event == 'message':
        handle_message(event_data.get('message'))
    elif event == 'message_ack':
        handle_ack(event_data.get('ackInfo'))
    elif event == 'reaction':
        handle_reaction(event_data.get('reaction'))
    
    # Always return 200 OK quickly
    return jsonify({'success': True}), 200

@app.route('/webhook/chat', methods=['POST'])
def chat_sync():
    data = request.json
    chat = data.get('chat')
    batch = data.get('batch')
    messages = data.get('messages')
    
    print(f"Batch {batch['number']}/{batch['total']} for chat {chat['name']}")
    print(f"Processing {batch['messageCount']} messages")
    
    # Store messages in database
    for msg in messages:
        save_message(msg)
    
    return jsonify({'success': True}), 200

def handle_message(message):
    phone = extract_phone(message['fromChat'])
    body = message['body']
    print(f"New message from {phone}: {body}")
    # Store in database, trigger workflows, etc.

def extract_phone(chat_obj):
    """Extract phone number from chat object handling @lid and @c.us formats"""
    if not chat_obj:
        return None
    
    chat_id = chat_obj.get('id', '')
    phone_number = chat_obj.get('phoneNumber')
    
    # If ID is @lid format, use phoneNumber field
    if '@lid' in chat_id:
        return extract_phone_value(phone_number)
    
    # If ID is @c.us format, it IS the phone number
    if '@c.us' in chat_id:
        return chat_id.replace('@c.us', '')
    
    # Fallback to phoneNumber field
    return extract_phone_value(phone_number)

def extract_phone_value(phone):
    """Extract phone value from string or object"""
    if isinstance(phone, str):
        return phone.replace('@c.us', '').replace('@lid', '')
    if isinstance(phone, dict) and '_serialized' in phone:
        return phone['_serialized'].replace('@c.us', '').replace('@lid', '')
    return None

def handle_ack(ack_info):
    msg_id = ack_info['messageId']
    ack = ack_info['ack']
    print(f"Message {msg_id} ack changed to {ack}")
    # Update message status in database

def handle_reaction(reaction):
    emoji = reaction['reactionText']
    msg_id = reaction['parentMsgKey']['_serialized']
    print(f"Reaction {emoji} on message {msg_id}")
    # Update message reactions in database

def save_message(message):
    # Implement your database logic here
    pass

if __name__ == '__main__':
    app.run(port=3000, debug=True)
```

### PHP

```php
<?php
// webhook.php

header('Content-Type: application/json');

// Get POST data
$input = file_get_contents('php://input');
$data = json_decode($input, true);

$event = $data['event'] ?? '';
$timestamp = $data['timestamp'] ?? 0;
$instanceId = $data['instanceId'] ?? '';
$eventData = $data['data'] ?? [];

error_log("Received event: $event");

// Process the event
switch($event) {
    case 'message':
        handleMessage($eventData['message']);
        break;
    case 'message_ack':
        handleAck($eventData['ackInfo']);
        break;
    case 'reaction':
        handleReaction($eventData['reaction']);
        break;
    case 'chat_sync':
        handleChatSync($data);
        break;
}

// Always return 200 OK quickly
http_response_code(200);
echo json_encode(['success' => true]);

function handleMessage($message) {
    $phone = extractPhone($message['fromChat']);
    $body = $message['body'];
    error_log("New message from $phone: $body");
    // Store in database, trigger workflows, etc.
}

function extractPhone($chatObj) {
    if (!$chatObj) return null;
    
    $chatId = $chatObj['id'] ?? '';
    $phoneNumber = $chatObj['phoneNumber'] ?? null;
    
    // If ID is @lid format, use phoneNumber field
    if (strpos($chatId, '@lid') !== false) {
        return extractPhoneValue($phoneNumber);
    }
    
    // If ID is @c.us format, it IS the phone number
    if (strpos($chatId, '@c.us') !== false) {
        return str_replace('@c.us', '', $chatId);
    }
    
    // Fallback to phoneNumber field
    return extractPhoneValue($phoneNumber);
}

function extractPhoneValue($phone) {
    if (is_string($phone)) {
        return str_replace(['@c.us', '@lid'], '', $phone);
    }
    if (is_array($phone) && isset($phone['_serialized'])) {
        return str_replace(['@c.us', '@lid'], '', $phone['_serialized']);
    }
    return null;
}

function handleAck($ackInfo) {
    $msgId = $ackInfo['messageId'];
    $ack = $ackInfo['ack'];
    error_log("Message $msgId ack changed to $ack");
    // Update message status in database
}

function handleReaction($reaction) {
    $emoji = $reaction['reactionText'];
    $msgId = $reaction['parentMsgKey']['_serialized'];
    error_log("Reaction $emoji on message $msgId");
    // Update message reactions in database
}

function handleChatSync($data) {
    $chat = $data['chat'];
    $batch = $data['batch'];
    $messages = $data['messages'];
    
    error_log("Batch {$batch['number']}/{$batch['total']} for chat {$chat['name']}");
    
    foreach($messages as $msg) {
        saveMessage($msg);
    }
}

function saveMessage($message) {
    // Implement your database logic here
}
?>
```

---

## Step 4: Configure WA-Sync

1. Click the WA-Sync extension icon in Chrome
2. Click on "Settings" or the gear icon
3. Configure the following:

### Basic Configuration

| Setting | Value | Description |
|---------|-------|-------------|
| **Webhook URL** | `https://your-domain.com/webhook` | Your endpoint URL |
| **Enable Webhook Sync** | ✅ Checked | Enables real-time event sending |
| **Instance ID** | Auto-generated | Unique identifier (read-only) |

### Custom Headers (Optional)

Add authentication headers if needed:

```
Authorization: Bearer your-secret-token
X-API-Key: your-api-key
```

### Event Filtering

Select which events to capture:
- ✅ Messages
- ✅ Message Acknowledgments
- ✅ Reactions
- ✅ Message Edits/Deletes
- ⬜ Battery Changes (optional)
- ⬜ App State Changes (optional)

### Message Type Filtering

Choose which message types to sync:
- ✅ Text (chat)
- ✅ Images
- ✅ Videos
- ✅ Audio/Voice Notes
- ✅ Documents
- ✅ Locations
- ✅ Contacts

### Advanced Settings

| Setting | Recommended Value | Description |
|---------|------------------|-------------|
| **Retry Attempts** | 3 | Number of retry attempts for failed webhooks |
| **Retry Delay** | 1000ms | Delay between retry attempts |
| **Timeout** | 30000ms | Request timeout |

---

## Step 5: Test Your Integration

### Using the Test Webhook Button

1. In WA-Sync settings, click "Test Webhook"
2. Check your server logs for the test payload:

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

3. Verify your endpoint returns `200 OK`

### Send a Test Message

1. Send a message to yourself or a friend on WhatsApp Web
2. Check your server receives the message event
3. Verify all fields are present and correct

---

## Step 6: Enable Batch Sync (Optional)

For periodic full chat history synchronization:

1. Go to WA-Sync settings → Auto-Sync tab
2. Enable "Auto Sync"
3. Configure:
   - **Interval**: Daily (recommended to start)
   - **Messages Per Batch**: 50 (default)
   - **Max Parallel Chats**: 5 (default)
   - **Sync Endpoint**: `/chat` (default)

4. Click "Save Configuration"
5. Optionally, click "Start Sync Now" to test

---

## Database Schema Suggestions

### Messages Table

```sql
CREATE TABLE messages (
    id VARCHAR(255) PRIMARY KEY,
    chat_id VARCHAR(100) NOT NULL,
    chat_id_type VARCHAR(10),         -- 'lid', 'c.us', 'g.us', 'newsletter'
    from_phone VARCHAR(50),
    to_phone VARCHAR(50),
    body TEXT,
    type VARCHAR(50),
    timestamp BIGINT,
    from_me BOOLEAN,
    has_media BOOLEAN,
    ack INT DEFAULT 0,
    is_forwarded BOOLEAN,
    is_starred BOOLEAN,
    notify_name VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_chat_id (chat_id),
    INDEX idx_chat_id_type (chat_id_type),
    INDEX idx_timestamp (timestamp),
    INDEX idx_from_phone (from_phone)
);
```

**Note:** Store `chat_id_type` to know whether to look for phone in ID or phoneNumber field.

### Reactions Table

```sql
CREATE TABLE reactions (
    id VARCHAR(255) PRIMARY KEY,
    message_id VARCHAR(255) NOT NULL,
    sender_jid VARCHAR(100),
    reaction_text VARCHAR(10),
    timestamp BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (message_id) REFERENCES messages(id),
    INDEX idx_message_id (message_id)
);
```

### Chats Table

```sql
CREATE TABLE chats (
    id VARCHAR(100) PRIMARY KEY,
    name VARCHAR(255),
    user_id VARCHAR(50),
    is_group BOOLEAN,
    unread_count INT DEFAULT 0,
    archived BOOLEAN DEFAULT FALSE,
    last_message_time BIGINT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Best Practices

### Performance

1. **Return 200 OK Immediately**
   - Process events asynchronously
   - Use message queues (RabbitMQ, Redis, SQS)
   - Don't block the webhook response

2. **Implement Idempotency**
   - Use message IDs to prevent duplicates
   - Check if message exists before inserting

3. **Handle Rate Limits**
   - Implement backpressure mechanisms
   - Queue processing during high volume

### Security

1. **Use HTTPS**
   - Always use HTTPS for production webhooks
   - Validate SSL certificates

2. **Implement Authentication**
   - Use custom headers for API keys
   - Validate webhook origin

3. **Sanitize Input**
   - Validate JSON structure
   - Escape user content before display

### Error Handling

1. **Log Everything**
   - Log all incoming webhooks
   - Track processing errors
   - Monitor retry patterns

2. **Graceful Degradation**
   - Handle missing fields gracefully
   - Provide default values
   - Don't crash on unexpected data

3. **Monitor Health**
   - Track webhook success rate
   - Set up alerts for failures
   - Monitor processing latency

---

## Common Use Cases

### 1. Customer Support Dashboard

```javascript
async function handleMessage(message) {
  // Check if it's a customer message
  if (!message.fromMe) {
    await db.customers.upsert({
      phone: message.fromChat.phoneNumber,
      lastMessage: message.body,
      lastMessageTime: message.timestamp,
      unreadCount: db.raw('unread_count + 1')
    });
    
    // Notify support team
    await notifySupport({
      customer: message.notifyName,
      message: message.body,
      chatId: message.fromChat.id
    });
  }
}
```

### 2. Auto-Responder

```javascript
async function handleMessage(message) {
  // Only respond to incoming messages
  if (message.fromMe) return;
  
  const body = message.body.toLowerCase();
  
  if (body.includes('hours') || body.includes('timing')) {
    await sendMessage(message.fromChat.id, 
      'We are open Monday-Friday, 9 AM to 6 PM.'
    );
  } else if (body.includes('price') || body.includes('cost')) {
    await sendMessage(message.fromChat.id,
      'Please visit our website for pricing: https://example.com/pricing'
    );
  }
}
```

### 3. Message Archive & Search

```javascript
async function handleMessage(message) {
  // Store with full-text search capability
  await db.messages.insert({
    id: message.id._serialized,
    chatId: message.fromChat.id,
    fromPhone: message.fromChat.phoneNumber,
    body: message.body,
    timestamp: message.timestamp,
    searchableText: message.body.toLowerCase()
  });
}

// Later: Search functionality
async function searchMessages(query) {
  return await db.messages.where(
    'searchableText', 'like', `%${query.toLowerCase()}%`
  ).get();
}
```

---

## Troubleshooting

### Webhook Not Receiving Events

1. **Check Extension Status**
   - Is the extension enabled?
   - Is WhatsApp Web loaded?
   - Check extension console for errors

2. **Verify URL**
   - Is the webhook URL correct?
   - Is it accessible from the internet?
   - Try the "Test Webhook" button

3. **Check Firewall/CORS**
   - Is your server blocking requests?
   - Check server logs for incoming requests

### Events Being Dropped

1. **Check Retry Settings**
   - Increase retry attempts
   - Increase timeout value

2. **Server Response Time**
   - Respond with 200 OK within 30 seconds
   - Process asynchronously if needed

3. **Check Event Filters**
   - Are the right event types enabled?
   - Are message type filters correct?

### Duplicate Events

1. **Implement Idempotency**
   - Use message ID as primary key
   - Check existence before insert
   - Use `INSERT IGNORE` or `ON CONFLICT`

---

## Next Steps

- 📖 Explore [Real-Time Events](real-time-events.md) for detailed event structures
- 🔄 Learn about [Batch Sync](batch-sync.md) for historical data
- 💡 Check out [Examples](examples.md) for more integration patterns
