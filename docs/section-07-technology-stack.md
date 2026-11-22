# Section 7: Complete Technology Stack and Dependencies

## 7.1 Frontend Technologies

| Technology | Version | Purpose | Mandatory? | Alternative |
|------------|---------|---------|------------|-------------|
| React Native | 0.73+ | Mobile app framework | Yes | Flutter |
| JavaScript (ES6+) | — | Frontend language | Yes | TypeScript |
| JSX | — | UI syntax | Yes | — |
| React Native StyleSheet | — | Styling | Yes | CSS-in-JS |
| React Navigation | 6.x | Screen navigation | Yes | React Native Navigation |
| Axios | 1.x | REST API communication | Yes | Fetch API |
| MapLibre React Native | Latest | Map visualization | Yes | React Native Maps |
| Expo Location | Latest | GPS access | Yes | react-native-location |
| Expo Task Manager | Latest | Background tasks | Yes | react-native-background-task |
| Expo Notifications | Latest | Push notifications | Yes | react-native-push-notification-ios |

## 7.2 Frontend Packages

| Package | What it is | Why required | Module | Mandatory | Alternative |
|---------|-----------|--------------|--------|-----------|-------------|
| react-native | App framework | Core UI | All | Yes | Flutter |
| expo | Expo SDK | Native modules bridge | All | Yes | Bare RN |
| expo-location | GPS API | Location acquisition | 5, 8, 11, 12 | Yes | react-native-geolocation |
| expo-task-manager | Background task registration | Background tracking | 5, 12 | Yes | react-native-background-geolocation |
| expo-notifications | Push notification handling | Notifications | 11 | Yes | react-native-push-notification |
| @react-navigation/native | Navigation | Screen routing | All | Yes | @react-native-community/react-native-navigation |
| @maplibre/react-native-maplibre-gl | Map rendering | Map visualization | 5, 6, 8, 9 | Yes | react-native-maps |
| axios | HTTP client | API calls | All | Yes | fetch |

## 7.3 Backend Technologies

| Technology | Version | Purpose | Mandatory? | Alternative |
|------------|---------|---------|------------|-------------|
| Python | 3.11/3.12 | Backend language | Yes | — |
| Django | 4.2 LTS | Web framework | Yes | FastAPI (not allowed per requirements) |
| Django REST Framework | 3.14+ | REST APIs | Yes | — |
| Django Channels | 4.0+ | WebSocket support | Yes | — |
| Daphne | 4.0+ | ASGI server | Yes | Uvicorn |
| Simple JWT | 5.3+ | JWT authentication | Yes | Django OAuth Toolkit |
| psycopg | Latest | PostgreSQL driver | Yes | psycopg2 |
| requests | Latest | External API calls | Yes | urllib, httpx |

## 7.4 Backend Packages

| Package | What it is | Why required | Module | Mandatory | Alternative |
|---------|-----------|--------------|--------|-----------|-------------|
| django | Web framework | Request/response, ORM | All | Yes | FastAPI (prohibited) |
| djangorestframework | REST API toolkit | Serializers, viewsets | All | Yes | — |
| channels | WebSocket framework | Real-time comms | 5, 7, 8, 10, 12 | Yes | — |
| daphne | ASGI server | WebSocket serving | 5, 7, 8, 10, 12 | Yes | uvicorn |
| channels-redis | Redis channel layer | Multi-process broadcast | 5, 7, 8, 10, 12 | Yes | — |
| djangorestframework-simplejwt | JWT auth | Token auth | All | Yes | — |
| psycopg | PostgreSQL adapter | DB connection | All | Yes | psycopg2-binary |
| requests | HTTP client | OSRM, Overpass calls | 6, 8 | Yes | httpx |

## 7.5 Database Technologies

| Technology | Version | Purpose | Mandatory? | Alternative |
|------------|---------|---------|------------|-------------|
| PostgreSQL | 14/15 | Relational database | Yes | MySQL (no PostGIS) |
| PostGIS | 3.x | Spatial data, geography type | Yes | MySQL Spatial |

## 7.6 Map and Routing Services

| Service | Purpose | Usage Limits | Cost | Mandatory | Alternative |
|---------|---------|--------------|------|-----------|-------------|
| MapLibre | Map rendering | Client-side library | Free | Yes | React Native Maps |
| OpenStreetMap | Map data/tiles | Usage policy, attribution | Free (with attribution) | Yes | MapTiler, Mapbox |
| OSRM | Route calculation, distance, ETA | Rate-limited (demo) | Free (self-hosted) | Yes | GraphHopper, Valhalla |
| Overpass API | Nearby POI search | Rate-limited | Free (self-hosted for prod) | Yes | Foursquare, Google Places |
| Nominatim | Geocoding/reverse | 1 req/sec, attribution | Free | Optional | Mapbox Geocoding |

## 7.7 Real-Time Communication

| Technology | Purpose | Mandatory? | Alternative |
|------------|---------|------------|-------------|
| WebSocket | Bi-directional real-time | Yes | SSE, Long polling |
| Django Channels | WebSocket consumers | Yes | Socket.io + Node.js |
| Redis | Channel layer backend | Yes | In-memory (single worker only) |
| Daphne | ASGI server | Yes | Uvicorn |

## 7.8 Notification Services

| Service | Purpose | Mandatory? | Alternative |
|---------|---------|------------|-------------|
| Expo Notifications | Push notification client SDK | Yes | react-native-push-notification |
| Firebase Cloud Messaging | Android push delivery | Yes (production) | OneSignal |

## 7.9 Development Tools

| Tool | Purpose | Mandatory? |
|------|---------|------------|
| Visual Studio Code | IDE | Yes |
| Git/GitHub | Version control | Yes |
| Postman | API testing | Yes |
| Android Studio | Android emulator/build | Yes |
| Docker | Containerization | No (native install used) |
| pgAdmin | Database admin | No (psql CLI optional) |

## 7.10 Dependency Summary

**Critical path dependencies:**
- Module 1 → None (base)
- Module 2 → Module 1
- Module 3 → Module 1, Module 2
- Module 4 → Module 2, Module 3
- Module 5 → Module 1, Module 4
- Module 6 → Module 2
- Module 7 → Module 5, Module 6
- Module 8 → Module 5, Module 6, Module 7
- Module 9 → Module 4, Module 5
- Module 10 → Module 4, Module 5, Module 11
- Module 11 → Module 1
- Module 12 → Module 5
- Module 13 → Module 4, Module 5, Module 6, Module 7, Module 9, Module 10

## External Service Limitations

- **OpenStreetMap public tiles:** Require attribution; rate limits apply; not guaranteed production SLA.
- **OSRM public demo server:** Strict rate limits; not for production use.
- **Overpass API public instances:** Strict rate limits; production requires self-hosting.
- **Nominatim public server:** 1 request/second max; attribution required; not for production.
- **Expo Notifications + FCM:** Production requires EAS build and FCM project setup.
