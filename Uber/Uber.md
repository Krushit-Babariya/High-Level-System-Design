# Uber

---

## Requirements
### **Function Requirements**
#### **Core Requirements**
- Riders should be able to input their pickup and destination locations to receive an estimated fare before booking a ride.
- Riders should be able to request a ride after reviewing and accepting the estimated fare provided by the system.
- Upon receiving a ride request, the system should automatically match the rider with the nearest available driver.
- Drivers should be able to accept or decline incoming ride requests and receive turn-by-turn navigation to pickup and drop-off locations.
- Drivers should be able to toggle their availability status (online/offline) to control when they receive ride requests.
- Both riders and drivers should be able to track the real-time location and status of their ongoing ride on a map interface.
- Riders should be able to view all available drivers/cabs in their vicinity on a map before requesting a ride.
- The system should handle secure payment transactions and automatically generate digital receipts after trip completion.
**Below the Line**

- Riders should be able to rate drivers and provide feedback after completing a trip.
- Drivers should be able to rate riders to maintain community standards and safety.
- Riders should be able to schedule rides in advance for future dates and times.
- Riders should be able to request different categories of rides based on vehicle type and comfort level (e.g., UberGo, Sedan, XL, Premium).
### **Non-Function Requirements**
#### **Core Requirements**
- The system should match riders with drivers within 1 minute of request submission or clearly indicate matching failure.
- The system should ensure strong consistency in ride matching to prevent any driver from being assigned multiple rides simultaneously.
- The system should handle high throughput of up to 100,000 simultaneous ride requests from the same geographic location during peak hours.
#### **Below the line**
- The system should ensure the security and privacy of user and driver data, complying with regulations like GDPR.
- The system should be resilient to failures, with redundancy and failover mechanisms in place.
- The system should have robust monitoring, logging, and alerting to quickly identify and resolve issues.
- The system should facilitate easy updates and maintenance without significant downtime (CI/CD pipelines).
---

## Capacity Estimation
### 1. Traffic & User Assumptions
### User Base
- **Monthly Active Users (MAU):** 180 million users
- **Total Drivers & Couriers:** 8.8 million globally
- **Daily Active Riders:** 20 million
- **Daily Active Drivers:** 2 million
- **Annual Trips:** 12 billion trips
### Daily Operations
- **Average Daily Rides:** 30 million trips/day
- **Peak Concurrent Users:** 3 million riders (10% of DAU during peak hours)
- **Peak Active Drivers:** 300,000 drivers (15% of daily active drivers)
### Read/Write Operations
- **Read:Write Ratio:** 100:1 (location queries are 100x more frequent than ride bookings)
### **2. Requests Per Second (RPS) Calculation**
### Peak Ride Requests
```xml
Daily rides = 30 million
Peak hour assumption = 10% of daily traffic
Peak hour rides = 30M × 0.10 = 3 million rides

Peak RPS = 3M rides / 3600 seconds = ~800 rides/sec
With safety margin (2x) = ~1,600 RPS for ride requests
```
### Location Update Frequency
**Driver Location Updates:**

- Active drivers send location every 3 seconds
- Peak active drivers = 300,000
```
 Location updates/sec = 300,000 drivers / 3 seconds
                      = 100,000 updates/sec
```
**Rider Location Tracking:**

- Active riders during peak = 3 million
- Update frequency = every 5 seconds (less frequent than drivers)
```
Rider location queries/sec = 3M / 5 = 600,000 queries/sec
```
### Total Peak API Load
```xml
Component                    | RPS
-----------------------------|----------
Ride Requests                | 1,600
Driver Location Updates      | 100,000
Rider Location Queries       | 600,000
ETA Calculations             | 1,600
Fare Estimations             | 1,600
Payment Processing           | 1,600
Driver Matching Queries      | 1,600
Real-time Tracking           | 300,000
Ratings & Reviews            | 850
-----------------------------|----------
Total Peak RPS               | ~1,000,000
```
### 3. Bandwidth Estimation
### Request/Response Payload Sizes
**Ride Request Payload:**

```xml
Field                     | Size
--------------------------|--------
User ID (UUID)            | 16 bytes
Pickup Location (lat/lon) | 16 bytes
Dropoff Location          | 16 bytes
Ride Type                 | 4 bytes
Timestamp                 | 8 bytes
Payment Method            | 4 bytes
User Preferences          | 20 bytes
--------------------------|--------
Total                     | 84 bytes ≈ 0.084 KB
```
**Driver Location Update:**

