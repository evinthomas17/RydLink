# Section 6: System Architecture

## 6.1 Overall System Architecture

```mermaid
graph TB
    subgraph Mobile_Frontend "Mobile Frontend (React Native + Expo)"
        A[App UI Layer]
        B[MapLibre Map]
        C[Expo Location GPS]
        D[WebSocket Client]
        E[Expo Notifications]
        F[NetInfo Network Monitor]
    end

    subgraph API_Gateway "API Layer (Django + DRF)"
        G[REST Controllers]
        H[JWT Authentication]
        I[Request Validation]
    end

    subgraph Real_Time "Real-Time Layer (Django Channels + Daphne)"
        J[WebSocket Consumers]
        K[Ride Groups]
        L[Event Broadcasting]
    end

    subgraph Business_Logic "Business Logic Layer"
        M[Ride Service]
        N[Monitoring Engine]
        O[Regrouping Algorithm]
        P[Event Service]
        Q[Emergency Service]
    end

    subgraph Data_Layer "Data Layer (PostgreSQL + PostGIS)"
        R[Users Table]
        S[Rides Table]
        T[Ride Participants]
        U[Location Updates]
        V[Alerts Table]
        W[Events Table]
        X[Emergency Reports]
    end

    subgraph External_Services "External Services"
        Y[OSRM Routing]
        Z[Overpass API]
        AA[OpenStreetMap Tiles]
        AB[FCM / Expo Push]
    end

    subgraph Cache "Cache (Redis)"
        AC[Channel Layer]
        AD[Session Cache]
    end

    A --> G
    B --> AA
    C --> D
    D --> J
    E --> AB
    G --> H
    G --> M
    J --> K
    K --> L
    L --> M
    M --> N
    M --> O
    N --> V
    O --> W
    P --> W
    Q --> X
    M --> R
    M --> S
    M --> T
    N --> U
    O --> U
    Y -.-> M
    Z -.-> O
    J <--> AC
    AC --> AD
```

### Component Communication

- **Frontend → Backend REST:** HTTPS API calls via Axios with JWT Bearer token.
- **Frontend ↔ Backend Real-time:** WebSocket (WSS) via Daphne/Channels, ride-scoped channel groups.
- **Backend → Database:** Django ORM over psycopg to PostgreSQL + PostGIS.
- **Backend → External Services:** HTTP requests to OSRM, Overpass API, Nominatim (optional).
- **Backend → Push:** HTTP to Expo Notifications / FCM.
- **Channels → Redis:** Pub/sub channel layer for multi-worker broadcast.

## 6.2 Frontend Architecture

```
Frontend Layers:
├── Presentation Layer (React Native components, JSX, StyleSheet)
│   ├── Screens (per module: auth, ride, lobby, map, alerts, events, history)
│   └── Components (markers, polylines, buttons, modals)
├── Navigation Layer (React Navigation stack/tab/drawer)
├── State Layer (React Context / Zustand / Redux)
├── Service Layer
│   ├── API Client (Axios with JWT interceptor)
│   ├── WebSocket Client (reconnect, auth, ride groups)
│   ├── Location Service (Expo Location, background tasks)
│   ├── Notification Service (Expo Notifications, FCM)
│   └── Network Service (NetInfo)
└── Map Layer (MapLibre React Native, OSM tiles)
```

## 6.3 Backend Architecture

```
Backend Layers (Django):
├── API Layer (Django REST Framework)
│   ├── ViewSets per module
│   ├── Serializers
│   ├── Permissions (ride membership, leader-only)
│   └── Authentication (Simple JWT)
├── WebSocket Layer (Django Channels + Daphne)
│   ├── Consumers (ride group, auth)
│   ├── Channel layer (Redis)
│   └── ASGI application
├── Service Layer
│   ├── RideService
│   ├── MonitoringEngine (Python rules)
│   │   ├── SeparationDetector (distance thresholds, confirmation timers)
│   │   ├── GPSValidator (accuracy ≤ 30 m, timestamp freshness < 2 min)
│   │   ├── GroupReferenceCalculator (centroid of active participants)
│   │   ├── RegroupingDetector (300 m for 30 seconds)
│   │   └── AlertManager (alert creation, resolution, broadcast)
│   ├── RegroupingService
│   ├── NotificationService
│   └── OSRMClient / OverpassClient
└── Data Layer (Django ORM → PostgreSQL + PostGIS)
```

### Monitoring Engine Responsibilities

- **GPS Validation:** Reject readings with accuracy > 30 metres or timestamp older than 2 minutes.
- **Group Reference Position:** Calculate geographic centroid of all active ride participants' latest valid locations. Handles distributed groups where participants may be in different locations.
- **Distance Calculation:** Use Haversine formula or PostGIS `ST_Distance` to compute rider-to-group distance in meters.
- **Confirmation Timers:** Track elapsed time above each threshold per rider. Reset/pause when rider returns below threshold.
- **Alert Management:** Create Separation Warning (> 500 m for 60s), High Separation Alert (> 1,000 m for 30s), and auto-resolve on regrouping (> 300 m for 30s). Prevent duplicate alerts for same unresolved event.
- **Broadcast:** Send separation alerts and regrouping notifications via WebSocket to Ride Leader and affected rider.

## 6.4 Database Architecture

- Primary: PostgreSQL 14/15 + PostGIS 3.x
- `geography` type for points (SRID 4326), `geometry` for route linestrings
- Spatial indexes (GiST) on location and route columns
- Indexes on foreign keys, status fields, timestamps

## 6.5 Real-Time Communication Architecture

```
Client (Expo) ← WebSocket (wss://) → Daphne (ASGI) → Redis Channel Layer → Django Consumers
                                              ↳ Broadcasts to ride groups
```

- Connection: `wss://api.rydlink.com/ws/rides/{ride_id}/` with JWT in query param or subprotocol
- Redis: channel layer for multi-worker broadcast
- Groups: one channel group per ride (`ride_{ride_id}`)

## 6.6 External Service Integration Architecture

| Service | Purpose | Integration | Fallback |
|---------|---------|-------------|----------|
| OSRM | Route calculation, distance, ETA | HTTP REST | Straight-line distance |
| Overpass API | Nearby POIs (fuel, food, parking) | HTTP REST | Manual leader entry |
| OpenStreetMap | Map tiles | Tile URL (MapLibre) | Cached/offline tiles |
| Nominatim | Geocoding (optional) | HTTP REST | User-entered coords |
| FCM/Expo | Push notifications | HTTP API | In-app only |

## 6.7 Deployment Architecture

```
Production:
[Mobile Clients] → [HTTPS Load Balancer] → [Daphne ASGI Workers] → [Redis] → [PostgreSQL + PostGIS]
                                                ↳ [OSRM Container] ← [Road Network .pbf]
                                                ↳ [Overpass Container]
```

Development:
- Django runserver + Daphne for WebSocket
- PostgreSQL + Redis installed natively
- OSRM demo server or local build
