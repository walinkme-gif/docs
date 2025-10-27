---
layout: default
title: Examples
nav_order: 6
---

# WA-Sync Integration Examples
{: .no_toc }

Real-world examples and integration patterns for WA-Sync.

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Complete Webhook Server Examples

### Node.js + Express + MongoDB

```javascript
const express = require('express');
const mongoose = require('mongoose');
const app = express();

app.use(express.json());

// MongoDB Models
const MessageSchema = new mongoose.Schema({
  messageId: { type: String, unique: true, required: true },
  chatId: String,
  fromPhone: String,
  toPhone: String,
  body: String,
  type: String,
  timestamp: Number,
  fromMe: Boolean,
  hasMedia: Boolean,
  ack: { type: Number, default: 0 },
  notifyName: String,
  mediaInfo: Object,
  quotedMsg: Object,
  createdAt: { type: Date, default: Date.now }
});

const ReactionSchema = new mongoose.Schema({
  reactionId: String,
  messageId: String,
  senderJid: String,
  reactionText: String,
  timestamp: Number,
  createdAt: { type: Date, default: Date.now }
});

const Message = mongoose.model('Message', MessageSchema);
const Reaction = mongoose.model('Reaction', ReactionSchema);

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/wa-sync', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Real-time events endpoint
app.post('/webhook', async (req, res) => {
  const { event, timestamp, instanceId, data } = req.body;
  
  try {
    switch(event) {
      case 'message':
        await handleMessage(data.message);
        break;
      case 'message_ack':
        await handleAck(data.ackInfo);
        break;
      case 'message_edit':
        await handleEdit(data.editInfo);
        break;
      case 'message_revoke':
        await handleRevoke(data.revokeInfo);
        break;
      case 'reaction':
        await handleReaction(data.reaction);
        break;
      case 'poll_vote':
        await handlePollVote(data.pollVote);
        break;
      case 'app_state_change':
        console.log(`WhatsApp state: ${data.state}`);
        break;
      case 'incoming_call':
        await handleCall(data);
        break;
    }
    
    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Error processing webhook:', error);
    res.status(500).json({ error: error.message });
  }
});

// Batch sync endpoint
app.post('/webhook/chat', async (req, res) => {
  const { chat, batch, messages } = req.body;
  
  try {
    console.log(`Processing batch ${batch.number}/${batch.total} for ${chat.name}`);
    
    for (const message of messages) {
      await Message.findOneAndUpdate(
        { messageId: message.id._serialized || message.id },
        {
          messageId: message.id._serialized || message.id,
          chatId: message.from,
          fromPhone: extractPhone(message.fromChat),
          toPhone: extractPhone(message.toChat),
          body: message.body,
          type: message.type,
          timestamp: message.timestamp,
          fromMe: message.fromMe,
          hasMedia: message.hasMedia,
          ack: message.ack,
          notifyName: message.notifyName,
          mediaInfo: extractMediaInfo(message),
          quotedMsg: message.quotedMsg
        },
        { upsert: true, new: true }
      );
    }
    
    res.status(200).json({ 
      success: true, 
      processed: messages.length 
    });
  } catch (error) {
    console.error('Error processing batch:', error);
    res.status(500).json({ error: error.message });
  }
});

// Event Handlers
async function handleMessage(message) {
  const messageId = message.id._serialized || message.id;
  
  await Message.findOneAndUpdate(
    { messageId },
    {
      messageId,
      chatId: message.from,
      fromPhone: extractPhone(message.fromChat),
      toPhone: extractPhone(message.toChat),
      body: message.body,
      type: message.type,
      timestamp: message.timestamp,
      fromMe: message.fromMe,
      hasMedia: message.hasMedia,
      ack: message.ack,
      notifyName: message.notifyName,
      mediaInfo: extractMediaInfo(message),
      quotedMsg: message.quotedMsg
    },
    { upsert: true, new: true }
  );
  
  console.log(`Saved message ${messageId}`);
}

async function handleAck(ackInfo) {
  await Message.updateOne(
    { messageId: ackInfo.messageId },
    { $set: { ack: ackInfo.ack } }
  );
  
  console.log(`Updated ack for ${ackInfo.messageId} to ${ackInfo.ack}`);
}

async function handleEdit(editInfo) {
  await Message.updateOne(
    { messageId: editInfo.messageId },
    { 
      $set: { 
        body: editInfo.newBody,
        edited: true,
        previousBody: editInfo.prevBody
      } 
    }
  );
  
  console.log(`Message ${editInfo.messageId} edited`);
}

async function handleRevoke(revokeInfo) {
  await Message.updateOne(
    { messageId: revokeInfo.messageId },
    { 
      $set: { 
        revoked: true,
        revokedBy: revokeInfo.revokedBy,
        revokeType: revokeInfo.revokeType
      } 
    }
  );
  
  console.log(`Message ${revokeInfo.messageId} revoked`);
}

async function handleReaction(reaction) {
  // Handle array of reactions
  const reactions = Array.isArray(reaction) ? reaction : [reaction];
  
  for (const r of reactions) {
    const messageId = r.parentMsgKey._serialized || r.parentMsgKey;
    
    if (r.reactionText === '') {
      // Remove reaction
      await Reaction.deleteOne({
        messageId,
        senderJid: r.senderUserJid
      });
    } else {
      // Add/update reaction
      await Reaction.findOneAndUpdate(
        {
          messageId,
          senderJid: r.senderUserJid
        },
        {
          reactionId: r.id,
          messageId,
          senderJid: r.senderUserJid,
          reactionText: r.reactionText,
          timestamp: r.timestamp
        },
        { upsert: true, new: true }
      );
    }
  }
}

async function handlePollVote(pollVote) {
  console.log(`Poll vote from ${pollVote.senderUserJid}`);
  console.log(`Selected options: ${pollVote.selectedOptions.join(', ')}`);
  // Store poll votes in database
}

async function handleCall(callData) {
  console.log(`Incoming ${callData.isVideo ? 'video' : 'voice'} call from ${callData.peerJid}`);
  // Log calls, send notifications, etc.
}

// Helper Functions
function extractPhone(chatObj) {
  if (!chatObj) return null;
  
  const chatId = chatObj.id || '';
  const phoneNumber = chatObj.phoneNumber;
  
  // If ID is @lid format, use phoneNumber field which will be @c.us
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

function extractMediaInfo(message) {
  if (!message.hasMedia) return null;
  
  return {
    mediaKey: message.mediaKey,
    mimetype: message.mimetype,
    filename: message.filename,
    size: message.size,
    width: message.width,
    height: message.height,
    duration: message.duration,
    caption: message.caption
  };
}

// API Endpoints for querying data
app.get('/api/messages/:chatId', async (req, res) => {
  const { chatId } = req.params;
  const { limit = 50, offset = 0 } = req.query;
  
  const messages = await Message.find({ chatId })
    .sort({ timestamp: -1 })
    .limit(parseInt(limit))
    .skip(parseInt(offset));
  
  res.json(messages);
});

app.get('/api/chats', async (req, res) => {
  const chats = await Message.aggregate([
    {
      $group: {
        _id: '$chatId',
        lastMessage: { $last: '$body' },
        lastMessageTime: { $max: '$timestamp' },
        messageCount: { $sum: 1 },
        unreadCount: {
          $sum: { $cond: [{ $and: [{ $eq: ['$fromMe', false] }, { $lt: ['$ack', 4] }] }, 1, 0] }
        }
      }
    },
    { $sort: { lastMessageTime: -1 } }
  ]);
  
  res.json(chats);
});

app.listen(3000, () => {
  console.log('WA-Sync webhook server running on port 3000');
});
```