```xml
Field                     | Size
--------------------------|--------
Driver ID                 | 16 bytes
Current Location          | 16 bytes
Heading/Direction         | 4 bytes
Speed                     | 4 bytes
Timestamp                 | 8 bytes
Status (available/busy)   | 2 bytes
--------------------------|--------
Total                     | 50 bytes = 0.05 KB
```
**Location Query Response:**

```xml
Field                       | Size
----------------------------|--------
Driver IDs (10 drivers)     | 160 bytes
Locations (10 coordinates)  | 160 bytes
ETAs (10 estimates)         | 40 bytes
Ratings                     | 40 bytes
Vehicle Info                | 100 bytes
----------------------------|--------
Total                       | 500 bytes = 0.5 KB
```
### Peak Bandwidth Calculation
**Ingress (Incoming to Servers):**

```
Location Updates: 100,000 updates/sec × 0.05 KB = 5,000 KB/sec = 5 MB/sec
Ride Requests: 1,600 requests/sec × 0.084 KB = 134.4 KB/sec = 0.1344 MB/sec
Rider Queries: 600,000 queries/sec × 0.084 KB = 50,400 KB/sec = 50.4 MB/sec
Other APIs: ~10 MB/sec

Total Ingress = 5 + 0.1344 + 50.4 + 10 = ~65.54 MB/sec
Daily Ingress = 65.54 MB/sec × 86,400 sec = 5.66 TB/day
```
**Egress (Outgoing from Servers):**

```
Location Query Responses: 600,000 × 0.5 KB = 300,000 KB/sec = 300 MB/sec
Real-time Tracking: 300,000 × 0.5 KB = 150 MB/sec
Ride Confirmations: 1,600 × 1 KB = 1.6 MB/sec
Other Responses: ~50 MB/sec

Total Egress = 300 + 150 + 1.6 + 50 = ~501.6 MB/sec
Daily Egress = 501.6 MB/sec × 86,400 sec = 43.3 TB/day
```
**Total Daily Bandwidth:**

```shell
Ingress + Egress = 5.66 TB + 43.3 TB = ~49 TB/day
```
### 4. Storage Estimation
### User & Driver Profiles
**Rider Profiles:**

```xml
Fields per rider:
- User ID (UUID): 16 bytes
- Name: 50 bytes
- Email: 50 bytes
- Phone: 15 bytes
- Payment Methods (2-3): 100 bytes
- Home/Work Locations: 50 bytes
- Preferences: 50 bytes
- Profile Picture Reference: 50 bytes
- Ratings Data: 20 bytes
- Created/Updated Timestamps: 16 bytes
--------------------------------------
Total per rider: ~417 bytes ≈ 0.5 KB

Total riders: 180 million
Storage: 180M × 0.5 KB = 90 GB
```
**Driver Profiles:**

```shell
Fields per driver:
- Driver ID: 16 bytes
- Personal Info (name, email, phone): 115 bytes
- License Details: 100 bytes
- Vehicle Information: 150 bytes
- Bank Account Details: 100 bytes
- Insurance Details: 100 bytes
- Documents (references): 100 bytes
- Rating History: 50 bytes
- Background Check Data: 100 bytes
- Current Status: 10 bytes
--------------------------------------
Total per driver: ~841 bytes ≈ 1 KB

Total drivers: 8 million
Storage: 8M × 1 KB = 8 GB
```
**Total Profile Storage:** 90 GB + 8 GB = **~100 GB**

### Ride History Data
**Per Ride Record:**

```xml
Field                        | Size
-----------------------------|--------
Ride ID (UUID)               | 16 bytes
Rider ID                     | 16 bytes
Driver ID                    | 16 bytes
Pickup Location              | 16 bytes
Dropoff Location             | 16 bytes
Pickup Timestamp             | 8 bytes
Dropoff Timestamp            | 8 bytes
Fare Amount                  | 8 bytes
Distance                     | 4 bytes
Duration                     | 4 bytes
Status                       | 2 bytes
Payment Method               | 4 bytes
Ride Type                    | 4 bytes
Surge Multiplier             | 4 bytes
-----------------------------|--------
Total                        | 126 bytes ≈ 0.13 KB
```
**Daily Ride Storage:**

```xml
Daily rides: 30 million
Storage per day: 30M × 0.13 KB = 3,900 MB ≈ 3.9 GB/day
```
**Yearly Ride Storage:**

