# WA-Sync Webhook API - Real-Time Events

**Version:** 1.0.0  
**Last Updated:** October 27, 2025

This document details all real-time webhook events sent by WA-Sync.

---

## Event 1: Message Event

**Event Type:** `message`

**Trigger:** When a new message is received or sent in WhatsApp Web

### Basic Text Message

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
        "participant": null,
        "_serialized": "false_1234567890@c.us_3EB0ABC123DEF456"
      },
      "ack": 1,
      "body": "Hello, how are you?",
      "type": "chat",
      "timestamp": 1730000000000,
      "from": "1234567890@c.us",
      "to": "0987654321@c.us",
      "notifyName": "John Doe",
      "self": "in",
      "fromMe": false,
      "hasMedia": false,
      "isForwarded": false,
      "isStatus": false,
      "isStatusV3": false,
      "isStarred": false,
      "broadcast": false,
      "hasQuotedMsg": false,
      "isEphemeral": false,
      "isGif": false,
      "links": [],
      "mentionedIds": [],
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      },
      "toChat": {
        "id": "0987654321@c.us",
        "phoneNumber": "+0987654321@c.us"
      }
    }
  }
}
```

**Key Fields:**
- `body` - Message text content
- `type` - Always "chat" for text messages
- `fromChat.phoneNumber` - Sender's phone number (string or object with `_serialized`)
- `toChat.phoneNumber` - Recipient's phone number
- `notifyName` - Sender's display name in WhatsApp
- `links` - Array of extracted links with suspicious link detection

### Image Message

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "ack": 1,
      "body": "",
      "type": "image",
      "timestamp": 1730000000000,
      "from": "1234567890@c.us",
      "to": "0987654321@c.us",
      "fromMe": false,
      "hasMedia": true,
      "body" : "base64 recieved",
      "mediaKey": "ABC123DEF456GHI789...",
      "size": 524288,
      "width": 1920,
      "height": 1080,
      "mimetype": "image/jpeg",
      "caption": "Check out this photo!",
      "filename": "IMG-20251027-WA0001.jpg",
      "filehash": "HASH123...",
      "encFilehash": "ENCHASH456...",
      "directPath": "/v/t62.../IMG-20251027-WA0001.jpg",
      "mediaKeyTimestamp": 1730000000,
      "isGif": false,
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

**Important Notes About Images:**
- `body` is **base 64** for image messages
- `caption` contains the image caption (if any)
- **NO base64 image data is included**
- Use `mediaKey`, `directPath`, and `encFilehash` for media download
- `size` is in bytes
- `width` and `height` in pixels

### Video Message

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "type": "video",
      "hasMedia": true,
      "body": "",
      "caption": "Amazing video!",
      "mediaKey": "VID123...",
      "size": 5242880,
      "width": 1920,
      "height": 1080,
      "mimetype": "video/mp4",
      "filename": "VID-20251027-WA0001.mp4",
      "duration": 45,
      "isGif": false,
      "gifPlayback": false,
      "streamingSidecar": "BASE64_SIDECAR...",
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**Video-Specific Fields:**
- `duration` - Video length in seconds
- `isGif` - true if sent as GIF
- `gifPlayback` - true if should play as GIF
- `streamingSidecar` - Video streaming metadata

### Audio / Voice Note (PTT)

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "type": "ptt",
      "hasMedia": true,
      "body": "",
      "mediaKey": "AUD123...",
      "size": 102400,
      "mimetype": "audio/ogg; codecs=opus",
      "filename": "PTT-20251027-WA0001.opus",
      "duration": 15,
      "waveform": [0, 12, 34, 56, 78, ...],
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**Audio Fields:**
- `type` - "ptt" for voice notes, "audio" for audio files
- `duration` - Audio length in seconds
- `waveform` - Array of 64 numbers (0-100) for waveform visualization
- `mimetype` - Usually "audio/ogg; codecs=opus" for PTT

### Document Message

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "type": "document",
      "hasMedia": true,
      "body": "Important document",
      "caption": "Please review this",
      "mediaKey": "DOC123...",
      "size": 1048576,
      "mimetype": "application/pdf",
      "filename": "Report_Q4_2025.pdf",
      "pageCount": 15,
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**Document Fields:**
- `filename` - Original file name
- `pageCount` - Number of pages (for PDFs)
- `body` - Can contain text description

### Location Message

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "type": "location",
      "body": "San Francisco, CA",
      "hasMedia": false,
      "lat": 37.7749,
      "lng": -122.4194,
      "loc": "San Francisco, CA",
      "clientUrl": "https://maps.google.com/?q=37.7749,-122.4194",
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**Location Fields:**
- `lat` - Latitude (number)
- `lng` - Longitude (number)
- `loc` - Location description
- `clientUrl` - Google Maps URL
- `body` - Location name/description

### Contact Card (vCard)

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "type": "vcard",
      "body": "BEGIN:VCARD\nVERSION:3.0\nFN:Jane Smith\nTEL:+1234567890@c.us\nEND:VCARD",
      "vcardFormattedName": "Jane Smith",
      "hasMedia": false,
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**vCard Fields:**
- `body` - Full vCard data (parseable)
- `vcardFormattedName` - Contact name
- `type` - "vcard" for single contact, "multi_vcard" for multiple

### Group Message

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": {
        "fromMe": false,
        "remote": "123456789-1234567890@g.us",
        "id": "3EB0ABC123DEF456",
        "participant": "1234567890@c.us",
        "_serialized": "false_123456789-1234567890@g.us_3EB0ABC123DEF456"
      },
      "body": "Hello everyone!",
      "type": "chat",
      "from": "123456789-1234567890@g.us",
      "to": "0987654321@c.us",
      "author": "1234567890@c.us",
      "notifyName": "John Doe",
      "isGroupMsg": true,
      "mentionedIds": ["9876543210@c.us"],
      "fromChat": {
        "id": "123456789-1234567890@g.us",
        "phoneNumber": null
      }
    }
  }
}
```

