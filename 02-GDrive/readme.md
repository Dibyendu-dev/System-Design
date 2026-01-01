# Google Drive / Dropbox Design

## 1. Requirements

### 1.1 Functional
- Users can upload files  
- Users can download files  
- Users can share files  
- Users can sync files across devices  

### 1.2 Non-Functional
- Availability prioritized over consistency  
- Support for large files (e.g., 100GB)  
- Secure and reliable system  
- Minimal latency  

---

## 2. Core Entities
- **File**  
- **FileMetadata**  
- **User**  

---

## 3. API

### 3.1 Upload File
```
POST /files
{
    "file": "<file_data>",
    "fileMetadata": {
        "name": "example.txt",
        "size": 1024,
        "type": "text/plain"
    }
}
```

### 3.2 Download File
```
GET /files/{file_id}

Response:
{
    "file": "<file_data>",
    "fileMetadata": {
        "name": "example.txt",
        "size": 1024,
        "type": "text/plain"
    }
}
```

### 3.3 Share File
```
POST /files/{file_id}/share
{
    "users": ["user1@example.com", "user2@example.com"]
}
```

### 3.4 Get File Changes to Sync
```
GET /files/{file_id}/changes

Response:
{
    "fileMetadata": [
        {
            "version": 2,
            "timestamp": "2024-01-01T12:00:00Z",
            "modifiedBy": "user@example.com"
        }
    ]
}
```

## 4. High level design
### 4.1 📁 File Uploading: Where to store file bytes and metadata

#### 1️⃣ Where Should File Metadata Be Stored?

### Metadata Includes:
```
fileId
userId
fileName
fileSize
contentType
storagePath
version
permissions
checksum
createdAt
```

### Why NoSQL Works Well Here (Your Thinking is Correct ✅)

- **Flexible schema** (versions, sharing, permissions)
- **High read/write throughput**
- **Easy horizontal scaling**
- **Metadata ≠ transactional money data**

### Common Choices:

- DynamoDB
- MongoDB
- Cassandra

### 📌 Conclusion (Metadata):

**File metadata → NoSQL database**

---

# File Storage Architecture Guide

## 1️⃣ Where Should Raw File Bytes Be Stored?

### ❌ NOT on Your Primary Application Server

Storing files directly on the API / primary server causes:

- **Disk limits** (100GB files 🚫)
- **Server crashes affect file availability**
- **Difficult horizontal scaling**
- **High latency under load**

> 👉 **Primary servers should never store raw file bytes**

### ✅ Use Blob / Object Storage

Raw file bytes should be stored in distributed object storage systems like:

- Amazon S3
- Google Cloud Storage
- Azure Blob Storage

#### Why Object Storage?

- Designed for huge files (TB scale)
- Automatically replicated (high availability)
- Cheap compared to DB or local disk
- Supports resumable & multipart uploads

### 📌 Conclusion (Raw Bytes):

**File bytes → Object / Blob Storage**

---

## 2️⃣ How Do Users Upload Files to Blob Storage?

### ❌ User → App Server → Blob Storage (Bad Approach)

**Reasons:**

- App server becomes bottleneck
- Huge bandwidth cost
- Server memory pressure
- Poor latency

![G-D](snaps/G1.png)

### ✅ User → Blob Storage Directly (Best Practice)

This is done using a **pre-signed URL**.

#### Flow (Simplified):

1. **User asks backend:** "I want to upload a file"
2. **Backend:**
   - Authenticates user
   - Creates metadata record (empty / pending)
   - Generates pre-signed upload URL
3. **User uploads file directly to blob storage**
4. **Blob storage confirms upload**

---
![G-D](snaps/G2.png)
