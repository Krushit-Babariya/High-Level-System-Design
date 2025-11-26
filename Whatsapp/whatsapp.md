# WhatsApp

---

## Requirements
### **Function Requirements**
#### **Core Requirements**
- Users should be able to send and receive text messages in real-time with end-to-end encryption.
- Users should be able to make one-on-one voice and video calls with low latency and high quality.
- Users should be able to send and receive multimedia files (images, videos, audio, documents) up to 2GB in size.
- Users should be able to create and participate in group chats with up to 1024 participants.
- Users should be able to see the online/offline status and last seen timestamp of their contacts.
- Users should be able to see message delivery status with checkmarks (sent, delivered, read).
- The system should store and deliver messages to offline users when they come online.
- Users should be able to make group voice and video calls with multiple participants simultaneously.

**Below the Line**
- Users should be able to share their live location with contacts.
- Users should be able to post status updates (stories) that disappear after 24 hours.
- Users should be able to create communities to organize multiple groups under one umbrella.
- Users should be able to send disappearing messages that auto-delete after a set time.
- Users should be able to make payments through WhatsApp Pay (in select countries).
- Users should be able to backup their chat history to cloud storage.

### **Non-Function Requirements**
#### **Core Requirements**
- The system should ensure message delivery within 1 second under normal network conditions.
- The system should guarantee end-to-end encryption for all messages, ensuring only sender and receiver can read them.
- The system should maintain 99.99% uptime and availability across all regions.
- The system should handle 150 billion messages per day with peak loads of 1.7 million messages per second.
- The system should scale horizontally to accommodate billions of concurrent connections.
- The system should minimize bandwidth usage with efficient binary protocols (FunXMPP).

#### **Below the line**
- The system should comply with data privacy regulations (GDPR, CCPA) across different jurisdictions.
- The system should implement robust spam detection and abuse prevention mechanisms.
- The system should support seamless code deployment without disconnecting active users.
- The system should maintain data consistency across distributed data centers with eventual consistency model.

---

## Capacity Estimation
### 1. Traffic & User Assumptions
### User Base
- **Monthly Active Users (MAU):** 3.14 billion users (January 2025)
- **Daily Active Users (DAU):** 2.3 billion users
- **Active concurrent users during peak:** 700 million users (30% of DAU)
- **WhatsApp Business Users:** 200 million businesses
- **Total registered phone numbers:** 3.5+ billion

### Daily Operations
- **Daily Messages:** 150 billion messages/day
- **Daily Voice Calls:** 2 billion minutes/day
- **Daily Video Calls:** 2 billion minutes/day
- **Daily Voice Messages:** 7 billion voice notes/day
- **Status Updates:** 500 million daily active users for Status
- **Group Messages:** Approximately 40% of all messages

### Read/Write Operations
- **Read:Write Ratio:** 80:20 (messages are delivered once but read/status updates happen multiple times)
- **Media:Text Ratio:** 30:70 (30% messages contain media, 70% are text-only)

### **2. Requests Per Second (RPS) Calculation**
### Peak Message Traffic
```xml
Daily messages = 150 billion
Peak hour assumption = 12% of daily traffic (concentrated in evening hours)
Peak hour messages = 150B × 0.12 = 18 billion messages

Peak RPS = 18B messages / 3600 seconds = 5,000,000 messages/sec
With safety margin (1.5x) = ~7.5M RPS for message delivery

However, considering distributed global load:
Regional peak RPS = ~1.5M - 2M per major region
```

### Connection Management
**Persistent WebSocket Connections:**
- Active concurrent users = 700 million during peak
- Each user maintains 1 persistent connection
```
Connection management overhead = 700M connections
Connection state updates/sec = 700M / 60 = ~11.6M updates/min
```

### Status Update Frequency
**Online/Offline Status:**
- Active users = 2.3 billion
- Status updates when users open/close app
```
Average status changes per user per day = 10 times
Status updates/day = 2.3B × 10 = 23 billion status updates
Status updates/sec (average) = 23B / 86400 = ~266,000/sec
Peak status updates/sec = 266K × 3 = ~800,000/sec
```

### Typing Indicators
**Real-time typing notifications:**
```
Active conversations (simultaneous) = 100M conversations
Average typing indicator updates/conversation = 10/min
Typing indicators/sec = (100M × 10) / 60 = ~16.6M/sec
```

### Message Receipt Acknowledgments
**Delivery and Read Receipts:**
```
Each message generates:
- 1 sent acknowledgment
- 1 delivered acknowledgment  
- 1 read acknowledgment (per recipient)

For 1:1 messages: 3 acknowledgments per message
For group messages (avg 10 members): 21 acknowledgments per message

Average acknowledgments/message = 6 (weighted average)
Daily acknowledgments = 150B × 6 = 900 billion
Acknowledgments/sec (average) = 900B / 86400 = ~10.4M/sec
Peak acknowledgments/sec = ~31M/sec
```

