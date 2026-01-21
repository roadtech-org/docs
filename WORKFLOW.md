# RoadTech - System Workflow Documentation

## Overview

RoadTech follows a request-response architecture where users can request roadside assistance and mechanics can accept and fulfill those requests in real-time.

---

## 1. User Authentication Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │     │   Backend   │     │  Security   │     │  Database   │
│  (React)    │     │ (Spring)    │     │   Filter    │     │  (MySQL)    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  POST /auth/login │                   │                   │
       │  {email, password}│                   │                   │
       │──────────────────>│                   │                   │
       │                   │                   │                   │
       │                   │  Validate credentials                 │
       │                   │──────────────────────────────────────>│
       │                   │                   │                   │
       │                   │  User data        │                   │
       │                   │<──────────────────────────────────────│
       │                   │                   │                   │
       │                   │  Generate JWT     │                   │
       │                   │  + Refresh Token  │                   │
       │                   │                   │                   │
       │  {accessToken,    │                   │                   │
       │   refreshToken,   │                   │                   │
       │   user}           │                   │                   │
       │<──────────────────│                   │                   │
       │                   │                   │                   │
       │  Store tokens in  │                   │                   │
       │  localStorage     │                   │                   │
       │                   │                   │                   │
```

### Token Refresh Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │     │   Backend   │     │  Database   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │  API Request      │                   │
       │  (expired token)  │                   │
       │──────────────────>│                   │
       │                   │                   │
       │  401 Unauthorized │                   │
       │<──────────────────│                   │
       │                   │                   │
       │  POST /auth/refresh                   │
       │  {refreshToken}   │                   │
       │──────────────────>│                   │
       │                   │                   │
       │                   │  Validate token   │
       │                   │──────────────────>│
       │                   │                   │
       │  New tokens       │                   │
       │<──────────────────│                   │
       │                   │                   │
       │  Retry original   │                   │
       │  request          │                   │
       │──────────────────>│                   │
       │                   │                   │
```

---

## 2. Service Request Flow

### 2.1 User Creates Request

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    User     │     │   Backend   │     │  WebSocket  │     │  Mechanics  │
│   Client    │     │   Server    │     │   Broker    │     │  (Online)   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  POST /service-requests               │                   │
       │  {issueType,      │                   │                   │
       │   latitude,       │                   │                   │
       │   longitude,      │                   │                   │
       │   description}    │                   │                   │
       │──────────────────>│                   │                   │
       │                   │                   │                   │
       │                   │  Save request     │                   │
       │                   │  status=PENDING   │                   │
       │                   │                   │                   │
       │                   │  Broadcast to     │                   │
       │                   │  /topic/mechanic/requests             │
       │                   │──────────────────>│                   │
       │                   │                   │                   │
       │                   │                   │  NEW_REQUEST      │
       │                   │                   │  notification     │
       │                   │                   │──────────────────>│
       │                   │                   │                   │
       │  {requestId,      │                   │                   │
       │   status: PENDING}│                   │                   │
       │<──────────────────│                   │                   │
       │                   │                   │                   │
       │  Redirect to      │                   │                   │
       │  /request/{id}    │                   │                   │
       │                   │                   │                   │
```

### 2.2 Mechanic Accepts Request

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Mechanic   │     │   Backend   │     │  WebSocket  │     │    User     │
│   Client    │     │   Server    │     │   Broker    │     │   Client    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  PUT /mechanic/requests/{id}/accept   │                   │
       │──────────────────>│                   │                   │
       │                   │                   │                   │
       │                   │  Update request   │                   │
       │                   │  status=ACCEPTED  │                   │
       │                   │  mechanicId=X     │                   │
       │                   │  Calculate ETA    │                   │
       │                   │                   │                   │
       │                   │  Send to          │                   │
       │                   │  /topic/user/{id} │                   │
       │                   │  /topic/request/{id}                  │
       │                   │──────────────────>│                   │
       │                   │                   │                   │
       │                   │                   │  STATUS_UPDATE    │
       │                   │                   │  {status,         │
       │                   │                   │   mechanic,       │
       │                   │                   │   ETA}            │
       │                   │                   │──────────────────>│
       │                   │                   │                   │
       │  {request with    │                   │                   │
       │   mechanic info}  │                   │                   │
       │<──────────────────│                   │                   │
       │                   │                   │                   │
```

### 2.3 Live Location Tracking

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Mechanic   │     │   Backend   │     │  WebSocket  │     │    User     │
│   Client    │     │   Server    │     │   Broker    │     │   Client    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  [Every 30 sec]   │                   │                   │
       │  PUT /mechanic/location               │                   │
       │  {lat, lng}       │                   │                   │
       │──────────────────>│                   │                   │
       │                   │                   │                   │
       │                   │  Update profile   │                   │
       │                   │  location         │                   │
       │                   │                   │                   │
       │                   │  For each active  │                   │
       │                   │  request, send to │                   │
       │                   │  /topic/request/{id}                  │
       │                   │──────────────────>│                   │
       │                   │                   │                   │
       │                   │                   │  LOCATION_UPDATE  │
       │                   │                   │  {lat, lng,       │
       │                   │                   │   mechanicId}     │
       │                   │                   │──────────────────>│
       │                   │                   │                   │
       │                   │                   │       Update map  │
       │                   │                   │       marker      │
       │                   │                   │                   │