```bash
Annual storage: 3.9 GB × 365 = 1,423 GB ≈ ~1.4 TB/year
```
**5-Year Retention:**

```bash
5-year storage: 1.4 TB × 5 = 7 TB
```
### Location History Data
**Driver Location Points:**

```xml
Each location point:
- Driver ID: 16 bytes
- Latitude: 8 bytes
- Longitude: 8 bytes
- Timestamp: 8 bytes
- Speed: 4 bytes
- Heading: 4 bytes
------------------------
Total: 48 bytes

Updates per driver per day:
- Active hours: 8 hours
- Update frequency: every 3 seconds
- Points per day: (8 × 3600) / 3 = 9,600 points

Storage per driver per day: 9,600 × 48 bytes = 460,800 bytes ≈ 0.46 MB

Total daily active drivers: 2 million
Daily location storage: 2M × 0.46 MB = 920 GB/day
```
**Retention Policy:** 30 days (for dispute resolution and analytics)

```xml
30-day storage: 920 GB × 30 = 27.6 TB
```
### Total Storage Summary
```xml
Component                    | Storage Size
-----------------------------|------------------
User Profiles                | 90 GB
Driver Profiles              | 8.8 GB
Ride History (5 years)       | 7 TB
Location History (30 days)   | 27.6 TB
Payment Transaction Logs     | 500 GB
Ratings & Reviews            | 200 GB
Analytics & Logs             | 2 TB
Backups & Redundancy (3x)    | 112 TB
-----------------------------|------------------
Total Storage                | ~150 TB
```
### 5. Database Sizing
### Relational Database (PostgreSQL/MySQL)
**For transactional data:**

- User/Driver profiles: 100 GB
- Active rides (in-progress): 50 GB
- Payment records: 500 GB
- **Total RDBMS: ~650 GB**
### NoSQL Database (Cassandra/DynamoDB)
**For high-throughput data:**

- Location history: 27.6 TB
- Ride history: 7.2 TB
- Real-time driver status: 100 GB
- **Total NoSQL: ~35 TB**
### Cache Layer (Redis/Memcached)
**For hot data:**

- Active driver locations: 50 GB
- Ongoing rides data: 20 GB
- User session data: 30 GB
- **Total Cache: ~100 GB**
### Object Storage (S3/GCS)
**For media and documents:**

- Profile pictures: 500 GB
- Driver documents: 1 TB
- Ride receipts/invoices: 500 GB
- **Total Object Storage: ~2 TB**
### 6. Summary Table
| Metric | Value |
| ----- | ----- |
| **Peak RPS** | 1,000,000 requests/sec |
| **Daily Rides** | 30 million |
| **Peak Bandwidth** | 567 MB/sec (ingress + egress) |
| **Daily Bandwidth** | ~49 TB/day |
| **Location Updates/sec** | 100,000 updates/sec |
| **Storage (Total)** | ~150 TB |
| **Database (RDBMS)** | 650 GB |
| **Database (NoSQL)** | 35 TB |
| **Cache Layer** | 100 GB |
| **Object Storage** | 2 TB |
### Application Servers
```xml
Assuming each server handles: 1,000 RPS
Required servers at peak: 1,000,000 / 1,000 = 1,000 servers
With redundancy (3x): 3,000 application servers
```
### Database Sharding
```xml
Shard by geographic region (city-based)
Estimated shards: 100-200 shards globally
Replication factor: 3x for high availability
```
### CDN & Edge Locations
```xml
Static assets and map tiles: 5 TB
CDN bandwidth: 10 TB/day
Edge locations: 50+ globally
```
### 8. Byte-Level Explanation
### Why These Sizes Matter
**UUID (16 bytes):** Universally unique identifier ensuring no collision across billions of records

**Timestamp (8 bytes):** Unix timestamp in milliseconds provides precision for time-based queries

**Location Coordinates (16 bytes):** Latitude + Longitude as double precision floating point (8 bytes each) gives ~1cm accuracy

**Status Flags (2 bytes):** Bitwise flags for multiple states (available, busy, offline, etc.)

### Optimization Techniques
1. **Data Compression:** Use Snappy/LZ4 for location data → 30-40% size reduction
2. **Archival:** Move old rides (>1 year) to cold storage → 70% cost reduction
3. **Indexes:** Geospatial indexes (R-tree, QuadTree) for fast location queries
4. **Partitioning:** Time-based partitioning for ride history → improved query performance


