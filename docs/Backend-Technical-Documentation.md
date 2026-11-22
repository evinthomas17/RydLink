# RydLink - Backend Technical Documentation

## Table of Contents
1. [Project Configuration and Architecture](#1-project-configuration-and-architecture)
2. [User Authentication and Profile Management](#2-user-authentication-and-profile-management)
3. [Ride Creation and Group Management](#3-ride-creation-and-group-management)
4. [In-App Invitation Management](#4-in-app-invitation-management)
5. [Ride Lobby and Participant Management](#5-ride-lobby-and-participant-management)
6. [Route Management](#6-route-management)
7. [Live Location Tracking APIs](#7-live-location-tracking-apis)
8. [WebSocket Communication](#8-websocket-communication)
9. [Intelligent Ride Monitoring](#9-intelligent-ride-monitoring)
10. [Rider Separation Detection and Alerts](#10-rider-separation-detection-and-alerts)
11. [Intelligent Regrouping](#11-intelligent-regrouping)
12. [Ride Event and Stop Management](#12-ride-event-and-stop-management)
13. [Emergency and Breakdown Management](#13-emergency-and-breakdown-management)
14. [Push Notification Management](#14-push-notification-management)
15. [Network Connectivity and Location Recovery](#15-network-connectivity-and-location-recovery)
16. [Ride Completion and History Management](#16-ride-completion-and-history-management)

---

## 1. Project Configuration and Architecture

### 1.1 Module Name
Project Configuration and Architecture

### 1.2 Module Objective
Define the overall backend project structure, settings, environment configuration, database setup, and server configuration for the RydLink Django application.

### 1.3 Module Purpose
Provide the foundational configuration for all backend modules, ensuring consistent project structure, environment management, and server setup across development, staging, and production environments.

### 1.4 Functional Requirements
- Establish Django project directory structure
- Configure Django settings (installed apps, middleware, databases)
- Manage environment variables via .env files
- Configure PostgreSQL with PostGIS extension
- Configure Django REST Framework settings
- Configure Django Channels with ASGI and Daphne
- Configure Redis channel layer
- Set up logging and exception handling
- Define development vs production configuration differences

### 1.5 Technologies and Packages Required
- Django 4.2 LTS
- Django REST Framework 3.14+
- Django Channels 4.0+
- Daphne 4.0+ (ASGI server)
- psycopg 3.x (PostgreSQL adapter)
- django-environ or python-dotenv (environment variable management)
- Redis 6/7 (channel layer, cache)
- pgcrypto extension (UUID primary keys)
- postgis extension (geographic operations)
- Python logging configuration (standard library)

### 1.6 Dependencies on Other Modules
- Required by all backend modules (user auth, rides, alerts, etc.)
- Provides the foundation upon which all other modules are built

### 1.7 Folder and File Structure
```
rydlink/
├── manage.py
├── rydlink/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── config/
│   ├── __init__.py
│   ├── celery.py
│   └── __pycache__/
├── apps/
│   ├── auth/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── migrations/
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── views_websocket.py
│   │   ├── urls.py
│   │   ├── permissions.py
│   │   ├── constants.py
│   │   ├── utils.py
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_auth.py
│   │       └── test_users.py
│   ├── rides/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── permissions.py
│   │   ├── constants.py
│   │   ├── utils.py
│   │   ├── routing.py
│   │   └── tests/
│   ├── participants/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── constants.py
│   │   └── tests/
│   ├── invitations/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── alerts/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── services.py
│   │   ├── utils.py
│   │   └── tests/
│   ├── monitoring/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── services.py
│   │   ├── algorithms.py
│   │   └── tests/
│   ├── events/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── emergencies/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── notifications/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── services.py
│   │   └── tests/
│   └── history/
│       ├── __init__.py
│       ├── apps.py
│       ├── models.py
│       ├── serializers.py
│       └── urls.py
├── .env (git-ignored)
├── requirements.txt
├── requirements-dev.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

### 1.8 Database Models and Fields
*(Refer to database-design-document.md for complete table specifications)*

Key models and their GIS fields:
- **User** (auth_user): `id` (UUID), `email`, `password_hash`, `is_active`, `created_at`
- **Ride** (rides): `id` (UUID), `name`, `leader_id` (FK), `status`, `departure_time`, `start_lat`, `start_lng`, `dest_lat`, `dest_lng`, `created_at`
- **RideParticipant** (ride_participants): `id` (UUID), `ride_id` (FK), `user_id` (FK), `role` (CHECK leader/participant), `status` (CHECK joined/left), `separation_status`, `separation_timer_started_at`
- **LocationUpdate** (location_updates): `id` (UUID), `ride_id` (FK), `user_id` (FK), `location` (geography Point SRID 4326), `accuracy`, `speed`, `heading`, `recorded_at`
- **RideRoute** (ride_routes): `id` (UUID), `ride_id` (FK UNIQUE), `geometry` (geometry LineString SRID 4326), `distance_m`, `duration_s`
- **RideAlert** (ride_alerts): `id` (UUID), `ride_id` (FK), `user_id` (FK), `type` (CHECK), `message`, `status` (CHECK active/resolved), `created_at`, `resolved_at`, `resolved_by`
- **MeetingPointRecommendation** (meeting_point_recommendations): `id` (UUID), `ride_id` (FK), `alert_id` (FK NULL), `name`, `type`, `lat`, `lng`, `rank_score`, `max_eta_s`, `status` (CHECK pending/approved/rejected/expired), `approved_by`, `approved_at`
- **RideEvent** (ride_events): `id` (UUID), `ride_id` (FK), `user_id` (FK), `type` (CHECK fuel/food/rest/breakdown/other), `location_lat`, `location_lng`, `address`, `note`, `status` (CHECK planned/completed/cancelled), `created_at`, `completed_at`
- **EmergencyReport** (emergency_reports): `id` (UUID), `ride_id` (FK), `user_id` (FK), `type` (CHECK breakdown/emergency), `severity` (CHECK low/medium/high/critical), `description`, `location_lat`, `location_lng`, `status` (CHECK active/acknowledged/resolved/false_alarm), `created_at`, `acknowledged_at`, `resolved_at`, `resolved_by`
- **DeviceToken** (device_tokens): `id` (UUID), `user_id` (FK), `token` (TEXT UNIQUE), `platform` (CHECK ios/android), `created_at`, `last_used_at`
- **Notification** (notifications): `id` (UUID), `user_id` (FK), `ride_id` (FK NULL), `type`, `title`, `body`, `data` (JSONB), `read_at`, `created_at`

**Spatial Indexes:**
- `lu_location_gist` on `location_updates.location` (GiST)
- `rr_geometry_gist` on `ride_routes.geometry` (GiST)

**PostGIS setup:**
```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS postgis;
```

### 1.9 API Specifications

#### Authentication Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/auth/register/` | POST | Public | `{email, password, phone, first_name, last_name}` | `201 {success, data, meta}` |
| `/api/auth/login/` | POST | Public | `{email, password}` | `200 {tokens: {access, refresh}, user}` |
| `/api/auth/refresh/` | POST | Public | `{refresh_token}` | `200 {access_token}` |
| `/api/auth/profile/` | GET/PUT | Bearer | `{first_name, last_name, profile_picture_url?}` | `200 {user}` |
| `/api/auth/logout/` | POST | Bearer | `{refresh_token}` | `200 {success}` |

#### Ride Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/rides/` | POST | Bearer | `{name, start_lat, start_lng, dest_lat, dest_lng, departure_time}` | `201 {ride}` |
| `/api/rides/` | GET | Bearer | — | `200 {list of rides}` |
| `/api/rides/{id}/` | GET | Bearer | — | `200 {ride details}` |
| `/api/rides/{id}/participants/` | GET | Bearer | — | `200 {participants list}` |
| `/api/rides/{id}/participants/` | POST | Bearer | `{invitee_id, message?}` | `201 {invitation sent}` |
| `/api/rides/{id}/participants/{user_id}/` | PUT | Bearer | `{role, status}` | `200 {updated participant}` |
| `/api/rides/{id}/start/` | POST | Leader only | — | `200 {ride started}` |
| `/api/rides/{id}/end/` | POST | Leader only | — | `200 {ride ended}` |
| `/api/rides/{id}/history/` | GET | Bearer | — | `200 {ride history}` |

#### Participant Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/participants/{ride_id}/status/` | GET/PUT | Bearer | `{status}` | `200 {participant status}` |
| `/api/participants/{ride_id}/separation-status/` | GET | Bearer | — | `200 {separation status}` |

#### Alert Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/alerts/` | GET | Bearer | — | `200 {alerts list}` |
| `/api/alerts/{id}/` | GET | Bearer | — | `200 {alert details}` |

#### Meeting Point Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/meeting-points/` | GET | Bearer | — | `200 {recommendations list}` |
| `/api/meeting-points/{id}/` | PUT | Bearer | `{approved: true|false}` | `200 {updated recommendation}` |
| `/api/meeting-points/{id}/share/` | POST | Leader | — | `200 {shared via WebSocket}` |

#### Event & Emergency Endpoints
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/events/` | POST | Bearer | `{ride_id, type, lat, lng}` | `201 {event created}` |
| `/api/events/{ride_id}/` | GET | Bearer | — | `200 {events list}` |
| `/api/emergencies/` | POST | Bearer | `{ride_id, type, severity, description, lat, lng}` | `201 {emergency reported}` |
| `/api/emergencies/{ride_id}/` | GET | Bearer | — | `200 {emergencies list}` |

#### Device Token Endpoint
| Endpoint | Method | Auth | Request Body | Response |
|----------|--------|------|--------------|----------|
| `/api/device-token/` | POST/PUT | Bearer | `{token, platform}` | `200/201 {token registered}` |

### 1.10 WebSocket Events

#### Connection
- URL: `ws://localhost:8000/ws/` or `wss://api.rydlink.com/ws/`
- Query params: `?ride_id={uuid}&user_id={uuid}&token={jwt}`
- Authentication: JWT token verification on connection
- Group: `ride_{ride_id}` channel

#### Event Names and Payloads

**Client → Server: `location_update`**
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

**Server → All Participants: `location_broadcast`**
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

**Server → Leader + Affected Rider: `separation_warning`**
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

**Server → Leader + Affected Rider: `high_separation`**
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

**Server → All Participants: `regrouped`**
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

**Server → All Participants: `ride_event`**
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

**Server → All Participants: `emergency`**
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

**Client → Server: `invitation_response`**
```json
{
  "type": "invitation_response",
  "ride_id": "uuid",
  "token": "uuid",
  "accepted": true|false
}
```

#### Connection Lifecycle
1. Client initiates WebSocket connection with ride_id, user_id, and JWT token
2. Server authenticates token, adds participant to `ride_{ride_id}` group
3. Server sends welcome message with participant role and status
4. Client sends `location_update` messages at configured frequency
5. Server broadcasts `location_broadcast` to all ride participants
6. Server evaluates separation detection engine every 30 seconds
7. Server generates alerts and broadcasts via WebSocket
8. Client disconnects → participant marked as `left`, ride ends

#### Redis Channel Layer Integration
- Channel layer configured as `channels_redis.core.RedisChannelLayer`
- Group name: `ride_{ride_id}`
- Message broadcasting: `channel_layer.group_send(ride_{ride_id}, ...)`
- Unicast to specific participant: `channel_layer.group_send(ride_{ride_id}, ...)` with participant filtering
- Fallback to in-memory channel layer for single-worker development

### 1.11 Processing Workflow

#### Location Update Processing
1. Frontend acquires GPS coordinates via Expo Location
2. Frontend sends `location_update` via WebSocket to backend
3. Backend WebSocket consumer receives message
4. Consumer validates GPS accuracy (≤ 30m) and timestamp freshness (< 2 min)
5. Invalid readings are rejected, last valid location used
6. Valid location stored in `location_updates` table via Django ORM
7. Consumer calculates group reference position (centroid of active participants)
8. Separation detection engine evaluates distance from group reference
9. If thresholds exceeded, alert created in `ride_alerts` table
10. Alert broadcast via WebSocket to ride participants
11. Frontend updates UI with current separation status

#### Separation Detection Workflow
1. Every 30 seconds (Celery beat or async loop), monitoring engine runs
2. For each active participant, calculate distance from group reference position
3. Apply configurable thresholds (normal/monitoring/warning/high/segregated)
4. Manage confirmation timers per participant per zone
5. Generate appropriate alert type if confirmation duration exceeded
6. Persist separation status in `ride_participants` table
7. Broadcast alert via WebSocket
8. If rider returns to normal zone, reset timers and resolve active alerts

### 1.12 Business Logic

#### Group Reference Position Calculation
- Compute geographic centroid of all active participant locations
- Use PostGIS `ST_Centroid` or average of lat/lng coordinates
- Handle distributed groups by weighting by participant count
- If only one active participant, group reference = that participant's location
- Exclude inactive/left participants from centroid calculation

#### Separation Zone Progression
- **Normal (0-300m)**: No alert, status = normal
- **Monitoring (300-500m)**: Start timer if transitioning from normal; monitor movement
- **Separation Warning (>500m)**: Start 60-second timer; generate alert if timer expires
- **High Separation (>1000m)**: Start 30-second timer; generate high-severity alert if timer expires
- **Regrouped (<300m for 30s)**: Auto-resolve active alert, broadcast regrouped notification

#### Timer Management Rules
- Timer starts when rider crosses into new distance zone
- Timer pauses/resets when rider returns below threshold
- After alert generation, timer resets to prevent duplicate alerts
- Timer state persisted in `ride_participants.separation_timer_started_at`
- Timer cleared when status returns to `normal`

### 1.13 Algorithms and Pseudocode

#### Group Reference Calculation
```python
def calculate_group_ref(participants):
    """Calculate geographic centroid of active participants."""
    valid_locations = [
        (p['latitude'], p['longitude']) 
        for p in participants.values() 
        if not p.get('is_left', False) and p.get('location_fresh', True)
    ]
    
    if not valid_locations:
        return None
    
    if len(valid_locations) == 1:
        return {'lat': valid_locations[0][0], 'lng': valid_locations[0][1]}
    
    # Calculate centroid using averaging (simple approach)
    # For production, use PostGIS ST_Centroid on geography points
    lat_sum = sum(p[0] for p in valid_locations)
    lng_sum = sum(p[1] for p in valid_locations)
    
    return {
        'lat': lat_sum / len(valid_locations),
        'lng': lng_sum / len(valid_locations)
    }
```

#### Separation Detection (expanded)
```python
def detect_separation(participants, group_ref, config):
    alerts = []
    now = datetime.utcnow()
    
    for user_id, rider in participants.items():
        # Skip stale locations
        if now - rider['timestamp'] > timedelta(minutes=config['location_freshness_minutes']):
            continue
        
        # Skip poor GPS accuracy
        if rider['accuracy'] > config['gps_accuracy_threshold_m']:
            continue
        
        # Calculate distance from group reference
        distance_m = haversine(rider['lat'], rider['lng'],
                               group_ref['lat'], group_ref['lng'])
        
        # Get current status and timer
        status = rider.get('separation_status', 'normal')
        timer = rider.get('separation_timer_started_at')
        
        # Zone evaluation
        if distance_m <= config['normal_distance_m']:
            # Regrouped
            if status != 'normal':
                alerts.append({...})  # regrouped alert
            status = 'normal'
            timer = None
            
        elif config['normal_distance_m'] < distance_m <= config['monitoring_distance_m']:
            if status != 'monitoring':
                status = 'monitoring'
                timer = now
            elif timer and (now - timer).seconds >= config['monitoring_confirmation_s']:
                pass  # In monitoring, no new alert
            else:
                status = 'monitoring'
                timer = now
                
        elif config['monitoring_distance_m'] < distance_m <= config['separation_warning_distance_m']:
            if status != 'separation_warning':
                status = 'separation_warning'
                timer = now
            elif timer and (now - timer).seconds >= config['separation_warning_confirmation_s']:
                alerts.append({...})  # separation_warning
                timer = now
            else:
                timer = now
                
        elif distance_m > config['high_separation_distance_m']:
            if status != 'high_separation':
                status = 'high_separation'
                timer = now
            elif timer and (now - timer).seconds >= config['high_separation_confirmation_s']:
                alerts.append({...})  # high_separation
                timer = now
            else:
                timer = now
        
        # Update rider state
        rider['separation_status'] = status
        rider['separation_distance'] = distance_m
        rider['separation_timer_started_at'] = timer
    
    return alerts
```

#### Timer Management
- **Confirmation timer** starts when rider crosses into a new distance zone
- Timer **pauses/resets** when rider returns below the relevant threshold
- **Duplicate alert prevention**: After generating an alert, reset timer to prevent repeated alerts for same event
- **Timer state** persisted in `ride_participants.separation_timer_started_at`

#### Automatic Regrouping Detection
- When separated rider returns within 300 m of group, start regrouping confirmation timer
- If rider remains within 300 m continuously for 30 seconds, auto-resolve active separation alert
- Update rider's `separation_status` to `normal`
- Send `separation.regrouped` notification via WebSocket to all participants
- Reset detection timers after resolution

### 1.14 Validation and Error Handling

#### GPS Validation Rules
- Latitude must be in range [-90, 90]
- Longitude must be in range [-180, 180]
- Accuracy must be non-negative and ≤ 30 metres for separation evaluation
- Timestamp must be within 2 minutes of current time
- Speed and heading should be reasonable (negative speed rejected, heading 0-360)

#### API Validation Errors
| Scenario | Error Code | Message | HTTP Status |
|----------|-----------|---------|-------------|
| Missing required fields | ERR_001 | "Required fields missing: <field>" | 400 |
| Invalid JWT token | ERR_002 | "Invalid or expired token" | 401 |
| User not in ride | ERR_003 | "User is not a participant of this ride" | 403 |
| Ride not active | ERR_007 | "Ride is not in active status" | 400 |
| Location too stale | ERR_007 | "Location timestamp is too old (>2 min)" | 400 |
| GPS accuracy insufficient | ERR_007 | "GPS accuracy exceeds threshold (>30m)" | 400 |
| Separation alert already active | ERR_008 | "Separation alert already exists for this rider" | 409 |
| WebSocket not connected | ERR_009 | "WebSocket connection required" | 400 |
| Database constraint violation | ERR_010 | "Duplicate entry or constraint violation" | 500 |

#### Exception Handling
- Django middleware catches unhandled exceptions
- Development: Full traceback in `details` field
- Production: Generic message only, logged server-side
- All API errors follow consistent format with `success`, `error`, `meta`

### 1.15 Security and Permissions

#### Authentication
- JWT access token expiry: 15 minutes (NFR-008)
- JWT refresh token expiry: 7 days (NFR-009)
- Password hashing: bcrypt or Argon2 (NFR-007)
- Token storage: Secure HTTP-only cookies or AsyncStorage

#### Permission Classes (DRF)
```python
class IsAuthenticated(BasePermission):
    def has_permission(self, request, view):
        auth = request.headers.get('Authorization', '').split()
        return bool(auth and auth[0].lower() == 'bearer')

class IsRideLeader(IsAuthenticated):
    def has_permission(self, request, view):
        if not super().has_permission(request, view):
            return False
        ride = view.get_ride()
        return ride.leader_id == request.user.id

class IsParticipant(IsAuthenticated):
    def has_permission(self, request, view):
        if not super().has_permission(request, view):
            return False
        ride = view.get_ride()
        return ride.participants.filter(id=request.user.id).exists()
```

#### Role-Based Access Control
- **Ride Leader**: Create ride, invite participants, start/end ride, approve meeting points, view all alerts, access emergency reports
- **Participant**: View ride, update own status, create events, respond to invitations, view own separation status
- **Anonymous**: Register, login, browse public ride listings (if enabled)

#### Input Validation
- All API endpoints validate request data via Django Serializers
- GPS coordinates validated: lat ∈ [-90, 90], lng ∈ [-180, 180]
- Accuracy values must be non-negative
- String fields have max_length constraints from model definitions
- Enum values constrained via CheckConstraints

### 1.16 Frontend–Backend Interaction

#### Data Flow: Location Update
1. Frontend (Expo Location) acquires GPS coordinates every 5-10s (foreground) or 30-60s (background)
2. Frontend sends WebSocket message: `{"type": "location_update", ...}`
3. Backend Channels consumer receives message on `ride_{ride_id}` group
4. Consumer validates accuracy and freshness
5. Valid location stored in PostgreSQL `location_updates` (PostGIS geography)
6. Group reference position recalculated
7. Separation detection engine evaluates thresholds
8. If alerts needed, created in `ride_alerts` table
9. WebSocket `separation_warning`/`high_separation`/`regrouped` broadcast to all participants
10. Frontend receives WebSocket message, updates UI markers and alerts

#### Data Flow: Ride Creation
1. Frontend sends POST `/api/rides/` with ride details (Bearer token auth)
2. Backend validates data via serializer
3. Ride created with leader_id = authenticated user
4. Ride status set to `planned`
5. Route calculated via OSRM (async task)
6. Route stored in `ride_routes` table
7. Response returns ride ID and details
8. Frontend navigates to ride lobby

#### Data Flow: Invitation
1. Frontend sends POST `/api/rides/{id}/participants/` with invitee_id
2. Backend creates `ride_invitations` record with unique token
3. Invitation expires at 24h (expires_at)
4. Invitation status set to `pending`
5. Email/notification sent to invitee (out of scope for backend)
6. Response returns invitation details and token
7. Frontend displays invitation in received invitations screen
8. Rider accepts/rejects via WebSocket `invitation_response` message

### 1.17 Expected Output

#### API Response Examples

**Registration Success:**
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "alice@example.com",
    "first_name": "Alice",
    "last_name": "Smith",
    "role": "participant"
  },
  "meta": {
    "timestamp": "2026-11-21T10:30:00Z",
    "request_id": "uuid-string"
  }
}
```

**Ride Creation Success:**
```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440001",
    "name": "Weekend Trip",
    "leader": "550e8400-e29b-41d4-a716-446655440000",
    "start_location": {"lat": 40.7128, "lng": -74.0060},
    "dest_location": {"lat": 40.7589, "lng": -73.9851},
    "departure_time": "2026-11-21T14:00:00Z",
    "status": "planned",
    "participant_count": 1
  },
  "meta": {
    "timestamp": "2026-11-21T10:30:00Z",
    "request_id": "uuid-string"
  }
}
```

**Separation Warning Broadcast:**
```json
{
  "type": "separation_warning",
  "ride_id": "660e8400-e29b-41d4-a716-446655440001",
  "user_id": "550e8400-e29b-41d4-a716-446655440001",
  "distance_m": 520,
  "severity": "warning",
  "message": "Separation Warning: 520m from group",
  "timestamp": "2026-11-21T10:30:00Z"
}
```

**Emergency Report Success:**
```json
{
  "success": true,
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440001",
    "ride_id": "660e8400-e29b-41d4-a716-446655440001",
    "user_id": "550e8400-e29b-41d4-a716-446655440001",
    "type": "emergency",
    "severity": "high",
    "description": "Rider experiencing mechanical failure",
    "location": {"lat": 40.7150, "lng": -74.0080},
    "status": "active",
    "created_at": "2026-11-21T10:30:00Z"
  },
  "meta": {
    "timestamp": "2026-11-21T10:30:00Z",
    "request_id": "uuid-string"
  }
}
```

### 1.18 Testing Requirements

#### Unit Tests for Configuration
- Test Django settings loaded correctly
- Test environment variables parsed correctly
- Test PostGIS extension enabled
- Test pgcrypto extension enabled
- Test Redis connection in channel layer
- Test Daphne ASGI server configuration
- Test logging configuration

#### Unit Tests for Settings
- Test DEBUG mode differences dev/prod
- Test ALLOWED_HOSTS configuration
- Test DATABASES configuration with PostGIS
- Test CORS settings
- Test JWT authentication settings
- Test channel layer configuration

#### Integration Tests
- Test full ride creation flow (API + DB + Channels)
- Test invitation send/receive flow
- Test location update WebSocket flow
- Test separation detection with various GPS readings
- Test alert generation and broadcasting
- Test meeting point recommendation flow
- Test emergency report flow

#### Configuration Tests
- Test threshold values are configurable
- Test default threshold values match approved config
- Test GPS accuracy threshold validation
- Test location freshness validation
- Test timer management edge cases

---