**Group Message Fields:**
- `from` - Group ID (ends with `@g.us`)
- `author` - Actual sender's ID
- `participant` - Same as author
- `isGroupMsg` - true
- `mentionedIds` - Array of @mentioned user IDs
- `fromChat.phoneNumber` - null for groups

### Message with Quote/Reply

```json
{
  "event": "message",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "message": {
      "id": { /* same structure */ },
      "body": "Yes, I agree!",
      "type": "chat",
      "hasQuotedMsg": true,
      "quotedMsg": {
        "id": {
          "_serialized": "false_1234567890@c.us_QUOTED123"
        },
        "body": "Should we meet tomorrow?",
        "type": "chat",
        "from": "1234567890@c.us",
        "notifyName": "John Doe"
      },
      "quotedMsgId": "false_1234567890@c.us_QUOTED123",
      "fromChat": {
        "id": "1234567890@c.us",
        "phoneNumber": "+1234567890@c.us"
      }
    }
  }
}
```

**Quote/Reply Fields:**
- `hasQuotedMsg` - true if replying to a message
- `quotedMsg` - Full quoted message object
- `quotedMsgId` - ID of quoted message (for reference)

---

## Event 2: Message Acknowledgment (ACK)

**Event Type:** `message_ack`

**Trigger:** Message delivery status changes

```json
{
  "event": "message_ack",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "ackInfo": {
      "messageId": "true_1234567890@c.us_3EB0ABC123DEF456",
      "ack": 3
    }
  }
}
```

**ACK Values:**
- `0` - ACK_ERROR (failed)
- `1` - ACK_PENDING (sent to server)
- `2` - ACK_SERVER (delivered to server)
- `3` - ACK_DEVICE (delivered to recipient)
- `4` - ACK_READ (read by recipient - blue checkmarks)
- `5` - ACK_PLAYED (audio/video played)

---

## Event 3: Message Edit

**Event Type:** `message_edit`

**Trigger:** User edits a message (within 15-minute window)

```json
{
  "event": "message_edit",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "editInfo": {
      "messageId": "true_1234567890@c.us_3EB0ABC123DEF456",
      "newBody": "Hello, how are you doing today?",
      "prevBody": "Hello, how are you doing?"
    }
  }
}
```

---

## Event 4: Message Revoke (Delete)

**Event Type:** `message_revoke`

**Trigger:** User deletes a message

```json
{
  "event": "message_revoke",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "revokeInfo": {
      "messageId": "false_1234567890@c.us_3EB0ABC123DEF456",
      "revokedBy": "1234567890@c.us",
      "revokeType": "everyone"
    }
  }
}
```

**Revoke Types:**
- `me` - Deleted for self only
- `everyone` - Deleted for all participants

---

## Event 5: Reaction

**Event Type:** `reaction`

**Trigger:** User reacts to a message with an emoji

