# Section 9: Complete REST API Documentation

## 9.1 API Conventions

- **Base URL:** `https://api.rydlink.com/api/v1/`
- **Auth:** `Authorization: Bearer <access_token>` (Simple JWT)
- **Content-Type:** `application/json`
- **Date format:** ISO 8601 (`2024-01-15T10:30:00Z`)
- **ID format:** UUID
- **Error envelope:** `{ "error": "code", "message": "...", "details": {} }`
- **Success envelope:** `{ "data": {...} }` or `{ "results": [...], "count": N }` for lists

## 9.2 Module 1: Authentication

| Method | URL | Purpose | Auth | Role | Request Body |
|--------|-----|---------|------|------|--------------|
| POST | `/auth/register` | Register user | No | – | `{ email, password, phone, first_name, last_name }` |
| POST | `/auth/login` | Login | No | – | `{ email, password }` |
| POST | `/auth/refresh` | Refresh access token | No | – | `{ refresh }` |
| POST | `/auth/logout` | Logout / blacklist refresh | Yes | Any | `{ refresh }` |
| GET | `/users/me` | Get own profile | Yes | Any | – |
| PATCH | `/users/me` | Update own profile | Yes | Any | `{ first_name, last_name, phone, ... }` |
| POST | `/auth/password-reset` | Request password reset | No | – | `{ email }` |

**Success (register/login):** `200` `{ "access": "...", "refresh": "...", "user": {...} }`
**Errors:** `400` validation, `401` invalid credentials, `409` duplicate email

## 9.3 Module 2: Rides

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| POST | `/rides` | Create ride | Yes | Any | `{ name, start_lat, start_lng, start_address, dest_lat, dest_lng, dest_address, departure_time }` |
| GET | `/rides/{id}` | Get ride details | Yes | Member | – |
| PATCH | `/rides/{id}` | Update ride | Yes | Leader | `{ name, departure_time, ... }` |
| DELETE | `/rides/{id}` | Cancel ride | Yes | Leader | – |
| GET | `/rides` | List my rides | Yes | Any | `?status=planned\|active\|completed` |
| POST | `/rides/{id}/start` | Start ride | Yes | Leader | – |
| POST | `/rides/{id}/end` | End/complete ride | Yes | Leader | – |

**Validation:** departure_time > now; start ≠ destination; only leader can patch/delete/start/end

## 9.4 Module 3: Invitations

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| POST | `/rides/{id}/invitations` | Send invitation | Yes | Leader | `{ invitee_email \| invitee_username }` |
| GET | `/invitations` | List received invitations | Yes | Any | `?status=pending` |
| POST | `/invitations/{token}/respond` | Accept/reject | Yes | Invitee | `{ action: accept\|reject }` |

**Validation:** inviter is leader; invitee exists; not already participant; token not expired; single-use

## 9.5 Module 4: Participants

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/participants` | List participants | Yes | Member | – |
| PATCH | `/rides/{id}/participants/{user_id}` | Approve/remove | Yes | Leader | `{ status: joined\|left }` |

**Validation:** only leader can modify; user must be member

## 9.6 Module 5: Locations

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/locations` | Latest locations all participants | Yes | Member | – |

**WebSocket handles real-time updates; REST is for initial sync.**

## 9.7 Module 6: Routes

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/route` | Get route polyline, distance, ETA | Yes | Member | – |

## 9.8 Module 7: Alerts

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/alerts` | List alerts | Yes | Member | `?status=active` |
| GET | `/rides/{id}/separation-status` | Get rider separation status | Yes | Member | – |
| PATCH | `/alerts/{id}/resolve` | Resolve alert | Yes | Leader | – |

**Alert types:**
- `separation_warning` — Rider > 500 m from group for 60 seconds
- `high_separation` — Rider > 1,000 m from group for 30 seconds  
- `regrouped` — Rider returned within 300 m for 30 seconds

**Validation:** only leader can resolve; user must be ride member

## 9.9 Module 8: Meeting Points

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/meeting-points` | List recommendations | Yes | Member | – |
| POST | `/meeting-points/{id}/approve` | Approve recommendation | Yes | Leader | – |
| POST | `/meeting-points/{id}/reject` | Reject recommendation | Yes | Leader | – |

## 9.9 Module 8: Meeting Points

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/{id}/meeting-points` | List recommendations | Yes | Member | – |
| POST | `/meeting-points/{id}/approve` | Approve recommendation | Yes | Leader | – |
| POST | `/meeting-points/{id}/reject` | Reject recommendation | Yes | Leader | – |

## 9.10 Module 9: Ride Events

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| POST | `/rides/{id}/events` | Create event | Yes | Member | `{ type, lat, lng, address, note, planned_time }` |
| GET | `/rides/{id}/events` | List events | Yes | Member | – |
| PATCH | `/events/{id}` | Update/complete event | Yes | Creator or Leader | `{ status }` |

## 9.11 Module 10: Emergencies

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| POST | `/rides/{id}/emergencies` | Report emergency | Yes | Member | `{ type, severity, description, lat, lng }` |
| GET | `/rides/{id}/emergencies` | List emergencies | Yes | Member | – |
| PATCH | `/emergencies/{id}` | Update status | Yes | Reporter or Leader | `{ status }` |

## 9.12 Module 11: Devices & Notifications

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| POST | `/devices/tokens` | Register device token | Yes | Any | `{ token, platform }` |
| GET | `/notifications` | My notification history | Yes | Any | `?read=false` |

## 9.13 Module 12: Connectivity

No dedicated REST endpoints (WebSocket-driven). Health check: `GET /health`

## 9.14 Module 13: Ride History

| Method | URL | Purpose | Auth | Role | Body |
|--------|-----|---------|------|------|------|
| GET | `/rides/history` | My completed rides | Yes | Any | – |
| GET | `/rides/{id}` | Completed ride detail | Yes | Participant | – |

## 9.15 Example: Create Ride

**Request:** `POST /api/v1/rides`
```json
{
  "name": "Weekend Mountain Trip",
  "start_lat": 40.7128,
  "start_lng": -74.0060,
  "start_address": "New York, NY",
  "dest_lat": 40.7589,
  "dest_lng": -73.9851,
  "dest_address": "Times Square, NY",
  "departure_time": "2024-02-15T08:00:00Z"
}
```

**Response 201:**
```json
{
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "Weekend Mountain Trip",
    "status": "planned",
    "invitation_code": "RYD-A1B2C3",
    "leader_id": "550e8400-e29b-41d4-a716-446655440000",
    "created_at": "2024-02-10T10:00:00Z"
  }
}
```

## 9.16 Example: Send Invitation

**Request:** `POST /api/v1/rides/{ride_id}/invitations`
```json
{ "invitee_email": "bob@example.com" }
```

**Response 201:**
```json
{
  "data": {
    "id": "770e8400-...",
    "token": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "pending",
    "expires_at": "2024-02-11T10:00:00Z"
  }
}
```

## 9.17 Example: Location Query (REST fallback)

**Request:** `GET /api/v1/rides/{ride_id}/locations`

**Response 200:**
```json
{
  "data": [
    {
      "user_id": "550e8400-...",
      "user_name": "Alice Smith",
      "location": { "lat": 40.7128, "lng": -74.0060 },
      "accuracy": 5.0,
      "speed": 12.5,
      "recorded_at": "2024-02-15T08:05:00Z",
      "server_received_at": "2024-02-15T08:05:01Z"
    }
  ]
}
```