### Voice and Video Call Signaling
**Call Setup and Management:**
```
Daily voice call minutes = 2 billion minutes
Daily video call minutes = 2 billion minutes
Total daily call minutes = 4 billion minutes

Average call duration = 4 minutes
Total daily calls = 4B / 4 = 1 billion calls

Call signaling operations/call = 10 (setup, ring, answer, hang up, etc.)
Daily signaling = 1B × 10 = 10 billion operations
Signaling/sec (average) = 10B / 86400 = ~115,700/sec
Peak signaling/sec = ~350,000/sec
```

### Total Peak API Load
```xml
Component                              | RPS
---------------------------------------|-------------
Text Message Delivery                  | 1,500,000
Media Message Processing               | 650,000
Message Acknowledgments                | 31,000,000
Connection Management                  | 200,000
Status Updates                         | 800,000
Typing Indicators                      | 16,600,000
Voice/Video Call Signaling             | 350,000
Group Message Distribution             | 3,000,000
Voice Message Processing               | 81,000
Last Seen Updates                      | 500,000
Profile Picture Requests               | 100,000
Contact Sync Operations                | 50,000
Backup Operations                      | 20,000
---------------------------------------|-------------
Total Peak RPS                         | ~54,000,000
```

### 3. Bandwidth Estimation
### Request/Response Payload Sizes
**Text Message Payload:**
```xml
Field                          | Size
-------------------------------|--------
Sender ID (Phone Number Hash)  | 16 bytes
Receiver ID                    | 16 bytes
Message ID (UUID)              | 16 bytes
Timestamp                      | 8 bytes
Message Content (avg)          | 100 bytes
Encryption Metadata            | 32 bytes
Message Type Flag              | 2 bytes
-------------------------------|--------
Total                          | 190 bytes ≈ 0.19 KB
```

**Media Message Metadata:**
```xml
Field                          | Size
-------------------------------|--------
Message Header                 | 190 bytes
Media Type                     | 2 bytes
Media Size                     | 8 bytes
Media URL/Reference            | 64 bytes
Thumbnail Data                 | 5 KB (for images/videos)
Encryption Key                 | 32 bytes
-------------------------------|--------
Total Metadata                 | 5.3 KB
```

**Connection Heartbeat:**
```xml
Field                          | Size
-------------------------------|--------
Connection ID                  | 16 bytes
Timestamp                      | 8 bytes
Status Flag                    | 2 bytes
-------------------------------|--------
Total                          | 26 bytes
```

**Status Update Payload:**
```xml
Field                          | Size
-------------------------------|--------
User ID                        | 16 bytes
Status (online/offline/typing) | 2 bytes
Timestamp                      | 8 bytes
Last Seen                      | 8 bytes
-------------------------------|--------
Total                          | 34 bytes
```

### Peak Bandwidth Calculation
**Ingress (Incoming to Servers):**
```
Text Messages: 1,500,000 × 0.19 KB = 285 MB/sec
Media Metadata: 650,000 × 5.3 KB = 3,445 MB/sec = 3.4 GB/sec
Media Content (30% of messages): 
  - Average media size = 500 KB
  - Media uploads/sec = 2,150,000 × 0.3 × 500 KB = 322.5 GB/sec
Acknowledgments: 31,000,000 × 0.05 KB = 1,550 MB/sec = 1.55 GB/sec
Status Updates: 800,000 × 0.034 KB = 27.2 MB/sec
Typing Indicators: 16,600,000 × 0.034 KB = 564 MB/sec
Voice Data (48 kbps codec): 
  - Concurrent calls = 500,000
  - Bandwidth = 500,000 × 48 kbps / 8 = 3 GB/sec
Video Data (1 Mbps codec):
  - Concurrent calls = 200,000  
  - Bandwidth = 200,000 × 1 Mbps / 8 = 25 GB/sec
Other Operations: ~5 GB/sec

Total Ingress = 0.285 + 3.4 + 322.5 + 1.55 + 0.027 + 0.564 + 3 + 25 + 5 = ~361 GB/sec
Daily Ingress = 361 GB/sec × 86,400 sec = 31.2 PB/day
```