```json
{
  "event": "reaction",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "reaction": {
      "id": "123456789",
      "orphan": 0,
      "orphanReason": null,
      "msgKey": {
        "fromMe": false,
        "remote": "1234567890@c.us",
        "id": "3EB0ABC123DEF456",
        "_serialized": "false_1234567890@c.us_3EB0ABC123DEF456"
      },
      "parentMsgKey": {
        "fromMe": true,
        "remote": "1234567890@c.us",
        "id": "3EB0PARENT123",
        "_serialized": "true_1234567890@c.us_3EB0PARENT123"
      },
      "reactionText": "👍",
      "read": false,
      "senderUserJid": "1234567890@c.us",
      "timestamp": 1730000000000
    }
  }
}
```

**Reaction Fields:**
- `reactionText` - Emoji (empty string "" means reaction removed)
- `msgKey` - ID of the reaction itself
- `parentMsgKey` - ID of the message being reacted to
- `senderUserJid` - Who sent the reaction
- `orphan` - 0 = normal, non-zero = orphaned reaction

**Note:** Array of reactions can be sent in single event

---

## Event 6: Poll Vote

**Event Type:** `poll_vote`

**Trigger:** User votes on a poll

```json
{
  "event": "poll_vote",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "pollVote": {
      "id": "vote_123456",
      "msgKey": {
        "_serialized": "vote_1234567890@c.us_VOTE123"
      },
      "parentMsgKey": {
        "fromMe": true,
        "remote": "123456789@g.us",
        "id": "3EB0POLL123",
        "_serialized": "true_123456789@g.us_3EB0POLL123"
      },
      "selectedOptions": [0, 2],
      "senderUserJid": "9876543210@c.us",
      "sender": {
        "_serialized": "9876543210@c.us"
      },
      "timestamp": 1730000000000,
      "parentMessage": {
        "type": "poll_creation",
        "pollName": "What's your favorite color?",
        "pollOptions": [
          { "name": "Red" },
          { "name": "Blue" },
          { "name": "Green" }
        ]
      }
    }
  }
}
```

**Poll Vote Fields:**
- `selectedOptions` - Array of option indices (0-based)
- `parentMsgKey` - ID of the poll message
- `parentMessage` - The original poll (if available)
- `senderUserJid` - Voter's ID

---

## Event 7: App State Change

**Event Type:** `app_state_change`

**Trigger:** WhatsApp connection state changes

```json
{
  "event": "app_state_change",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "state": "CONNECTED"
  }
}
```

**Possible States:**
- `UNPAIRED` - Not logged in
- `UNPAIRED_IDLE` - Logged out
- `OPENING` - Connecting
- `PAIRING` - Scanning QR code
- `CONNECTED` - Fully connected
- `TIMEOUT` - Connection timeout
- `CONFLICT` - Session conflict

---

## Event 8: Battery Change

**Event Type:** `battery_change`

**Trigger:** Phone battery status changes

```json
{
  "event": "battery_change",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "battery": 75,
    "plugged": true,
    "powersave": false
  }
}
```

---

## Event 9: Incoming Call

**Event Type:** `incoming_call`

**Trigger:** WhatsApp call received

```json
{
  "event": "incoming_call",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "id": "call_ABC123DEF456",
    "peerJid": "1234567890@c.us",
    "offerTime": 1730000000,
    "isVideo": false,
    "isGroup": false,
    "canHandleLocally": true,
    "outgoing": false
  }
}
```

**Call Fields:**
- `isVideo` - true for video calls
- `isGroup` - true for group calls
- `peerJid` - Caller ID

---

## Event 10: Chat Archived

**Event Type:** `chat_archived`

**Trigger:** Chat archived/unarchived

```json
{
  "event": "chat_archived",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "archiveInfo": {
      "chatId": "1234567890@c.us",
      "currentState": true,
      "prevState": false
    }
  }
}
```

---

## Event 11: Chat Unread Count

**Event Type:** `chat_unread_count`

**Trigger:** Unread message count changes

```json
{
  "event": "chat_unread_count",
  "timestamp": 1730000000000,
  "instanceId": "wa-sync-1730000000000-abc123xyz",
  "data": {
    "unreadInfo": {
      "chatId": "1234567890@c.us",
      "unreadCount": 5
    }
  }
}
```

---

## All Other Events

Events 12-16 (`message_ciphertext`, `message_change`, `message_type_change`, `media_uploaded`, `chat_removed`) follow similar patterns with event-specific data structures.

---

**Next:** [Batch Sync Documentation](./03-Batch-Sync.md)
