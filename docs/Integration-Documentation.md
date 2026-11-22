# RydLink - Frontend–Backend Integration Documentation

## Table of Contents
1. [Frontend and Backend API Integration](#1-frontend-and-backend-api-integration)
2. [WebSocket Integration](#2-websocket-integration)
3. [PostgreSQL and PostGIS Integration](#3-postgresql-and-postgis-integration)
4. [Redis Integration](#4-redis-integration)
5. [MapLibre and OpenStreetMap Integration](#5-maplibre-and-openstreetmap-integration)
6. [OSRM Routing Integration](#6-osrm-routing-integration)
7. [Overpass API Integration](#7-overpass-api-integration)
8. [Push Notification Integration](#8-push-notification-integration)
9. [Background GPS Tracking Integration](#9-background-gps-tracking-integration)

---

## 1. Frontend and Backend API Integration

### 1.1 Module Name
Frontend and Backend API Integration

### 1.2 Module Objective
Define complete data flow between React Native frontend and Django REST backend, including all API endpoints, request/response formats, authentication requirements, and error handling for every module.

### 1.3 Module Purpose
Ensure consistent integration patterns between frontend and backend, provide development contracts for both teams, and document every API communication with complete specifications.

### 1.4 Functional Requirements
- Document every API endpoint with method, path, auth, request, and response
- Specify validation rules for all inputs
- Define error response formats and HTTP status codes
- Map every endpoint to its frontend screen
- Identify backend module responsible for each endpoint
- Document WebSocket event payloads and broadcasting rules
- Specify reconnection and recovery procedures

### 1.5 Technologies and Packages Required
- Axios 1.x (HTTP client)
- Django REST Framework 3.14+ (backend)
- Simple JWT 5.3+ (authentication)
- Django Channels 4.0+ (WebSocket)
- Redis 6/7 (channel layer)
- WebSocket API (platform native)
- Expo Notifications (push)
- FCM SDK (production push, Android)

### 1.6 Dependencies on Other Modules
- Every frontend screen depends on auth API integration
- Live tracking depends on location update API + WebSocket
- Regrouping depends on Overpass API + OSRM integration
- Emergency depends on emergency API + push notification integration
- Notification list depends on notification API + WebSocket

### 1.7 Folder and File Structure
```
src/services/
├── api.js              # Axios instance, base URL, interceptors
├── auth.js             # Auth token management, auth API calls
├── rides.js            # Ride-related API calls
├── participants.js     # Participant status, separation API
├── alerts.js           # Alert-related API calls
├── meetings.js         # Meeting point API calls
├── events.js           # Ride event API calls
├── emergencies.js      # Emergency report API calls
├── notifications.js    # Notification preferences, token mgmt
├── websocket.js        # WebSocket connection and events
├── routing.js          # OSRM, Overpass API calls
└── index.js            # Service exports

src/store/
├── apiSlice.js         # RTK Query or Redux API endpoints
├── websocketSlice.js   # WebSocket state management
└── index.js            # Store configuration
```

### 1.8 Database Models and Fields (Integration Perspective)
*(Frontend sees serialized API output - models described from response perspective)*

**User Serializer Output:**
```json
{
  "id": "uuid",
  "email": "alice@example.com",
  "first_name": "Alice",
  "last_name": "Smith",
  "phone": "+1234567890",
  "profile_picture_url": "https://...",
  "is_active": true
}
```

**Ride Serializer Output:**
```json
{
  "id": "uuid",
  "name": "Weekend Trip",
  "leader": "uuid",
  "start_location": {"lat": float, "lng": float},
  "dest_location": {"lat": float, "lng": float},
  "departure_time": "datetime",
  "status": "planned|active|completed|cancelled",
  "participant_count": int,
  "max_participants": 50
}
```

**Participant Serializer Output:**
```json
{
  "user_id": "uuid",
  "display_name": "Alice",
  "role": "leader|participant",
  "status": "joined|left|active",
  "separation_status": "normal|monitoring|separation_warning|high_separation|regrouped",
  "separation_distance": float
}
```

**Alert Serializer Output:**
```json
{
  "id": "uuid",
  "type": "separation_warning|high_separation|regrouped|deviation|stationary",
  "severity": "warning|high|info",
  "distance_m": float,
  "message": "string",
  "timestamp": "datetime"
}
```

### 1.9 API Specifications (Complete)

#### Authentication Endpoints

**Register**
- **API Name**: User Registration
- **HTTP Method**: POST
- **Endpoint**: `/api/auth/register/`
- **Authentication**: Public (no auth required)
- **Request Parameters**:
  - `email` (string, required, VARCHAR 255, UNIQUE)
  - `password` (string, required, min 8 chars)
  - `phone` (string, optional, VARCHAR 20, E.164 format)
  - `first_name` (string, optional)
  - `last_name` (string, optional)
- **Request Body**: JSON
- **Response (201)**:
  ```json
  {
    "success": true,
    "data": {"id": "uuid", "email": "alice@example.com", "first_name": "Alice"},
    "meta": {"timestamp": "2026-11-21T10:30:00Z", "request_id": "uuid"}
  }
  ```
- **Response (400)**:
  ```json
  {
    "success": false,
    "error": {"code": "ERR_001", "message": "Required fields missing: email", "details": {}},
    "meta": {"timestamp": "...", "request_id": "..."}
  }
  ```
- **Related Frontend Screen**: Registration Screen
- **Related Backend Module**: Auth (User model, registration view, serializer)

**Login**
- **API Name**: User Login
- **HTTP Method**: POST
- **Endpoint**: `/api/auth/login/`
- **Authentication**: Public
- **Request Body**: `{email, password}`
- **Response (200)**: `{tokens: {access, refresh}, user}`
- **Response (400)**: Invalid credentials error
- **Related Frontend Screen**: Login Screen
- **Related Backend Module**: Auth (Login view, JWT token generation)

**Refresh**
- **API Name**: Token Refresh
- **HTTP Method**: POST
- **Endpoint**: `/api/auth/refresh/`
- **Authentication**: Public
- **Request Body**: `{refresh_token}`
- **Response (200)**: `{access_token}`
- **Response (401)**: Invalid/expired refresh token
- **Related Frontend Screen**: Token refresh (interceptor)
- **Related Backend Module**: Auth (TokenRefreshView, Simple JWT)

**Profile**
- **API Name**: User Profile
- **HTTP Method**: GET/PUT
- **Endpoint**: `/api/auth/profile/`
- **Authentication**: Bearer (access token)
- **Request Body (PUT)**: `{first_name, last_name, profile_picture_url?}`
- **Response (200)**: `{user: {id, email, first_name, last_name, ...}}`
- **Response (401)**: Missing/invalid token
- **Related Frontend Screen**: Profile Screen
- **Related Backend Module**: Auth (Profile view, serializer)

**Logout**
- **API Name**: User Logout
- **HTTP Method**: POST
- **Endpoint**: `/api/auth/logout/`
- **Authentication**: Bearer (refresh token)
- **Request Body**: `{refresh_token}`
- **Response (200)**: `{success: true}`
- **Response (401)**: Invalid refresh token
- **Related Frontend Screen**: Auth persistence (AsyncStorage cleanup)
- **Related Backend Module**: Auth (Token blacklisting, view)

#### Ride Endpoints

**Create Ride**
- **API Name**: Create Ride
- **HTTP Method**: POST
- **Endpoint**: `/api/rides/`
- **Authentication**: Bearer (access token)
- **Request Body**:
  ```json
  {
    "name": "Weekend Trip",
    "start_lat": 40.7128,
    "start_lng": -74.0060,
    "dest_lat": 40.7589,
    "dest_lng": -73.9851,
    "departure_time": "2026-11-21T14:00:00Z"
  }
  ```
- **Response (201)**: Created ride object
- **Response (400)**: Validation errors (invalid lat/lng, missing fields)
- **Related Frontend Screen**: Ride Creation Screen
- **Related Backend Module**: Rides (Create view, Ride serializer, OSRM routing task)

**List Rides**
- **API Name**: List User Rides
- **HTTP Method**: GET
- **Endpoint**: `/api/rides/`
- **Authentication**: Bearer
- **Query Parameters**: `?status=active&type=created|invited`
- **Response (200)**: `{rides: [...]}`
- **Related Frontend Screen**: Ride Lobby (initial load)
- **Related Backend Module**: Rides (List view, filtering by user)

**Ride Details**
- **API Name**: Ride Details
- **HTTP Method**: GET
- **Endpoint**: `/api/rides/{id}/`
- **Authentication**: Bearer
- **URL Parameters**: `id` (UUID)
- **Response (200)**: `{ride: {...}}`
- **Response (404)**: Ride not found
- **Related Frontend Screen**: Live Map, Ride Lobby
- **Related Backend Module**: Rides (Retrieve view, participant filtering)

**Invite Participant**
- **API Name**: Invite Participant
- **HTTP Method**: POST
- **Endpoint**: `/api/rides/{id}/participants/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (ride UUID)
- **Request Body**: `{invitee_id, message?}`
- **Response (201)**: `{invitation: {token, status, expires_at}}`
- **Response (403)**: User is not ride leader
- **Response (400)**: User already participant, invitation already sent
- **Related Frontend Screen**: User Search and Invitation
- **Related Backend Module**: Invitations (Create invitation, token generation, email notification)

**Update Participant**
- **API Name**: Update Participant Role/Status
- **HTTP Method**: PUT
- **Endpoint**: `/api/rides/{id}/participants/{user_id}/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (ride UUID), `user_id` (participant UUID)
- **Request Body**: `{role: leader|participant, status: joined|left}`
- **Response (200)**: `{participant: {...}}`
- **Related Frontend Screen**: Ride Lobby (participant list update)
- **Related Backend Module**: RideParticipants (Update, role/status change)

**Start Ride**
- **API Name**: Start Ride
- **HTTP Method**: POST
- **Endpoint**: `/api/rides/{id}/start/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (ride UUID)
- **Response (200)**: `{ride: {status: active, started_at}}`
- **Related Frontend Screen**: Live Map starts
- **Related Backend Module**: Rides (Start view, status change, WebSocket broadcast)

**End Ride**
- **API Name**: End Ride
- **HTTP Method**: POST
- **Endpoint**: `/api/rides/{id}/end/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (ride UUID)
- **Response (200)**: `{ride: {status: completed, ended_at}}`
- **Related Frontend Screen**: Ride completion, history
- **Related Backend Module**: Rides (End view, archive to history, data retention)

#### Participant Endpoints

**Participant Status**
- **API Name**: Participant Status
- **HTTP Method**: GET/PUT
- **Endpoint**: `/api/participants/{ride_id}/status/`
- **Authentication**: Bearer
- **URL Parameters**: `ride_id` (ride UUID)
- **Request Body (PUT)**: `{status: joined|left|active}`
- **Response (200)**: `{participant: {role, status, separation_status, separation_distance}}`
- **Related Frontend Screen**: Ride Lobby participant list
- **Related Backend Module**: RideParticipants (Status update, separation tracking reset)

**Separation Status**
- **API Name**: Separation Status
- **HTTP Method**: GET
- **Endpoint**: `/api/participants/{ride_id}/separation-status/`
- **Authentication**: Bearer
- **URL Parameters**: `ride_id`
- **Response (200)**: `{separation_status, distance_m, severity, last_update}`
- **Related Frontend Screen**: Live tracking separation indicator
- **Related Backend Module**: Monitoring Engine (current separation calculation)

#### Alert Endpoints

**List Alerts**
- **API Name**: List Ride Alerts
- **HTTP Method**: GET
- **Endpoint**: `/api/alerts/`
- **Authentication**: Bearer
- **Query Parameters**: `?ride_id={id}`
- **Response (200)**: `{alerts: [{id, type, severity, distance_m, message, timestamp}]}`
- **Related Frontend Screen**: Separation Alerts notification list
- **Related Backend Module**: Alerts (Filter by ride, recent alerts)

**Alert Details**
- **API Name**: Alert Details
- **HTTP Method**: GET
- **Endpoint**: `/api/alerts/{id}/`
- **Authentication**: Bearer
- **URL Parameters**: `id` (alert UUID)
- **Response (200)**: `{alert: {...}}`
- **Related Frontend Screen**: Alert detail view
- **Related Backend Module**: Alerts (Retrieve specific alert)

#### Meeting Point Endpoints

**List Recommendations**
- **API Name**: List Meeting Point Recommendations
- **HTTP Method**: GET
- **Endpoint**: `/api/meeting-points/`
- **Authentication**: Bearer
- **Query Parameters**: `?ride_id={id}`
- **Response (200)**: `{recommendations: [{id, name, type, lat, lng, rank_score, max_eta_s, status, approved_by}]}`
- **Related Frontend Screen**: Regrouping Recommendations
- **Related Backend Module**: MeetingPointRecommendations (Calculation, ranking, storage)

**Approve Meeting Point**
- **API Name**: Approve Meeting Point
- **HTTP Method**: PUT
- **Endpoint**: `/api/meeting-points/{id}/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (recommendation UUID)
- **Request Body**: `{approved: true|false}`
- **Response (200)**: `{recommendation: {approved_at, approved_by}}`
- **Response (403)**: User is not ride leader
- **Related Frontend Screen**: Leader approval screen
- **Related Backend Module**: MeetingPointRecommendations (Approve, WebSocket broadcast)

**Share Meeting Point**
- **API Name**: Share Approved Meeting Point
- **HTTP Method**: POST
- **Endpoint**: `/api/meeting-points/{id}/share/`
- **Authentication**: Bearer (leader only)
- **URL Parameters**: `id` (recommendation UUID)
- **Response (200)**: `{success: true, message: "Shared via WebSocket"}`
- **Backend Action**: Broadcast `meeting_point_approved` via WebSocket
- **Related Frontend Screen**: All participants receive recommendation
- **Related Backend Module**: MeetingPointRecommendations (WebSocket broadcast event)

#### Event Endpoints

**Create Event**
- **API Name**: Create Ride Event
- **HTTP Method**: POST
- **Endpoint**: `/api/events/`
- **Authentication**: Bearer
- **Request Body**:
  ```json
  {
    "ride_id": "uuid",
    "type": "fuel|food|rest|breakdown|other",
    "lat": 40.7200,
    "lng": -74.0100,
    "note": "Optional description"
  }
  ```
- **Response (201)**: `{event: {...}}`
- **Response (400)**: Invalid type, missing required fields
- **Related Frontend Screen**: Event creation (from lobby or live map)
- **Related Backend Module**: Events (Create, WebSocket broadcast to ride participants)

**List Events**
- **API Name**: List Ride Events
- **HTTP Method**: GET
- **Endpoint**: `/api/events/{ride_id}/`
- **Authentication**: Bearer
- **URL Parameters**: `ride_id` (ride UUID)
- **Response (200)**: `{events: [{id, type, display_name, location, status, created_at}]}`
- **Related Frontend Screen**: Ride events list, event detail
- **Related Backend Module**: Events (Filter by ride, ordering by created_at)

#### Emergency Endpoints

**Create Emergency**
- **API Name**: Report Emergency/Breakdown
- **HTTP Method**: POST
- **Endpoint**: `/api/emergencies/`
- **Authentication**: Bearer
- **Request Body**:
  ```json
  {
    "ride_id": "uuid",
    "type": "breakdown|emergency",
    "severity": "low|medium|high|critical",
    "description": "Rider experiencing mechanical failure",
    "lat": 40.7150,
    "lng": -74.0080
  }
  ```
- **Response (201)**: `{emergency: {id, type, severity, status, created_at}}`
- **Response (400)**: Validation errors (invalid severity, missing fields)
- **Related Frontend Screen**: Emergency Management
- **Related Backend Module**: EmergencyReports (Create, high-priority WebSocket broadcast)

**List Emergencies**
- **API Name**: List Emergencies for Ride
- **HTTP Method**: GET
- **Endpoint**: `/api/emergencies/{ride_id}/`
- **Authentication**: Bearer
- **URL Parameters**: `ride_id` (ride UUID)
- **Response (200)**: `{emergencies: [{id, type, severity, status, description, location, created_at, acknowledged_at}]}`
- **Related Frontend Screen**: Emergency history, current emergency banner
- **Related Backend Module**: EmergencyReports (Filter by ride, status tracking)

#### Device Token Endpoint

**Register Device Token**
- **API Name**: Register Device Push Token
- **HTTP Method**: POST/PUT
- **Endpoint**: `/api/device-token/`
- **Authentication**: Bearer (access token)
- **Request Body**: `{token: "ExpoPushToken[...]", platform: "ios|android"}`
- **Response (200/201)**: `{token: {id, user, token, platform, created_at}}`
- **Response (401)**: Invalid token format
- **Related Frontend Screen**: Profile → Notification Settings
- **Related Backend Module**: DeviceTokens (Create/update, user association, cleanup)

#### Push Token Management
- **Foreground registration**: On first app launch, after permission grant
- **Background/Production**: Requires EAS build, FCM project configuration
- **Token renewal**: Refresh when token expires or user reinstalls app
- **Token revocation**: On logout or account deletion
- **Multi-platform**: Separate tokens for iOS and Android per user

### 1.10 WebSocket Integration (in Integration Doc - see Section 2)

---

## 2. WebSocket Integration

### 2.1 Module Name
WebSocket Integration

### 2.2 Module Objective
Define complete WebSocket integration specification including connection URL, authentication, event names, payloads, broadcasting rules, connection lifecycle, reconnection strategy, and error handling.

### 2.3 Module Purpose
Provide complete real-time communication specification between React Native frontend and Django Channels backend, ensuring consistent event handling and reliable location broadcasting.

### 2.4 Functional Requirements
- Establish WebSocket connection with authentication
- Join ride-specific WebSocket groups
- Broadcast location updates to ride participants
- Send/receive separation alerts
- Handle reconnection and error recovery
- Prevent duplicate messages
- Support connection lifecycle management

### 2.5 Technologies and Packages Required
- Django Channels 4.0+
- Daphne 4.0+ (ASGI server)
- Redis 6/7 (channel layer)
- WebSocket API (platform native - React Native)
- JSON message serialization

### 2.6 Dependencies on Other Modules
- Live Location Tracking (frontend)
- Intelligent Ride Monitoring (backend)
- Separation Detection (backend)
- Event and Emergency Management (backend/frontend)

### 2.7 Folder and File Structure
```
src/services/
├── websocket.js        # WebSocket connection management
├── websocket-events.js # Event type definitions and handlers
└── websocket-manager.js # Connection state management

apps/
├── rides/
│   └── routing.py      # Django Channels routing
├── monitoring/
│   └── consumers.py    # WebSocket consumers for location/alerts
└── invitations/
    └── consumers.py    # WebSocket consumers for invitation responses
```

### 2.8 WebSocket Connection URL
- **Development**: `ws://localhost:8000/ws/`
- **Production**: `wss://api.rydlink.com/ws/`

### 2.9 Authentication Method
- **Query parameters**: `?ride_id={uuid}&user_id={uuid}&token={jwt-access-token}`
- **Server-side verification**: JWT token decoded on connection
- **Invalid token**: Connection closed with code 4001, error message sent
- **Missing ride_id**: Connection rejected, "Invalid ride" error
- **User not participant**: Connection rejected, "Not a participant" error

### 2.10 Ride-Specific WebSocket Groups
- **Group name**: `ride_{ride_id}`
- **Channel layer**: Redis channel layer (`channels_redis.core.RedisChannelLayer`)
- **Join on connect**: `channel_layer.group_add(ride_{ride_id}, channel_name)`
- **Leave on disconnect**: `channel_layer.group_discard(ride_{ride_id}, channel_name)`
- **Message broadcasting**: `channel_layer.group_send(ride_{ride_id}, {type, message})`

### 2.11 Complete WebSocket Event Specifications

#### Client → Server Events

1. **`location_update`**
   - **Direction**: Client → Server
   - **Payload**:
     ```json
     {
       "type": "location_update",
       "ride_id": "uuid",
       "user_id": "uuid",
       "latitude": 40.7128,
       "longitude": -74.0060,
       "accuracy": 5.2,
       "speed": 12.5,
       "heading": 85.0,
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Frequency**: Every 5-10s foreground, 30-60s background
   - **Purpose**: Transmit GPS location for monitoring
   - **Validation**: Frontend validates accuracy ≤ 30m before sending
   - **Response**: Server broadcasts `location_broadcast`

2. **`invitation_response`**
   - **Direction**: Client → Server
   - **Payload**:
     ```json
     {
       "type": "invitation_response",
       "ride_id": "uuid",
       "token": "uuid-invitation-token",
       "accepted": true|false
     }
     ```
   - **Purpose**: Accept or reject ride invitation
   - **Validation**: Token must be valid, not expired (24h)
   - **Response**: Invitation status updated, WebSocket `invitation_accepted`/`rejected`

#### Server → Client Events

1. **`location_broadcast`**
   - **Direction**: Server → All Participants
   - **Payload**:
     ```json
     {
       "type": "location_broadcast",
       "ride_id": "uuid",
       "participants": [
         {
           "user_id": "uuid",
           "display_name": "Alice",
           "latitude": 40.7128,
           "longitude": -74.0060,
           "accuracy": 5.2,
           "role": "leader|participant",
           "status": "active|left"
         }
       ],
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Update all riders with current participant locations
   - **Frequency**: Every location update cycle (5-10s)
   - **Recipient**: All connected riders of the ride
   - **Frontend Action**: Update Map markers, participant list, separation calculation

2. **`separation_warning`**
   - **Direction**: Server → Leader + Affected Rider
   - **Payload**:
     ```json
     {
       "type": "separation_warning",
       "ride_id": "uuid",
       "user_id": "uuid",
       "distance_m": 520,
       "severity": "warning",
       "message": "Separation Warning: 520m from group",
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Alert when rider >500m for 60s continuous
   - **Recipient**: Ride leader + affected rider (user_id)
   - **Frontend Action**: Show separation warning banner, start timer, push notification
   - **Related API**: Alert created in `ride_alerts` table

3. **`high_separation`**
   - **Direction**: Server → Leader + Affected Rider
   - **Payload**:
     ```json
     {
       "type": "high_separation",
       "ride_id": "uuid",
       "user_id": "uuid",
       "distance_m": 1100,
       "severity": "high",
       "message": "High Separation Alert: 1100m from group",
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Alert when rider >1000m for 30s continuous
   - **Recipient**: Ride leader + affected rider
   - **Frontend Action**: Show high-separation critical banner, high-priority push notification
   - **Related API**: High-separation alert created in `ride_alerts`

4. **`regrouped`**
   - **Direction**: Server → All Participants
   - **Payload**:
     ```json
     {
       "type": "regrouped",
       "ride_id": "uuid",
       "user_id": "uuid",
       "distance_m": 150,
       "message": "Alice has regrouped with the group",
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Auto-resolve when rider returns within 300m for 30s
   - **Recipient**: All ride participants
   - **Frontend Action**: Dismiss alert banner, change marker to green, resume normal monitoring
   - **Timer**: 30-second confirmation timer

5. **`ride_event`**
   - **Direction**: Server → All Participants
   - **Payload**:
     ```json
     {
       "type": "ride_event",
       "ride_id": "uuid",
       "event_type": "fuel|food|rest|breakdown|other",
       "user_id": "uuid",
       "display_name": "Alice",
       "message": "Fuel stop at gas station",
       "location": {"latitude": 40.7200, "longitude": -74.0100},
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Broadcast ride events (fuel, food, rest, breakdown)
   - **Recipient**: All ride participants
   - **Frontend Action**: Add event to events list, show on map marker
   - **Related API**: `ride_events` table entry created

6. **`emergency`**
   - **Direction**: Server → All Participants, High Priority
   - **Payload**:
     ```json
     {
       "type": "emergency",
       "ride_id": "uuid",
       "user_id": "uuid",
       "display_name": "Alice",
       "severity": "high|critical",
       "description": "Rider experiencing mechanical failure",
       "location": {"latitude": 40.7150, "longitude": -74.0080},
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: High-priority emergency/breakdown report
   - **Recipient**: All ride participants + ride leader
   - **Frontend Action**: Full-screen emergency banner, push notification, acknowledge/resolve
   - **Related API**: `emergency_reports` table entry created

7. **`invitation_response`** (Server → Client, status update)
   - **Direction**: Server → Client (after acceptance/rejection)
   - **Payload**:
     ```json
     {
       "type": "invitation_response",
       "ride_id": "uuid",
       "token": "uuid",
       "accepted": true|false,
       "status": "accepted|rejected|expired"
     }
     ```
   - **Purpose**: Confirm invitation response status
   - **Recipient**: Invitee (client who sent the response)
   - **Frontend Action**: Close invitation screen, show confirmation

8. **`meeting_point_approved`**
   - **Direction**: Server → All Participants
   - **Payload**:
     ```json
     {
       "type": "meeting_point_approved",
       "ride_id": "uuid",
       "meeting_point": {
         "id": "uuid",
         "name": "Coffee Shop",
         "type": "restaurant",
         "lat": 40.7200,
         "lng": -74.0050,
         "approved_by": "leader-uuid"
       },
       "timestamp": "2026-11-21T10:30:00Z"
     }
     ```
   - **Purpose**: Broadcast approved meeting point to all participants
   - **Recipient**: All ride participants
   - **Frontend Action**: Show meeting point on map, navigate there, update ETA

9. **`error`**
   - **Direction**: Server → Client
   - **Payload**:
     ```json
     {
       "type": "error",
       "message": "Invalid ride or Not a participant"
     }
     ```
   - **Purpose**: WebSocket connection or message error
   - **Recipient**: Connecting or messaging client
   - **Frontend Action**: Show error banner, disconnect/reconnect, redirect

### 2.12 Connection Lifecycle
1. **Client initiates**: `new WebSocket(wsUrl + "?ride_id=&user_id=&token=")`
2. **Connection open**: `onopen` handler, join ride group via channel layer
3. **Authentication**: Server verifies JWT token, adds to `ride_{ride_id}` group
4. **Welcome message**: Server sends connection confirmation with participant role
5. **Client sends**: `location_update` messages at configured frequency
6. **Server processes**: Validates, stores, evaluates separation, broadcasts
7. **Server broadcasts**: `location_broadcast`, alerts, events, emergencies
8. **Client receives**: Processes messages, updates state/UI
9. **Connection close**: `onclose` handler, attempt reconnection
10. **Client disconnects**: Participant marked as `left`, ride ends

### 2.13 Reconnection Strategy
- **Automatic**: Attempt reconnection after 3-second delay on close
- **Exponential backoff**: 3s → 5s → 10s → 15s (max), then give up
- **State preservation**: Last known locations preserved in Redux/store
- **Group rejoin**: On reconnection, client rejoins `ride_{ride_id}` group
- **Location resync**: Request current participant locations from API
- **Timer preservation**: Separation timers persisted in backend, restored on reconnect
- **Max retry attempts**: 5 attempts, then show manual reconnect UI

### 2.14 Message Validation
- **Schema validation**: Each message type has expected fields
- **Type checking**: Verify `type` field matches expected value
- **Field presence**: Required fields must be present and non-empty
- **Range validation**: lat ∈ [-90, 90], lng ∈ [-180, 180], accuracy ≥ 0
- **Duplicate prevention**: Same `location_update` not processed twice within 1 second
- **Message ordering**: Preserve order within each WebSocket connection

### 2.15 Error Handling
- **Authentication failure**: Close connection, code 4001, show "Auth failed, reconnect"
- **Invalid ride_id**: Server sends `{"type": "error", "message": "Invalid ride"}`, then close
- **Participant not in ride**: Server sends `{"type": "error", "message": "Not a participant"}`, then close
- **Rate limiting**: Server throttles `location_update` (>10/s), client respects throttle
- **Network disconnect**: Client shows stale-location indicator, resumes on reconnection
- **Server error**: Display error banner, offer "Retry" option, log to console
- **Message parse error**: Skip malformed message, continue with next

### 2.16 Broadcasting Rules
- **Ride isolation**: Each ride's participants only receive ride-specific messages
- **Cross-ride messages**: Not sent (different `ride_id` groups)
- **Anonymous users**: Cannot connect to ride WebSocket without valid token
- **Token validation**: JWT must be valid and user must be participant of ride
- **Group join**: Server checks `ride_participants` table before adding to group
- **Group leave**: On disconnect, remove from group, mark status = left

### 2.17 Expected Output Examples

**WebSocket Connection Success:**
```
WebSocket readyState: 1 (OPEN)
Welcome message: {"type": "connection_established", "role": "leader", "ride_id": "uuid"}
```

**Location Broadcast Received:**
```json
{
  "type": "location_broadcast",
  "ride_id": "uuid",
  "participants": [...],
  "timestamp": "2026-11-21T10:30:00Z"
}
```

**Error Response:**
```json
{
  "type": "error",
  "message": "Invalid token or user not participant"
}
```

### 2.18 Testing Requirements
- Test connection establishment with valid token
- Test connection rejection with invalid token
- Test ride group isolation (messages only within ride)
- Test location broadcast delivery to all participants
- Test reconnection after network interruption
- Test duplicate message prevention
- Test WebSocket error handling
- Test Redis channel layer broadcasting

---

## 3. PostgreSQL and PostGIS Integration

### 3.1 Module Name
PostgreSQL and PostGIS Integration

### 3.2 Module Objective
Define the database integration between Django ORM and PostgreSQL with PostGIS extension, including geographical fields, spatial queries, distance calculations, spatial indexes, database migrations, transaction handling, and data validation.

### 3.3 Module Purpose
Provide the complete database layer specification for RydLink, ensuring reliable storage, querying, and manipulation of geospatial data for location tracking and distance calculations.

### 3.4 Functional Requirements
- Configure PostgreSQL with PostGIS extension
- Define Django ORM models with GIS fields
- Implement spatial queries for distance calculations
- Create spatial indexes for performance
- Handle database migrations
- Manage transaction handling
- Validate geospatial data

### 3.5 Technologies and Packages Required
- PostgreSQL 14/15
- PostGIS 3.x
- psycopg 3.x
- Django with django.contrib.gis
- pgcrypto extension
- GistIndex for spatial indexes

### 3.6 Dependencies on Other Modules
- All backend modules depend on database models
- Location tracking depends on PostGIS geography fields
- Separation detection depends on spatial distance queries
- Route management depends on PostGIS LineString geometry

### 3.7 Folder and File Structure
```
apps/
├── auth/
│   └── models.py         # User model
├── rides/
│   └── models.py         # Ride model
├── participants/
│   └── models.py         # RideParticipant model
├── monitoring/
│   └── models.py         # LocationUpdate model (PostGIS Point)
├── routes/
│   └── models.py         # RideRoute model (PostGIS LineString)
├── alerts/
│   └── models.py         # RideAlert model
├── events/
│   └── models.py         # RideEvent model
├── emergencies/
│   └── models.py         # EmergencyReport model
├── notifications/
│   └── models.py         # DeviceToken, Notification models
└── history/
    └── models.py         # Ride history models

rydlink/
├── settings.py           # Database configuration
└── urls.py
```

### 3.8 Database Configuration (Django settings.py)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': os.environ.get('DB_NAME', 'rydlink'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD', ''),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}
```

### 3.9 PostGIS Extensions Setup
```sql
-- Enable pgcrypto for UUID generation
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Enable PostGIS for geographic operations
CREATE EXTENSION IF NOT EXISTS postgis;

-- Verify installation
SELECT PostGIS_Version();
```

### 3.10 Geographical Fields and Models

#### LocationUpdate Model (PostGIS Point)
```python
from django.contrib.gis.db import models

class LocationUpdate(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    ride = models.ForeignKey('rides.Ride', on_delete=models.CASCADE)
    user = models.ForeignKey('users.User', on_delete=models.CASCADE)
    location = models.PointField(geography=True, srid=4326)
    accuracy = models.FloatField(null=True)  # metres
    speed = models.FloatField(null=True)     # m/s
    heading = models.FloatField(null=True)   # degrees
    recorded_at = models.DateTimeField()
    server_received_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        indexes = [
            models.Index(fields=['ride', 'user', '-recorded_at']),
            models.GisIndex(fields=['location'], name='lu_location_gist'),
        ]
```

#### RideRoute Model (PostGIS LineString)
```python
class RideRoute(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    ride = models.OneToOneField('rides.Ride', on_delete=models.CASCADE)
    geometry = models.LineStringField(srid=4326)
    distance_m = models.FloatField()
    duration_s = models.IntegerField()
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        indexes = [
            models.GisIndex(fields=['geometry'], name='rr_geometry_gist'),
        ]
```

### 3.11 Spatial Queries

#### Distance Calculation (Participant to Group Reference)
```python
from django.contrib.gis.geos import Point
from django.contrib.gis.measure import Distance

# Calculate distance between participant and group reference
participant_location = Point(lng, lat, srid=4326)
group_reference = Point(group_lng, group_lat, srid=4326)

distance_m = participant_location.distance(group_reference)  # metres

# Alternative: Use ST_Distance via raw SQL for performance
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("""
        SELECT ST_Distance(
            ST_SetSRID(ST_MakePoint(%s, %s), 4326)::geography,
            ST_SetSRID(ST_MakePoint(%s, %s), 4326)::geography
        ) as distance_m
    """, [lng, lat, group_lng, group_lat])
    result = cursor.fetchone()
    distance_m = result[0]
```

#### Find Nearby Participants (Within Radius)
```python
# Find all participants within 1km of a reference point
nearby_updates = LocationUpdate.objects.filter(
    ride=ride,
    location__distance_lte=(Point(lng, lat, srid=4326), Distance(m=1000))
)
```

#### Route Deviation Detection
```python
# Calculate perpendicular distance from point to route LineString
from django.contrib.gis.db.models import Func

deviation = LocationUpdate.objects.annotate(
    deviation=Func(
        models.F('location'),
        models.F('ride__route__geometry'),
        function='ST_Distance',
        output_field=models.FloatField()
    )
).filter(deviation__gt=200)  # More than 200m off route
```

### 3.12 Spatial Indexes

| Table | Index Name | Columns | Type | Purpose |
|-------|-----------|---------|------|---------|
| `location_updates` | `lu_location_gist` | `location` | GiST | Spatial queries on participant locations |
| `ride_routes` | `rr_geometry_gist` | `geometry` | GiST | Spatial queries on route geometry |

### 3.13 Distance Calculations

#### Haversine Formula (Python fallback)
```python
import math

def haversine(lat1, lng1, lat2, lng2):
    """Calculate distance between two points in metres using Haversine formula."""
    R = 6371000  # Earth radius in metres
    
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lat = math.radians(lat2 - lat1)
    delta_lng = math.radians(lng2 - lng1)
    
    a = math.sin(delta_lat / 2)**2 + math.cos(lat1_rad) * math.cos(lat2_rad) * math.sin(delta_lng / 2)**2
    c = 2 * math.asin(math.sqrt(a))
    
    return R * c
```

#### PostGIS ST_Distance (Production)
```sql
-- Geography type returns metres directly
SELECT ST_Distance(
    ST_SetSRID(ST_MakePoint(-74.0060, 40.7128), 4326)::geography,
    ST_SetSRID(ST_MakePoint(-74.0080, 40.7150), 4326)::geography
) as distance_m;
```

### 3.14 Database Migrations

```python
# Migration to enable PostGIS extensions
from django.db import migrations
from django.contrib.postgres.operations import CreateExtension

class Migration(migrations.Migration):
    dependencies = []
    
    operations = [
        CreateExtension('pgcrypto'),
        CreateExtension('postgis'),
    ]
```

**Migration Rules:**
1. Enable pgcrypto extension first (UUID generation)
2. Enable postgis extension (geographic operations)
3. Initial migration creates all 12 tables
4. Spatial indexes via `django.contrib.gis.db.models.indexes.GistIndex`
5. No destructive migrations after production; use additive changes

### 3.15 Transaction Handling

```python
from django.db import transaction

with transaction.atomic():
    # Create ride
    ride = Ride.objects.create(...)
    
    # Add participants
    for user_id in participant_ids:
        RideParticipant.objects.create(ride=ride, user_id=user_id)
    
    # Create route
    RideRoute.objects.create(ride=ride, geometry=...)
```

**Transaction Rules:**
- All ride creation operations wrapped in atomic transaction
- WebSocket events sent only after successful transaction commit
- Failed transactions roll back all changes
- No partial state visible to other transactions

### 3.16 Data Validation

#### Geospatial Validation
- Latitude: -90 to 90 degrees
- Longitude: -180 to 180 degrees
- SRID: 4326 (WGS84)
- Accuracy: Non-negative metres
- Timestamp: Within reasonable range

#### Model Validation (Django CheckConstraints)
| Table | Constraint | Condition |
|-------|-----------|-----------|
| rides | rides_status_check | status IN ('planned','active','completed','cancelled') |
| rides | rides_start_lat_check | start_lat BETWEEN -90 AND 90 |
| rides | rides_start_lng_check | start_lng BETWEEN -180 AND 180 |
| ride_participants | rp_role_check | role IN ('leader','participant') |
| ride_alerts | ra_type_check | type IN ('separation_warning','high_separation','regrouped','deviation','stationary') |

### 3.17 Frontend–Backend Interaction

#### Database Query Flow
1. Frontend requests ride data via REST API
2. Django ORM queries PostgreSQL with PostGIS
3. Spatial queries executed with PostGIS functions
4. Results serialized via Django REST Framework serializers
5. JSON response sent to frontend
6. Frontend deserializes and displays data

#### Location Update Storage Flow
1. Frontend sends location via WebSocket
2. Django Channels consumer receives message
3. LocationUpdate model creates PostGIS geography point
4. Point stored with SRID 4326
5. Spatial index updated automatically
6. Separation detection queries latest locations using spatial queries

### 3.18 Testing Requirements

#### Unit Tests
- Test PostGIS extension enabled
- Test spatial index creation
- Test distance calculation accuracy
- Test geography vs geometry field behavior
- Test SRID handling (4326)
- Test migration application

#### Integration Tests
- Test full ride creation with route storage
- Test location update storage and retrieval
- Test spatial query performance
- Test concurrent location updates
- Test database transaction rollback on error

---

## 4. Redis Integration

### 4.1 Module Name
Redis Integration

### 4.2 Module Objective
Define Redis installation, configuration, Django Channels channel layer integration, message broadcasting, ride-specific groups, connection management, and development/production configuration.

### 4.3 Module Purpose
Provide the complete Redis layer specification for Django Channels, ensuring reliable real-time message broadcasting and WebSocket communication across multiple server processes.

### 4.4 Functional Requirements
- Install and configure Redis server
- Configure Django Channels channel layer
- Support message broadcasting to ride-specific groups
- Manage WebSocket connections across processes
- Configure development vs production settings
- Handle connection failures gracefully

### 4.5 Technologies and Packages Required
- Redis 6/7
- channels-redis (Django Channels Redis backend)
- Django Channels 4.0+

### 4.6 Dependencies on Other Modules
- WebSocket Communication (backend)
- Live Location Tracking (frontend)
- Separation Detection (backend)
- All real-time features

### 4.7 Folder and File Structure
```
config/
├── celery.py           # Celery configuration with Redis broker
rydlink/
├── settings.py         # Redis channel layer configuration
apps/
└── monitoring/
    └── consumers.py    # WebSocket consumers using channel layer
```

### 4.8 Redis Installation and Configuration

#### Development Configuration
```python
# settings.py (development)
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('localhost', 6379)],
            'capacity': 100,
            'expiry': 60,
        },
    },
}
```

#### Production Configuration
```python
# settings.py (production)
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [os.environ.get('REDIS_URL', 'redis://redis:6379')],
            'capacity': 1000,
            'expiry': 300,
            'channel_capacity': {
                'ride_*': 500,
            },
        },
    },
}
```

#### Redis Configuration File (redis.conf)
```
# Redis configuration for RydLink
bind 0.0.0.0
port 6379
maxmemory 256mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
```

### 4.9 Django Channels Channel Layer

```python
# Channel layer configuration
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('localhost', 6379)],
        },
    },
}
```

**Channel Group Naming:**
- Ride group: `ride_{ride_id}`
- Pattern: All channel names follow consistent naming convention

**Message Broadcasting:**
```python
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

channel_layer = get_channel_layer()

# Broadcast to all participants in a ride
async_to_sync(channel_layer.group_send)(
    f"ride_{ride_id}",
    {
        "type": "location_broadcast",
        "ride_id": ride_id,
        "participants": [...],
        "timestamp": "..."
    }
)
```

### 4.10 Message Broadcasting

#### Group Broadcasting Rules
- **Ride isolation**: Messages only broadcast to `ride_{ride_id}` group
- **All participants**: Every connected client in group receives message
- **Leader-only**: Message tagged with leader filter for selective delivery
- **Specific participant**: Direct message to specific channel (rarely used)

#### Broadcast Examples
```python
# Location broadcast to all ride participants
await channel_layer.group_send(
    f"ride_{ride_id}",
    {
        "type": "location_broadcast",
        "ride_id": str(ride_id),
        "participants": participant_list,
        "timestamp": timestamp
    }
)

# Separation alert to leader + affected rider
await channel_layer.group_send(
    f"ride_{ride_id}",
    {
        "type": "separation_warning",
        "ride_id": str(ride_id),
        "user_id": str(user_id),
        "distance_m": distance,
        "severity": "warning",
        "message": alert_message,
        "timestamp": timestamp
    }
)
```

### 4.11 Connection Management

#### Connection Tracking
- Each WebSocket connection creates a channel name
- Channel added to `ride_{ride_id}` group on connect
- Channel removed from group on disconnect
- Multiple connections per user possible (different devices)

#### Connection Limits
- Maximum connections per ride: Configurable (default 100)
- Maximum rides per user: 1 active ride
- Connection timeout: 30 seconds inactivity

### 4.12 Development vs Production Configuration

| Setting | Development | Production |
|---------|------------|------------|
| Redis host | localhost | Redis service URL |
| Redis port | 6379 | 6379 (via service) |
| Channel capacity | 100 | 1000 |
| Message expiry | 60 seconds | 300 seconds |
| Password required | No | Yes (REDIS_URL with password) |
| TLS/SSL | No | Yes (rediss://) |

### 4.13 Error Handling

| Error | Handling |
|-------|----------|
| Redis unavailable | Fallback to in-memory channel layer (single worker only) |
| Connection refused | Retry with exponential backoff |
| Message too large | Reject with error, log warning |
| Group full | Reject new connections, notify admin |

### 4.14 Frontend–Backend Interaction

#### Redis Channel Flow
1. Frontend connects to WebSocket with ride_id
2. Backend consumer adds channel to `ride_{ride_id}` Redis group
3. Frontend sends location_update WebSocket message
4. Backend consumer processes message, broadcasts to group via Redis
5. All participants in group receive location_broadcast
6. Redis persists messages temporarily for delivery guarantees

#### Data Flow Example
```
Frontend → WebSocket → Django Consumer → Redis Channel Layer → Group Members → Frontend
```

### 4.15 Testing Requirements

#### Unit Tests
- Test Redis connection establishment
- Test channel group add/remove
- Test message broadcasting to group
- Test channel layer configuration
- Test message expiry and cleanup

#### Integration Tests
- Test multi-process WebSocket broadcasting
- Test Redis failure fallback
- Test concurrent connections per ride
- Test message delivery under load
- Test connection cleanup on disconnect

---

## 5. MapLibre and OpenStreetMap Integration

### 5.1 Module Name
MapLibre and OpenStreetMap Integration

### 5.2 Module Objective
Define map rendering, tile configuration, attribution, participant markers, route rendering, geographical coordinates, map permissions, tile provider requirements, and map service limitations.

### 5.3 Module Purpose
Provide the complete map integration specification for RydLink, ensuring reliable map display with OpenStreetMap tiles and proper attribution.

### 5.4 Functional Requirements
- Render MapLibre map with OpenStreetMap-compatible tiles
- Display participant markers on map
- Render route polylines
- Handle geographical coordinates
- Manage map permissions
- Configure tile provider requirements
- Handle map service limitations

### 5.5 Technologies and Packages Required
- MapLibre React Native Latest
- OpenStreetMap tile service
- @maplibre/react-native-maplibre-gl
- React Native StyleSheet

### 5.6 Dependencies on Other Modules
- Live Map Integration (frontend)
- Route Visualization (frontend)
- Real-Time Participant Tracking (frontend)
- Separation Alert Interface (frontend)

### 5.7 Folder and File Structure
```
src/components/map/
├── MapComponent.js       # MapLibre MapView component
├── ParticipantMarker.js  # Custom marker component
├── RoutePolyline.js      # Route polyline renderer
└── MarkerCluster.js      # Marker clustering for multiple participants

src/constants/
├── map-tiles.js          # Tile source configurations
└── themes.js             # Map styling constants
```

### 5.8 Map Rendering

#### MapLibre Map Initialization
```javascript
import MapboxGL from '@maplibre/react-native-maplibre-gl';

// Configure MapLibre with OpenStreetMap tiles
MapboxGL.setAccessToken(''); // Not required for OSM

const MAP_STYLE = {
  version: 8,
  sources: {
    'osm': {
      type: 'raster',
      tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
      tileSize: 256,
      attribution: '© OpenStreetMap contributors'
    }
  },
  layers: [{
    id: 'base-layer',
    type: 'raster',
    source: 'osm',
    minzoom: 0,
    maxzoom: 22
  }]
};
```

#### Map Component Props
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| style | Object | Yes | Map style configuration |
| initialRegion | Object | Yes | Initial camera position |
| showsUserLocation | Boolean | No | Show user location dot |
| styleURL | String | No | Remote style URL |
| zoomEnabled | Boolean | No | Allow zoom gestures |
| rotateEnabled | Boolean | No | Allow rotate gestures |

### 5.9 Map Tile Configuration

#### OpenStreetMap Tile Source
- **URL**: `https://tile.openstreetmap.org/{z}/{x}/{y}.png`
- **Attribution**: `© OpenStreetMap contributors`
- **Max Zoom**: 19
- **Tile Size**: 256x256 pixels
- **Rate Limit**: No strict limit for development, attribution required

#### Alternative Tile Providers
| Provider | URL Pattern | Cost | Attribution | Notes |
|----------|------------|------|-------------|-------|
| OpenStreetMap | `https://tile.openstreetmap.org/{z}/{x}/{y}.png` | Free | Required | Public, rate-limited |
| MapTiler | `https://api.maptiler.com/maps/streets/{z}/{x}/{y}.png?key={API_KEY}` | Free/Paid | Required | API key required |
| Mapbox | `https://api.mapbox.com/styles/v1/mapbox/streets-v11/tiles/{z}/{x}/{y}?access_token={TOKEN}` | Paid | Required | Access token required |

**Important**: MapLibre itself does NOT provide map tiles. A tile source must be configured separately.

### 5.10 Map Attribution

#### Required Attribution Text
```
© OpenStreetMap contributors
```

#### Attribution Display
- Bottom-right corner of map
- Clickable link to OpenStreetMap copyright page
- Always visible when using OSM tiles
- Cannot be removed or hidden

### 5.11 Participant Markers

#### Marker Component
```javascript
// ParticipantMarker.js
const ParticipantMarker = ({ coordinate, title, role, status }) => {
  const icon = role === 'leader' ? LEADER_ICON : PARTICIPANT_ICON;
  const color = SEPARATION_COLORS[status] || '#4A90E2';
  
  return (
    <Marker coordinate={coordinate} title={title} image={icon} />
  );
};
```

#### Marker Types
| Type | Icon | Color | Description |
|------|------|-------|-------------|
| Ride Leader | Crown | #4A90E2 | Leader of the ride |
| Participant | Circle | #FF6B6B | Regular participant |
| Stale | Faded circle | #999999 | Location >2 min old |
| Destination | Target | #FFD700 | Ride destination |
| Start | Home pin | #4CAF50 | Ride starting point |

### 5.12 Route Rendering

#### Polyline Configuration
```javascript
// RoutePolyline.js
const ROUTE_STYLE = {
  strokeColor: '#FF6B6B',
  strokeWidth: 4,
  strokeDasharray: null, // Solid line
  lineCap: 'round',
  lineJoin: 'round',
};
```

#### Route Display Steps
1. Retrieve route geometry from OSRM (encoded polyline)
2. Decode polyline to coordinate array
3. Render as MapLibre polyline
4. Style with route color and width
5. Update on route changes

### 5.13 Geographical Coordinates

#### Coordinate Format
- **Format**: `{latitude: float, longitude: float}`
- **Range**: lat ∈ [-90, 90], lng ∈ [-180, 180]
- **Precision**: 6 decimal places (approx 0.1m precision)
- **SRID**: 4326 (WGS84)

#### Coordinate Conversion
```javascript
// Convert backend format to MapLibre format
const maplibreCoord = {
  latitude: backendLat,
  longitude: backendLng,
};
```

### 5.14 Map Permissions

#### Location Permission
- Required for user location display
- iOS: NSLocationWhenInUseUsageDescription
- Android: ACCESS_FINE_LOCATION

#### Notification Permission
- Required for push notifications
- iOS: NSUserNotificationsUsageDescription
- Android: POST_NOTIFICATIONS (Android 13+)

### 5.15 Tile Provider Requirements

| Requirement | Description |
|-------------|-------------|
| Attribution | Must display OSM attribution |
| Rate limits | Respect provider rate limits |
| Availability | No guaranteed SLA for public tiles |
| Caching | May cache tiles for offline use (check license) |
| HTTPS | Required for production |

### 5.16 Map Service Limitations

| Limitation | Description | Mitigation |
|------------|-------------|------------|
| OSM public tiles | Rate-limited, no SLA | Use paid provider for production |
| Public OSRM | Strict rate limits | Self-host OSRM for production |
| Public Overpass | Strict rate limits | Self-host Overpass for production |
| Public Nominatim | 1 req/sec max | Self-host or use paid geocoding |
| Network dependency | Map tiles require internet | Cache tiles, show offline state |

### 5.17 Frontend–Backend Interaction

#### Map Data Flow
1. Frontend initializes MapLibre with OSM tile source
2. Frontend requests route from backend API
3. Backend calculates route via OSRM
4. Backend returns route geometry (LineString)
5. Frontend renders route polyline on map
6. Frontend receives WebSocket location updates
7. Frontend updates participant markers in real-time

#### Attribution Flow
```
MapLibre renders OSM tiles → Attribution displayed in corner → User clicks attribution → OSM copyright page opens
```

### 5.18 Testing Requirements

#### Unit Tests
- Test map initialization
- Test tile source configuration
- Test marker rendering
- Test polyline rendering
- Test coordinate conversion

#### Integration Tests
- Test map loads with OSM tiles
- Test route displays correctly
- Test participant markers update
- Test map permissions request
- Test offline map behavior

---

## 6. OSRM Routing Integration

### 6.1 Module Name
OSRM Routing Integration

### 6.2 Module Objective
Define route request, coordinate format, route geometry, distance, estimated duration, API response handling, routing failures, and alternative routes.

### 6.3 Module Purpose
Provide the complete OSRM routing specification for RydLink, ensuring reliable route calculation and travel time estimation.

### 6.4 Functional Requirements
- Request route between two coordinates
- Handle coordinate format (longitude, latitude)
- Parse route geometry (encoded polyline)
- Extract distance and duration
- Handle API response correctly
- Handle routing failures gracefully
- Support alternative routes (optional)

### 6.5 Technologies and Packages Required
- OSRM (self-hosted for production)
- requests (Python HTTP client)
- axios (frontend HTTP client)
- Polyline decoder

### 6.6 Dependencies on Other Modules
- Route Management (backend)
- Route Visualization (frontend)
- Intelligent Regrouping (backend)
- GPS Location Tracking (frontend/backend)

### 6.7 Folder and File Structure
```
src/services/
├── routing.js         # OSRM API calls from frontend

apps/
├── routes/
│   └── services.py    # OSRM integration service
│   └── utils.py       # Polyline decoding utilities
```

### 6.8 Route Request

#### OSRM API Endpoint
```
GET /route/v1/driving/{start_lng},{start_lat};{dest_lng},{dest_lat}?overview=full
```

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| start_lng | float | Yes | Starting longitude |
| start_lat | float | Yes | Starting latitude |
| dest_lng | float | Yes | Destination longitude |
| dest_lat | float | Yes | Destination latitude |
| overview | string | No | 'full' for geometry, 'simplified' for summary |
| geometries | string | No | 'polyline' for encoded geometry |
| alternatives | boolean | No | Return alternative routes |

#### Coordinate Format
- **Order**: Longitude, Latitude (lng, lat) - OSRM standard
- **Format**: Decimal degrees
- **Example**: `-74.0060,40.7128;-73.9851,40.7589`

### 6.9 Route Geometry

#### Encoded Polyline
- **Format**: Google Polyline algorithm
- **Decoding**: Use `polyline` library (Python) or `react-native-polyline` (frontend)
- **Output**: Array of `{lat, lng}` coordinate pairs

#### Example Route Geometry
```json
{
  "type": "LineString",
  "coordinates": [
    [-74.0060, 40.7128],
    [-74.0050, 40.7130],
    [-74.0040, 40.7135],
    [-73.9851, 40.7589]
  ]
}
```

### 6.10 Distance and Duration

#### Route Summary Fields
| Field | Type | Unit | Description |
|-------|------|------|-------------|
| distance | float | metres | Total route distance |
| duration | float | seconds | Estimated travel time |
| geometry | string | encoded | Polyline-encoded route |

#### Example API Response
```json
{
  "code": "Ok",
  "routes": [{
    "distance": 4500.5,
    "duration": 600.0,
    "geometry": "abc{~lxyz",
    "legs": [{
      "distance": 4500.5,
      "duration": 600.0,
      "steps": []
    }]
  }],
  "waypoints": [...]
}
```

### 6.11 API Response Handling

#### Python (Backend)
```python
import requests

def calculate_route(start_lat, start_lng, dest_lat, dest_lng):
    url = f"https://router.project-osrm.org/route/v1/driving/{start_lng},{start_lat};{dest_lng},{dest_lat}?overview=full&geometries=polyline"
    
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        data = response.json()
        
        if data['code'] != 'Ok':
            return None
        
        route = data['routes'][0]
        return {
            'distance_m': route['distance'],
            'duration_s': route['duration'],
            'geometry': route['geometry'],  # Encoded polyline
        }
    except requests.RequestException as e:
        # Log error and return None
        return None
```

#### JavaScript (Frontend)
```javascript
import axios from 'axios';

export const calculateRoute = async (startLat, startLng, destLat, destLng) => {
  try {
    const response = await axios.get(
      `https://router.project-osrm.org/route/v1/driving/${startLng},${startLat};${destLng},${destLat}?overview=full&geometries=polyline`
    );
    
    if (response.data.code !== 'Ok') {
      throw new Error('Route calculation failed');
    }
    
    const route = response.data.routes[0];
    return {
      distance: route.distance,      // metres
      duration: route.duration,      // seconds
      geometry: route.geometry,      // Encoded polyline
    };
  } catch (error) {
    console.error('OSRM error:', error);
    return null;
  }
};
```

### 6.12 Routing Failures

| Failure | Cause | Handling |
|---------|-------|----------|
| No route found | No road network between points | Show "No route available" |
| API timeout | OSRM server slow | Retry with timeout, show error |
| Rate limited | Too many requests | Exponential backoff |
| Invalid coordinates | Coordinates out of range | Validate before request |
| Service unavailable | OSRM down | Show cached route or error |

### 6.13 Alternative Routes (Optional)

#### When to Use
- Multiple route options for ride planning
- Avoid traffic or road closures
- Provide scenic vs. fastest route

#### Implementation
```python
# Request alternatives
url = f"https://router.project-osrm.org/route/v1/driving/{lng1},{lat1};{lng2},{lat2}?overview=full&alternatives=true"

# Parse multiple routes
routes = data['routes']  # List of route objects
best_route = min(routes, key=lambda r: r['duration'])  # Fastest
```

### 6.14 Frontend–Backend Interaction

#### Route Calculation Flow
1. Frontend sends start/dest coordinates to backend API
2. Backend calls OSRM API with coordinates
3. OSRM returns route geometry, distance, duration
4. Backend stores route in RideRoute table (PostGIS LineString)
5. Backend returns route data to frontend
6. Frontend decodes polyline and renders on MapLibre

#### Data Flow Example
```
Frontend → POST /api/rides/ → Backend → OSRM API → Backend → RideRoute (PostGIS) → Frontend → MapLibre polyline
```

### 6.15 Testing Requirements

#### Unit Tests
- Test OSRM API request construction
- Test polyline decoding
- Test distance/duration extraction
- Test error handling for invalid responses

#### Integration Tests
- Test full route calculation flow
- Test route storage in database
- Test route retrieval for frontend
- Test OSRM service unavailability
- Test alternative routes (if implemented)

---

## 7. Overpass API Integration

### 7.1 Module Name
Overpass API Integration

### 7.2 Module Objective
Define nearby POI discovery, fuel stations, restaurants, parking locations, search radius, query structure, result processing, duplicate filtering, API limitations, and failure handling.

### 7.3 Module Purpose
Provide the complete Overpass API specification for RydLink, ensuring reliable nearby point-of-interest discovery for regrouping recommendations.

### 7.4 Functional Requirements
- Discover nearby POIs using Overpass API
- Filter by type: fuel stations, restaurants, parking areas
- Configure search radius
- Construct Overpass QL queries
- Process API results
- Filter duplicate results
- Handle API limitations and failures

### 7.5 Technologies and Packages Required
- Overpass API (self-hosted for production)
- requests (Python HTTP client)
- axios (frontend HTTP client)

### 7.6 Dependencies on Other Modules
- Intelligent Regrouping (backend)
- Regrouping Recommendation Interface (frontend)
- Meeting Point Discovery (backend)

### 7.7 Folder and File Structure
```
src/services/
├── routing.js         # Overpass API calls from frontend

apps/
├── monitoring/
│   └── services.py    # Overpass integration service
│   └── utils.py       # POI filtering and deduplication
```

### 7.8 Nearby POI Discovery

#### Overpass API Endpoint
```
POST https://overpass-api.de/api/interpreter
```

#### Query Structure
```overpass
[out:json][timeout:25];
(
  node["amenity"="fuel"](around:5000,40.7128,-74.0060);
  node["amenity"="restaurant"](around:5000,40.7128,-74.0060);
  node["amenity"="cafe"](around:5000,40.7128,-74.0060);
  node["parking"="yes"](around:5000,40.7128,-74.0060);
);
out center;
```

#### Search Radius
- **Default**: 5 km (configurable)
- **Minimum**: 1 km
- **Maximum**: 50 km (performance consideration)
- **Unit**: metres

### 7.9 POI Types

| Type | Overpass Query Tag | Description |
|------|-------------------|-------------|
| Fuel Station | `amenity=fuel` | Gas stations for refueling |
| Restaurant | `amenity=restaurant` | Dining establishments |
| Cafe | `amenity=cafe` | Coffee shops, light dining |
| Parking | `parking=yes` | Parking lots, garages |

### 7.10 Result Processing

#### Python Example
```python
import requests

def find_nearby_pois(lat, lng, radius_m=5000):
    query = f"""
    [out:json][timeout:25];
    (
      node["amenity"="fuel"](around:{radius_m},{lat},{lng});
      node["amenity"="restaurant"](around:{radius_m},{lat},{lng});
      node["amenity"="cafe"](around:{radius_m},{lat},{lng});
      node["parking"="yes"](around:{radius_m},{lat},{lng});
    );
    out center;
    """
    
    try:
        response = requests.post(
            'https://overpass-api.de/api/interpreter',
            data={'data': query},
            timeout=30
        )
        response.raise_for_status()
        data = response.json()
        
        pois = []
        for element in data['elements']:
            pois.append({
                'id': element['id'],
                'lat': element.get('lat', element.get('center', {}).get('lat')),
                'lng': element.get('lon', element.get('center', {}).get('lon')),
                'type': element['tags'].get('amenity', element['tags'].get('parking')),
                'name': element['tags'].get('name', 'Unknown'),
            })
        
        # Remove duplicates
        pois = deduplicate_pois(pois)
        return pois
    except requests.RequestException as e:
        return []
```

#### Deduplication Logic
```python
def deduplicate_pois(pois):
    """Remove duplicate POIs based on coordinates."""
    seen = set()
    unique = []
    
    for poi in pois:
        key = (round(poi['lat'], 6), round(poi['lng'], 6))
        if key not in seen:
            seen.add(key)
            unique.append(poi)
    
    return unique
```

### 7.11 Duplicate Filtering

| Method | Description |
|--------|-------------|
| Coordinate-based | Round lat/lng to 6 decimal places (~0.1m) |
| Name-based | Match POI names (case-insensitive) |
| ID-based | Use Overpass element ID as unique identifier |

### 7.12 API Limitations

| Limitation | Description | Mitigation |
|------------|-------------|------------|
| Rate limits | Public instances rate-limited | Self-host Overpass for production |
| Timeout | 25 second default timeout | Increase timeout, optimize queries |
| Result size | Large results may be truncated | Limit search radius, filter types |
| Availability | Public instances may be down | Fallback to cached data |

### 7.13 Failure Handling

| Failure | Cause | Handling |
|---------|-------|----------|
| API timeout | Server too slow | Return cached/empty results |
| Rate limited | Too many requests | Exponential backoff |
| Invalid response | Malformed JSON | Log error, return empty |
| Service unavailable | Overpass down | Show "No meeting points available" |

### 7.14 Frontend–Backend Interaction

#### POI Discovery Flow
1. Ride Leader initiates regrouping recommendation
2. Frontend sends group centroid to backend API
3. Backend queries Overpass API for nearby POIs
4. Backend filters and deduplicates results
5. Backend calculates OSRM travel times for each POI
6. Backend ranks POIs by worst-case ETA
7. Backend returns ranked recommendations to frontend
8. Frontend displays recommendations on map

#### Data Flow Example
```
Frontend → GET /api/meeting-points/ → Backend → Overpass API → Backend → OSRM API → Backend → Ranked POIs → Frontend → Map markers
```

### 7.15 Testing Requirements

#### Unit Tests
- Test Overpass query construction
- Test POI deduplication
- Test result parsing
- Test empty results handling

#### Integration Tests
- Test full POI discovery flow
- Test Overpass API unavailability
- Test result ranking with OSRM
- Test duplicate filtering accuracy

---

## 8. Push Notification Integration

### 8.1 Module Name
Push Notification Integration

### 8.2 Module Objective
Define device token registration, notification permission, notification creation, backend notification processing, Expo Notifications, FCM integration, foreground/background notification handling, and delivery failures.

### 8.3 Module Purpose
Provide the complete push notification specification for RydLink, ensuring reliable delivery of critical alerts to mobile devices.

### 8.4 Functional Requirements
- Register device push tokens
- Request notification permission
- Create notifications for various events
- Process notifications in backend
- Integrate Expo Notifications
- Integrate FCM (Android production)
- Handle foreground notifications
- Handle background notifications
- Handle delivery failures

### 8.5 Technologies and Packages Required
- Expo Notifications Latest
- Firebase Cloud Messaging (Android production)
- Django REST Framework (backend notification API)
- channels-redis (for notification triggering)

### 8.6 Dependencies on Other Modules
- Separation Detection (backend triggers)
- Emergency Management (backend triggers)
- Invitation Management (backend triggers)
- Notification Interface (frontend)

### 8.7 Folder and File Structure
```
src/services/
├── notifications.js    # Expo Notifications setup

apps/
├── notifications/
│   ├── models.py       # DeviceToken, Notification models
│   ├── services.py     # Push notification sending service
│   └── views.py        # Device token registration API
```

### 8.8 Device Token Registration

#### Frontend Registration Flow
```javascript
import * as Notifications from 'expo-notifications';

export const registerForPushNotifications = async () => {
  // Request permission
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;
  
  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }
  
  if (finalStatus !== 'granted') {
    return null;
  }
  
  // Get token
  const token = await Notifications.getExpoPushTokenAsync();
  
  // Send to backend
  await api.post('/api/device-token/', {
    token: token.data,
    platform: Platform.OS, // 'ios' or 'android'
  });
  
  return token.data;
};
```

#### Backend Device Token Model
```python
class DeviceToken(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    user = models.ForeignKey('users.User', on_delete=models.CASCADE)
    token = models.TextField(unique=True)
    platform = models.CharField(max_length=10, choices=[('ios', 'iOS'), ('android', 'Android')])
    created_at = models.DateTimeField(auto_now_add=True)
    last_used_at = models.DateTimeField(null=True)
```

### 8.9 Notification Categories

| Category | Title Template | Body Template | Actions |
|----------|---------------|---------------|---------|
| separation_warning | Separation Warning | {name} is {distance}m from group | I'm okay, Contact Leader |
| high_separation | High Separation Alert | {name} is {distance}m from group | I'm okay, Contact Leader, Emergency |
| emergency | Emergency Reported | {name} reported {type} - {severity} | Acknowledge, Share Location |
| ride_event | New Ride Event | {name} added {type} stop | View Event, Mark Completed |
| regrouped | Rider Regrouped | {name} has returned to group | None |
| invitation | New Invitation | {name} invited you to {ride} | Accept, Reject |

### 8.10 Backend Notification Processing

#### Notification Service
```python
from expo_server_sdk import ExpoPushClient

class PushNotificationService:
    def __init__(self):
        self.client = ExpoPushClient()
    
    def send_notification(self, user_id, title, body, data=None, category=None):
        # Get device tokens for user
        tokens = DeviceToken.objects.filter(user_id=user_id)
        
        messages = []
        for token in tokens:
            messages.append({
                'to': token.token,
                'title': title,
                'body': body,
                'data': data or {},
                'category': category,
                'sound': 'default' if category == 'emergency' else None,
            })
        
        # Send in chunks (Expo limit: 100 per request)
        for i in range(0, len(messages), 100):
            chunk = messages[i:i+100]
            self.client.send_push_notifications(chunk)
```

#### Notification Creation API
```python
# POST /api/notifications/
class NotificationCreateView(APIView):
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        serializer = NotificationSerializer(data=request.data)
        if serializer.is_valid():
            notification = serializer.save(user=request.user)
            
            # Send push notification
            PushNotificationService().send_notification(
                user_id=request.user.id,
                title=serializer.validated_data['title'],
                body=serializer.validated_data['body'],
                data=serializer.validated_data.get('data', {}),
                category=serializer.validated_data.get('category'),
            )
            
            return Response(serializer.data, status=201)
        return Response(serializer.errors, status=400)
```

### 8.11 Expo Notifications Integration

#### Configuration (app.json)
```json
{
  "expo": {
    "notifications": {
      "ios": {
        "allowInBackground": true,
        "foreground": true
      },
      "android": {
        "mode": "default",
        "icon": "./assets/icon.png",
        "color": "#4A90E2"
      }
    }
  }
}
```

#### Foreground Notification Handler
```javascript
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: true,
  }),
});

Notifications.addNotificationReceivedListener(notification => {
  // Handle notification while app is foreground
  console.log('Notification received:', notification);
});

Notifications.addNotificationResponseReceivedListener(response => {
  // Handle notification tap
  const { data } = response.notification.request.content;
  if (data.ride_id) {
    navigation.navigate('LiveMapScreen', { rideId: data.ride_id });
  }
});
```

### 8.12 FCM Integration (Android Production)

#### FCM Setup Requirements
- Firebase project created
- FCM credentials configured in Expo
- google-services.json in android/app/
- Push notification enabled in Firebase Console

#### FCM Token Flow
1. Expo Notifications registers device with FCM
2. FCM returns device token
3. Expo sends token to backend
4. Backend stores token in DeviceToken table
5. Backend sends push via Expo/FCM when needed

### 8.13 Foreground vs Background Notification Handling

| Aspect | Foreground | Background |
|--------|-----------|------------|
| Delivery | Expo Notifications JS handler | System notification |
| Display | Custom in-app UI | System notification banner |
| Sound | Configurable (shouldPlaySound) | System default |
| Badge | Configurable (shouldSetBadge) | System badge |
| Action | Notification response listener | App opens on tap |
| Permission | Required for iOS | Required for iOS/Android |

### 8.14 Notification Delivery Failures

| Failure | Handling |
|---------|----------|
| Invalid token | Remove from database, log warning |
| Token expired | Request new token on next app launch |
| Network error | Retry up to 3 times, then dead letter queue |
| Rate limited | Exponential backoff, queue for later |
| User disabled | Respect user preference, don't retry |

### 8.15 Frontend–Backend Interaction

#### Notification Flow
1. Event triggers (separation, emergency, invitation)
2. Backend creates notification record
3. Backend sends push notification via Expo/FCM
4. Device receives notification
5. User taps notification → App opens to related screen
6. Frontend marks notification as read
7. Backend updates read status

#### Data Flow Example
```
Event → Backend API → PushNotificationService → Expo/FCM → Device → User taps → Frontend navigation → Mark as read → Backend update
```

### 8.16 Testing Requirements

#### Unit Tests
- Test device token registration
- Test notification creation
- Test push notification sending
- Test notification category mapping
- Test token cleanup on logout

#### Integration Tests
- Test full notification flow (event → push → device)
- Test foreground notification handling
- Test background notification handling
- Test delivery failure retry
- Test FCM integration (Android)

---

## 9. Background GPS Tracking Integration

### 9.1 Module Name
Background GPS Tracking Integration

### 9.2 Module Objective
Define background location permission, background task configuration, location update strategy, Android foreground service requirements, application termination behaviour, battery considerations, network interruption, location synchronization, and testing requirements.

### 9.3 Module Purpose
Provide the complete background GPS tracking specification for RydLink, ensuring reliable location updates even when the app is not in the foreground.

### 9.4 Functional Requirements
- Request background location permission
- Configure background tasks
- Define location update strategy for background
- Implement Android foreground service requirements
- Handle application termination behaviour
- Optimize battery consumption
- Handle network interruption
- Synchronize location after interruption
- Test background tracking scenarios

### 9.5 Technologies and Packages Required
- Expo Location Latest
- Expo Task Manager Latest
- React Native BackgroundTask (alternative)
- Android Foreground Service

### 9.6 Dependencies on Other Modules
- GPS Location Integration (frontend)
- Real-Time Participant Tracking (frontend)
- WebSocket Integration (frontend/backend)

### 9.7 Folder and File Structure
```
src/services/
├── background-tracking.js    # Background task registration

android/
├── app/src/main/AndroidManifest.xml  # Foreground service declaration
```

### 9.8 Background Location Permission

#### Permission Request Flow
1. Request foreground location permission first
2. After user grants, request background location permission
3. Explain why background location is needed
4. Handle permission denial gracefully

#### iOS Configuration (Info.plist)
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>RydLink needs your location to coordinate group rides</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>RydLink needs background location to track participants during rides</string>
<key>UIBackgroundModes</key>
<array>
  <string>location</string>
  <string>remote-notification</string>
</array>
```

#### Android Configuration (AndroidManifest.xml)
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

<service
    android:name="expo.modules.location.LocationForegroundService"
    android:foregroundServiceType="location" />
```

### 9.9 Background Task Configuration

#### Expo Task Manager Setup
```javascript
import * as TaskManager from 'expo-task-manager';
import * as Location from 'expo-location';

const LOCATION_TASK_NAME = 'rydlink-location-task';

TaskManager.defineTask(LOCATION_TASK_NAME, ({ data, error }) => {
  if (error) {
    console.error('Background location error:', error);
    return;
  }
  
  const { locations } = data;
  const location = locations[0];
  
  // Send location to backend via API or store locally
  sendLocationToBackend({
    latitude: location.coords.latitude,
    longitude: location.coords.longitude,
    accuracy: location.coords.accuracy,
    speed: location.coords.speed,
    timestamp: location.timestamp,
  });
});
```

#### Start Background Tracking
```javascript
export const startBackgroundTracking = async () => {
  await Location.startLocationUpdatesAsync(LOCATION_TASK_NAME, {
    accuracy: Location.Accuracy.Balanced,
    distanceInterval: 10, // metres
    timeInterval: 60000,  // 1 minute
    foreground: false,
  });
};
```

### 9.10 Location Update Strategy

| State | Update Interval | Accuracy | Battery Impact |
|-------|----------------|----------|----------------|
| Foreground (active ride) | 5-10 seconds | High | Moderate |
| Background (ride active) | 30-60 seconds | Balanced | Low |
| App terminated | No updates | N/A | None |
| Ride paused | No updates | N/A | None |

### 9.11 Android Foreground Service Requirements

#### Foreground Service Notification
- Must show persistent notification when background tracking active
- Notification title: "RydLink - Location Tracking"
- Notification content: "Sharing location with ride participants"
- Cannot be dismissed by user while tracking active

#### Service Declaration
```xml
<service
    android:name=".LocationForegroundService"
    android:enabled="true"
    android:exported="false"
    android:foregroundServiceType="location" />
```

### 9.12 Application Termination Behaviour

| Event | Behaviour |
|-------|-----------|
| App minimized | Background tracking continues |
| Screen locked | Background tracking continues |
| App killed by OS | Tracking stops, must restart on relaunch |
| App killed by user | Tracking stops, must restart on relaunch |
| Device reboot | Tracking stops, must restart manually |

### 9.13 Battery Considerations

#### Optimization Strategies
- Use `distanceInterval: 10` to reduce updates when stationary
- Use `Accuracy.Balanced` instead of `Accuracy.High` for background
- Increase time interval in background (60s vs 10s)
- Pause tracking when ride is not active
- Show battery warning after 2 hours continuous tracking

#### Battery Impact Estimates
| Scenario | Estimated Drain |
|----------|----------------|
| Foreground tracking (1 hour) | ~5-8% battery |
| Background tracking (1 hour) | ~2-4% battery |
| No tracking (idle) | ~0.5% battery |

### 9.14 Network Interruption Handling

#### Offline Behaviour
- Location updates stored locally when offline
- Sync with backend when connection restored
- WebSocket reconnection with exponential backoff
- Last known location used for separation detection

#### Sync Strategy
```javascript
const syncPendingLocations = async () => {
  const pending = await getStoredLocations();
  for (const location of pending) {
    try {
      await sendLocationUpdate(location);
      await removeStoredLocation(location.id);
    } catch (e) {
      // Keep for next sync attempt
      break;
    }
  }
};
```

### 9.15 Location Synchronization

#### After App Restart
1. Check for pending unsynced locations
2. Request current location from GPS
3. Sync with backend API
4. Resume background tracking if ride is active

#### After Network Restoration
1. Flush pending location queue
2. Request current participant locations from API
3. Resume WebSocket connection
4. Update UI with synced data

### 9.16 Testing Requirements

#### Background Location Test Cases

| Test Case | Scenario | Expected Result |
|-----------|----------|-----------------|
| TC-BG01 | App in foreground | Location updates every 5-10s |
| TC-BG02 | App minimized | Location updates every 30-60s |
| TC-BG03 | Screen locked | Background tracking continues |
| TC-BG04 | App restarted | Tracking resumes if ride active |
| TC-BG05 | Network unavailable | Locations stored locally, synced later |
| TC-BG06 | Permission revoked | Tracking stops, user notified |
| TC-BG07 | Android battery restrictions | Tracking may be limited by OS |
| TC-BG08 | Recovery after interruption | Locations synced, tracking resumes |

#### Battery Testing
- Measure battery drain over 2 hours of continuous tracking
- Compare foreground vs. background drain
- Verify tracking pauses when ride ends
- Test with GPS accuracy settings

#### Network Testing
- Test location storage during offline periods
- Test sync when connection restored
- Test WebSocket reconnection
- Test stale location indicators

---