**Egress (Outgoing from Servers):**
```
Message Delivery: 1,500,000 × 0.19 KB = 285 MB/sec
Media Downloads: 322.5 GB/sec (same as uploads, delivered to recipients)
Acknowledgments: 31,000,000 × 0.05 KB = 1.55 GB/sec
Status Broadcasts: 800,000 × 0.034 KB = 27.2 MB/sec
Typing Indicators: 16,600,000 × 0.034 KB = 564 MB/sec
Call Media (Voice + Video): 28 GB/sec
Group Message Multiplier: 3,000,000 × 0.19 KB × 10 (avg group size) = 5.7 GB/sec
Profile Pictures: 100,000 × 50 KB = 5 GB/sec
Other Responses: ~5 GB/sec

Total Egress = 0.285 + 322.5 + 1.55 + 0.027 + 0.564 + 28 + 5.7 + 5 + 5 = ~368.6 GB/sec
Daily Egress = 368.6 GB/sec × 86,400 sec = 31.8 PB/day
```

**Total Daily Bandwidth:**
```shell
Ingress + Egress = 31.2 PB + 31.8 PB = ~63 PB/day
```

### 4. Storage Estimation
### User Profiles
**User Account Data:**
```xml
Fields per user:
- Phone Number (hashed): 32 bytes
- User ID (UUID): 16 bytes
- Display Name: 100 bytes
- Profile Picture Reference: 50 bytes
- About/Status Text: 200 bytes
- Last Seen: 8 bytes
- Account Created: 8 bytes
- Privacy Settings: 50 bytes
- Security Tokens: 128 bytes
- Linked Devices: 100 bytes
--------------------------------------
Total per user: ~692 bytes ≈ 0.7 KB

Total users: 3.5 billion (registered numbers)
Storage: 3.5B × 0.7 KB = 2.45 TB
```

### Message Storage
**Undelivered Messages (Offline Queue):**
```xml
Average undelivered messages per user: 100 messages
Users with undelivered messages (20% of DAU): 460M users

Message metadata per undelivered message: 0.19 KB
Undelivered messages storage: 460M × 100 × 0.19 KB = 8.74 TB

Media references in queue: 30% × 8.74 TB × 5.3/0.19 = 72.5 TB
Total offline queue: 8.74 + 72.5 = ~81 TB
```

**Message History (30-day retention for backup):**
```xml
Daily messages: 150 billion
Message size (text): 0.19 KB
Daily text storage: 150B × 0.7 × 0.19 KB = 19.95 TB/day

30-day storage: 19.95 TB × 30 = ~600 TB
```

### Media Storage
**Media Files:**
```xml
Daily media messages: 150B × 0.3 = 45 billion media files/day

Average media file size: 500 KB (weighted average of photos, videos, docs)
Daily media storage: 45B × 500 KB = 22.5 PB/day

7-day hot storage (CDN cache): 22.5 PB × 7 = 157.5 PB
30-day warm storage: 22.5 PB × 30 = 675 PB
90-day cold storage (user backups): 22.5 PB × 90 = 2,025 PB = 2 EB
```

**Profile Pictures:**
```xml
Total users with profile pictures: 2.8 billion (80% of users)
Average profile picture size: 50 KB (compressed)
Total storage: 2.8B × 50 KB = 140 TB
```

**Voice Messages:**
```xml
Daily voice messages: 7 billion
Average voice message size: 100 KB (60 seconds at 12 kbps)
Daily storage: 7B × 100 KB = 700 TB/day
30-day storage: 700 TB × 30 = 21 PB
```

**Status Updates (24-hour retention):**
```xml
Daily status posts: 500M users × 2 posts = 1 billion posts
Average status size: 200 KB (mix of photos and videos)
Daily storage: 1B × 200 KB = 200 TB
```

### Total Storage Summary
```xml
Component                              | Storage Size
---------------------------------------|------------------
User Profiles                          | 2.45 TB
Contact Lists (cached)                 | 500 TB
Offline Message Queue                  | 81 TB
Message History (30 days)              | 600 TB
Media Hot Storage (7 days)             | 157.5 PB
Media Warm Storage (30 days)           | 675 PB
Media Cold Storage (90 days)           | 2 EB
Profile Pictures                       | 140 TB
Voice Messages (30 days)               | 21 PB
Status Updates (24 hours)              | 200 TB
Group Metadata                         | 50 TB
Encryption Keys & Sessions             | 100 TB
Analytics & Logs                       | 500 TB
Backups & Redundancy (3x replication)  | 8 EB
---------------------------------------|------------------
Total Storage (with redundancy)        | ~11 EB
Active Hot Storage                     | ~160 PB
```

### 5. Database Sizing
### In-Memory Cache (Redis/Memcached)
**For ultra-low latency operations:**
- User session data (active users): 700M × 10 KB = 7 TB
- Online status cache: 2.3B × 1 KB = 2.3 TB
- Recent messages (last 100 per user): 2.3B × 100 × 0.19 KB = 43.7 TB
- Typing indicators: 10 GB
- Active call sessions: 50 GB
- **Total Cache: ~53 TB**