---

## Python + FastAPI + PostgreSQL

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional, List, Dict, Any
import asyncpg
from datetime import datetime
import json

app = FastAPI()

# Database connection pool
db_pool = None

# Pydantic Models
class WebhookPayload(BaseModel):
    event: str
    timestamp: int
    instanceId: str
    data: Dict[str, Any]

class ChatSyncPayload(BaseModel):
    event: str
    timestamp: int
    instanceId: str
    chat: Dict[str, Any]
    batch: Dict[str, Any]
    messages: List[Dict[str, Any]]

@app.on_event("startup")
async def startup():
    global db_pool
    db_pool = await asyncpg.create_pool(
        'postgresql://user:password@localhost/wa_sync',
        min_size=10,
        max_size=20
    )
    
    # Create tables
    async with db_pool.acquire() as conn:
        await conn.execute('''
            CREATE TABLE IF NOT EXISTS messages (
                message_id VARCHAR(255) PRIMARY KEY,
                chat_id VARCHAR(100),
                from_phone VARCHAR(50),
                to_phone VARCHAR(50),
                body TEXT,
                type VARCHAR(50),
                timestamp BIGINT,
                from_me BOOLEAN,
                has_media BOOLEAN,
                ack INTEGER DEFAULT 0,
                notify_name VARCHAR(255),
                media_info JSONB,
                quoted_msg JSONB,
                revoked BOOLEAN DEFAULT FALSE,
                edited BOOLEAN DEFAULT FALSE,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        
        await conn.execute('''
            CREATE TABLE IF NOT EXISTS reactions (
                id SERIAL PRIMARY KEY,
                message_id VARCHAR(255),
                sender_jid VARCHAR(100),
                reaction_text VARCHAR(10),
                timestamp BIGINT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                UNIQUE(message_id, sender_jid)
            )
        ''')
        
        await conn.execute('''
            CREATE INDEX IF NOT EXISTS idx_messages_chat_id ON messages(chat_id);
            CREATE INDEX IF NOT EXISTS idx_messages_timestamp ON messages(timestamp);
            CREATE INDEX IF NOT EXISTS idx_reactions_message_id ON reactions(message_id);
        ''')

@app.on_event("shutdown")
async def shutdown():
    await db_pool.close()

@app.post('/webhook')
async def webhook(payload: WebhookPayload):
    event = payload.event
    data = payload.data
    
    try:
        if event == 'message':
            await handle_message(data.get('message'))
        elif event == 'message_ack':
            await handle_ack(data.get('ackInfo'))
        elif event == 'message_edit':
            await handle_edit(data.get('editInfo'))
        elif event == 'message_revoke':
            await handle_revoke(data.get('revokeInfo'))
        elif event == 'reaction':
            await handle_reaction(data.get('reaction'))
        elif event == 'poll_vote':
            await handle_poll_vote(data.get('pollVote'))
        elif event == 'app_state_change':
            print(f"WhatsApp state: {data.get('state')}")
        elif event == 'incoming_call':
            await handle_call(data)
        
        return {'success': True}
    except Exception as e:
        print(f"Error processing webhook: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.post('/webhook/chat')
async def chat_sync(payload: ChatSyncPayload):
    chat = payload.chat
    batch = payload.batch
    messages = payload.messages
    
    try:
        print(f"Processing batch {batch['number']}/{batch['total']} for {chat['name']}")
        
        async with db_pool.acquire() as conn:
            for message in messages:
                await save_message(conn, message)
        
        return {'success': True, 'processed': len(messages)}
    except Exception as e:
        print(f"Error processing batch: {e}")
        raise HTTPException(status_code=500, detail=str(e))

# Event Handlers
async def handle_message(message):
    async with db_pool.acquire() as conn:
        await save_message(conn, message)

async def save_message(conn, message):
    message_id = extract_message_id(message.get('id'))
    
    await conn.execute('''
        INSERT INTO messages (
            message_id, chat_id, from_phone, to_phone, body, type,
            timestamp, from_me, has_media, ack, notify_name,
            media_info, quoted_msg
        ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13)
        ON CONFLICT (message_id) DO UPDATE SET
            ack = EXCLUDED.ack,
            body = EXCLUDED.body
    ''', 
        message_id,
        message.get('from'),
        extract_phone(message.get('fromChat')),
        extract_phone(message.get('toChat')),
        message.get('body'),
        message.get('type'),
        message.get('timestamp'),
        message.get('fromMe'),
        message.get('hasMedia'),
        message.get('ack', 0),
        message.get('notifyName'),
        json.dumps(extract_media_info(message)),
        json.dumps(message.get('quotedMsg'))
    )
    
    print(f"Saved message {message_id}")

async def handle_ack(ack_info):
    async with db_pool.acquire() as conn:
        await conn.execute(
            'UPDATE messages SET ack = $1 WHERE message_id = $2',
            ack_info['ack'],
            ack_info['messageId']
        )
    
    print(f"Updated ack for {ack_info['messageId']} to {ack_info['ack']}")

async def handle_edit(edit_info):
    async with db_pool.acquire() as conn:
        await conn.execute(
            'UPDATE messages SET body = $1, edited = TRUE WHERE message_id = $2',
            edit_info['newBody'],
            edit_info['messageId']
        )
    
    print(f"Message {edit_info['messageId']} edited")

async def handle_revoke(revoke_info):
    async with db_pool.acquire() as conn:
        await conn.execute(
            'UPDATE messages SET revoked = TRUE WHERE message_id = $1',
            revoke_info['messageId']
        )
    
    print(f"Message {revoke_info['messageId']} revoked")

async def handle_reaction(reaction):
    reactions = reaction if isinstance(reaction, list) else [reaction]
    
    async with db_pool.acquire() as conn:
        for r in reactions:
            message_id = extract_message_id(r.get('parentMsgKey'))
            
            if r.get('reactionText') == '':
                # Remove reaction
                await conn.execute(
                    'DELETE FROM reactions WHERE message_id = $1 AND sender_jid = $2',
                    message_id,
                    r.get('senderUserJid')
                )
            else:
                # Add/update reaction
                await conn.execute('''
                    INSERT INTO reactions (message_id, sender_jid, reaction_text, timestamp)
                    VALUES ($1, $2, $3, $4)
                    ON CONFLICT (message_id, sender_jid) DO UPDATE
                    SET reaction_text = EXCLUDED.reaction_text,
                        timestamp = EXCLUDED.timestamp
                ''',
                    message_id,
                    r.get('senderUserJid'),
                    r.get('reactionText'),
                    r.get('timestamp')
                )

async def handle_poll_vote(poll_vote):
    print(f"Poll vote from {poll_vote.get('senderUserJid')}")
    print(f"Selected options: {poll_vote.get('selectedOptions')}")

async def handle_call(call_data):
    call_type = 'video' if call_data.get('isVideo') else 'voice'
    print(f"Incoming {call_type} call from {call_data.get('peerJid')}")

# Helper Functions
def extract_message_id(id_obj):
    if isinstance(id_obj, str):
        return id_obj
    if isinstance(id_obj, dict):
        return id_obj.get('_serialized') or id_obj.get('id')
    return None

def extract_phone(chat_obj):
    """Extract phone number handling @lid and @c.us formats"""
    if not chat_obj:
        return None
    
    chat_id = chat_obj.get('id', '')
    phone_number = chat_obj.get('phoneNumber')
    
    # If ID is @lid format, use phoneNumber field which will be @c.us
    if '@lid' in chat_id:
        return extract_phone_value(phone_number)
    
    # If ID is @c.us format, it IS the phone number
    if '@c.us' in chat_id:
        return chat_id.replace('@c.us', '')
    
    # Fallback to phoneNumber field
    return extract_phone_value(phone_number)

def extract_phone_value(phone):
    if isinstance(phone, str):
        return phone.replace('@c.us', '').replace('@lid', '')
    if isinstance(phone, dict):
        return phone.get('_serialized', '').replace('@c.us', '').replace('@lid', '')
    return None

def extract_media_info(message):
    if not message.get('hasMedia'):
        return None
    
    return {
        'mediaKey': message.get('mediaKey'),
        'mimetype': message.get('mimetype'),
        'filename': message.get('filename'),
        'size': message.get('size'),
        'width': message.get('width'),
        'height': message.get('height'),
        'duration': message.get('duration'),
        'caption': message.get('caption')
    }

# Query API
@app.get('/api/messages/{chat_id}')
async def get_messages(chat_id: str, limit: int = 50, offset: int = 0):
    async with db_pool.acquire() as conn:
        rows = await conn.fetch(
            '''SELECT * FROM messages 
               WHERE chat_id = $1 
               ORDER BY timestamp DESC 
               LIMIT $2 OFFSET $3''',
            chat_id, limit, offset
        )
        
        return [dict(row) for row in rows]

@app.get('/api/chats')
async def get_chats():
    async with db_pool.acquire() as conn:
        rows = await conn.fetch('''
            SELECT 
                chat_id,
                MAX(timestamp) as last_message_time,
                COUNT(*) as message_count,
                SUM(CASE WHEN NOT from_me AND ack < 4 THEN 1 ELSE 0 END) as unread_count
            FROM messages
            GROUP BY chat_id
            ORDER BY last_message_time DESC
        ''')
        
        return [dict(row) for row in rows]

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='0.0.0.0', port=3000)
```

---

## Simple Testing Server

For quick testing without a database:

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/webhook', (req, res) => {
  console.log('\n=== NEW EVENT ===');
  console.log('Event:', req.body.event);
  console.log('Timestamp:', new Date(req.body.timestamp).toISOString());
  console.log('Data:', JSON.stringify(req.body.data, null, 2));
  console.log('================\n');
  
  res.status(200).json({ success: true });
});

app.post('/webhook/chat', (req, res) => {
  const { chat, batch, messages } = req.body;
  
  console.log('\n=== BATCH SYNC ===');
  console.log(`Chat: ${chat.name} (${chat.id})`);
  console.log(`Batch: ${batch.number}/${batch.total}`);
  console.log(`Messages: ${batch.messageCount}`);
  console.log('==================\n');
  
  res.status(200).json({ success: true });
});

app.listen(3000, () => {
  console.log('Test webhook server running on http://localhost:3000');
});
```

---

## Docker Compose Setup

```yaml
version: '3.8'

services:
  webhook:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongo:27017/wa-sync
    depends_on:
      - mongo
    restart: unless-stopped

  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - webhook
    restart: unless-stopped

volumes:
  mongo-data:
```

---

## Integration Patterns

### Queue-Based Processing

```javascript
const Queue = require('bull');
const messageQueue = new Queue('messages', 'redis://localhost:6379');

app.post('/webhook', async (req, res) => {
  // Add to queue immediately
  await messageQueue.add(req.body);
  
  // Return 200 OK immediately
  res.status(200).json({ success: true });
});

// Process queue in background
messageQueue.process(async (job) => {
  const { event, data } = job.data;
  
  switch(event) {
    case 'message':
      await processMessage(data.message);
      break;
    // ... handle other events
  }
});
```