```

### 2.4 Service Completion Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Mechanic   │     │   Backend   │     │    User     │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │  PUT /mechanic/requests/{id}/start    │
       │──────────────────>│                   │
       │                   │                   │
       │                   │  status=IN_PROGRESS
       │                   │──────────────────>│
       │                   │                   │
       │  [Service work]   │                   │
       │                   │                   │
       │  PUT /mechanic/requests/{id}/complete │
       │──────────────────>│                   │
       │                   │                   │
       │                   │  status=COMPLETED │
       │                   │  Update mechanic  │
       │                   │  totalJobs++      │
       │                   │──────────────────>│
       │                   │                   │
       │                   │       Show completion
       │                   │       message     │
       │                   │                   │
```

---

## 3. Request State Machine

```
                    ┌───────────┐
                    │  PENDING  │
                    └─────┬─────┘
                          │
            ┌─────────────┼─────────────┐
            │             │             │
            ▼             │             ▼
     ┌───────────┐        │      ┌───────────┐
     │ CANCELLED │        │      │ ACCEPTED  │
     └───────────┘        │      └─────┬─────┘
            ▲             │            │
            │             │            ▼
            │             │     ┌───────────┐
            │             │     │IN_PROGRESS│
            │             │     └─────┬─────┘
            │             │           │
            └─────────────┘           ▼
                              ┌───────────┐
                              │ COMPLETED │
                              └───────────┘
```

### State Transitions

| Current State | Action | Next State | Triggered By |
|--------------|--------|------------|--------------|
| - | Create Request | PENDING | User |
| PENDING | Accept | ACCEPTED | Mechanic |
| PENDING | Cancel | CANCELLED | User |
| ACCEPTED | Start Service | IN_PROGRESS | Mechanic |
| ACCEPTED | Reject | PENDING | Mechanic |
| ACCEPTED | Cancel | CANCELLED | User |
| IN_PROGRESS | Complete | COMPLETED | Mechanic |

---

## 4. WebSocket Topics

| Topic | Publisher | Subscribers | Payload |
|-------|-----------|-------------|---------|
| `/topic/mechanic/requests` | Backend | All online mechanics | New pending requests |
| `/topic/mechanic/{id}` | Backend | Specific mechanic | Request cancelled notifications |
| `/topic/user/{id}` | Backend | Specific user | Status updates |
| `/topic/request/{id}` | Backend | User & Mechanic | Status & location updates |

### Message Types

```typescript
interface WebSocketMessage {
  type: 'NEW_REQUEST' | 'STATUS_UPDATE' | 'LOCATION_UPDATE' | 'REQUEST_CANCELLED';
  payload: object;
  timestamp: number;
}
```

---

## 5. Database Relationships

```
┌─────────────────┐       ┌─────────────────────┐
│     users       │       │  mechanic_profiles  │
├─────────────────┤       ├─────────────────────┤
│ id (PK)         │───┐   │ id (PK)             │
│ email           │   │   │ user_id (FK, UQ)    │───┐
│ password        │   └──>│ specializations     │   │
│ full_name       │       │ is_available        │   │
│ phone           │       │ current_latitude    │   │
│ role            │       │ current_longitude   │   │
│ is_active       │       │ rating              │   │
│ created_at      │       │ total_jobs          │   │
└────────┬────────┘       └─────────────────────┘   │
         │                                          │
         │  ┌───────────────────────────────────────┘
         │  │
         ▼  ▼
┌─────────────────────┐       ┌─────────────────┐
│  service_requests   │       │ refresh_tokens  │
├─────────────────────┤       ├─────────────────┤
│ id (PK)             │       │ id (PK)         │
│ user_id (FK)        │───┐   │ user_id (FK)    │
│ mechanic_id (FK)    │   │   │ token           │
│ issue_type          │   │   │ expires_at      │
│ description         │   │   └─────────────────┘
│ latitude            │   │
│ longitude           │   │
│ status              │   │
│ estimated_arrival   │   │
│ created_at          │   │
│ accepted_at         │   │
│ completed_at        │   │
└─────────────────────┘   │
                          │
                          └─── References users table
```

---

## 6. API Request/Response Examples

### Create Service Request

**Request:**
```http
POST /api/service-requests
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "issueType": "FLAT_TIRE",
  "description": "Front left tire is flat",
  "latitude": 28.6139,
  "longitude": 77.2090,
  "address": "123 Main Street, Delhi"
}
```

**Response:**
```json
{
  "id": 1,
  "userId": 5,
  "mechanicId": null,
  "issueType": "FLAT_TIRE",
  "description": "Front left tire is flat",
  "latitude": 28.6139,
  "longitude": 77.2090,
  "address": "123 Main Street, Delhi",
  "status": "PENDING",
  "estimatedArrival": null,
  "createdAt": "2024-01-15T10:30:00",
  "acceptedAt": null,
  "completedAt": null
}
```

### Accept Request (Mechanic)

**Request:**
```http
PUT /api/mechanic/requests/1/accept
Authorization: Bearer <mechanic_jwt_token>
```

**Response:**
```json
{
  "id": 1,
  "userId": 5,
  "mechanicId": 10,
  "issueType": "FLAT_TIRE",
  "status": "ACCEPTED",
  "estimatedArrival": "2024-01-15T10:45:00",
  "acceptedAt": "2024-01-15T10:32:00",
  "user": {
    "id": 5,
    "fullName": "John Doe",
    "phone": "+919876543210"
  },
  "mechanic": {
    "id": 10,
    "fullName": "Mike Smith",
    "phone": "+919876543211",
    "rating": 4.5,
    "totalJobs": 150
  }
}
```