### NoSQL Database (Cassandra/HBase)
**For distributed, high-throughput data:**
- User profiles and contacts: 3 TB
- Message metadata (90 days): 2.5 PB
- Group chat metadata: 50 TB
- Device registration data: 200 TB
- Message delivery receipts: 5 PB
- **Total NoSQL: ~7.75 PB**

### Key-Value Store (Mnesia/ETS - In Erlang VM)
**For routing and connection management:**
- Connection routing tables: 5 TB
- Message routing metadata: 2 TB
- Presence information: 3 TB
- **Total KV Store: ~10 TB**

### Object Storage (S3/GCS/Azure Blob)
**For media and large files:**
- Media files (with lifecycle policies): 2.8 EB
- Voice messages: 100 PB
- Profile pictures: 200 TB
- Status media: 1 PB
- User backups: 500 PB
- **Total Object Storage: ~3.4 EB**

### Relational Database (PostgreSQL/MySQL)
**For critical transactional data:**
- User authentication & registration: 10 TB
- Payment transactions (WhatsApp Pay): 50 TB
- Business API usage logs: 100 TB
- Audit logs: 200 TB
- **Total RDBMS: ~360 TB**

### 6. Summary Table
| Metric | Value |
| ----- | ----- |
| **Peak RPS** | 54,000,000 requests/sec |
| **Daily Messages** | 150 billion |
| **Peak Bandwidth (Ingress)** | 361 GB/sec |
| **Peak Bandwidth (Egress)** | 368.6 GB/sec |
| **Daily Bandwidth** | ~63 PB/day |
| **Message Delivery/sec** | 1.5M - 2M per region |
| **Storage (Total with redundancy)** | ~11 EB |
| **Active Hot Storage** | 160 PB |
| **Database (NoSQL)** | 7.75 PB |
| **Database (RDBMS)** | 360 TB |
| **Cache Layer (In-Memory)** | 53 TB |
| **Object Storage** | 3.4 EB |
| **Concurrent Connections** | 700 million |

### 7. Infrastructure Sizing
### Application Servers (Erlang/BEAM)
```xml
Connection capacity per server: 1-2 million connections (optimized Erlang)
Required servers for connections: 700M / 1.5M = ~467 servers
With redundancy (3x): 1,400 chat servers

Message processing servers: 500 servers
With redundancy: 1,500 total application servers
```

### Load Balancers
```xml
Global load balancers: 50+ edge locations
Regional load balancers: 20 per major region
Total: 200+ load balancing nodes
```

### Database Sharding
```xml
Shard by: User ID hash (consistent hashing)
Number of shards: 10,000+ shards globally
Replication factor: 3x (primary + 2 replicas)
Total database nodes: 30,000+
```

### CDN & Edge Infrastructure
```xml
Edge locations: 100+ globally
CDN storage for media: 200 PB across edge nodes
CDN bandwidth capacity: 1 Tbps aggregate
```

### 8. Byte-Level Explanation
### Why These Sizes Matter
**Binary Protocol Efficiency:** WhatsApp uses FunXMPP (a modified XMPP) with binary encoding instead of XML, reducing message overhead by 60-70%. A typical XMPP stanza of 500 bytes becomes ~150 bytes in FunXMPP.

**Connection Optimization:** Each persistent WebSocket connection is maintained with minimal overhead (~26 bytes heartbeat every 30 seconds), allowing a single server with 64 GB RAM to handle 1-2 million concurrent connections.

**Message Encryption Overhead:** End-to-end encryption using Signal Protocol adds ~32 bytes per message for encryption metadata (keys, signatures), but this is negligible compared to content.

**Voice Codec Efficiency:** WhatsApp uses Opus codec at 48 kbps for voice calls, providing high quality while consuming just 360 KB per minute compared to traditional phone calls.

**Video Compression:** H.264 codec with adaptive bitrate (250 kbps - 1.5 Mbps) balances quality and bandwidth, using ~5 MB per minute on average connections.

### Optimization Techniques
1. **Message Batching:** Multiple messages to same recipient batched in single packet → 40% reduction in network overhead
2. **Differential Sync:** Only sync changes in contact list, not full list → 95% bandwidth savings
3. **Thumbnail Generation:** Images sent with 5 KB thumbnail first, full image loaded on demand → faster perceived delivery
4. **Adaptive Media Quality:** Videos automatically compressed based on network speed → consistent user experience
5. **Message Pruning:** Old messages archived/deleted automatically → keeps active storage manageable
6. **Connection Pooling:** Reuse existing connections for multiple operations → reduces SSL handshake overhead
7. **Geographic Sharding:** Route users to nearest data center → 30-50ms latency reduction

---