# Section 2: Requirement Analysis

## 2.1 Functional Requirements

| ID | Requirement | Module | Priority |
|----|-------------|--------|----------|
| FR-001 | User shall register with email, password, and phone number. | 1 | High |
| FR-002 | User shall log in and receive JWT access + refresh tokens. | 1 | High |
| FR-003 | User shall view and update profile (name, phone, profile picture, emergency contact). | 1 | Medium |
| FR-004 | User shall change password and reset via email. | 1 | Medium |
| FR-005 | Authenticated user shall create a ride (name, start, destination, departure time, ride type). | 2 | High |
| FR-006 | Ride creator shall be automatically assigned as Ride Leader. | 2 | High |
| FR-007 | Ride Leader shall invite registered users by username/email. | 2, 3 | High |
| FR-008 | System shall generate unique invitation code per ride. | 3 | High |
| FR-009 | Invited user shall accept or reject invitation. | 3 | High |
| FR-010 | Ride Leader shall view and manage participants (approve, remove). | 4 | High |
| FR-011 | Participant shall view ride details and participant list in lobby. | 4 | High |
| FR-012 | Ride Leader shall start and end the ride. | 4, 13 | High |
| FR-013 | App shall acquire GPS coordinates (latitude, longitude, accuracy, timestamp). | 5 | High |
| FR-014 | App shall transmit location updates to backend via WebSocket. | 5 | High |
| FR-015 | Backend shall broadcast location updates to ride participants via WebSocket. | 5 | High |
| FR-016 | Map shall display all participants as markers with ride status. | 5 | High |
| FR-017 | System shall calculate route (polyline, distance, ETA) between start and destination via OSRM. | 6 | High |
| FR-018 | Route shall be displayed on map as polyline. | 6 | High |
| FR-019 | System shall detect group separation using configurable distance thresholds with confirmation duration. | 7 | High |
| FR-020 | System shall detect route deviation based on distance-from-route threshold. | 7 | High |
| FR-021 | System shall detect unusual stationary activity (speed < threshold, duration). | 7 | High |
| FR-022 | System shall generate and broadcast alerts via WebSocket. | 7 | High |
| FR-034 | System shall validate GPS accuracy (≤ 30 metres) before using location for separation evaluation. | 7 | High |
| FR-035 | System shall validate location timestamp freshness before using location for separation evaluation. | 7 | High |
| FR-036 | System shall classify rider distance from group as Normal (0–300 m), Monitoring (300–500 m), Separation Warning (>500 m for 60s), or High Separation (>1,000 m for 30s). | 7 | High |
| FR-037 | System shall generate Separation Warning alert when rider remains >500 m from group continuously for 60 seconds. | 7 | High |
| FR-038 | System shall generate High Separation Alert when rider remains >1,000 m from group continuously for 30 seconds. | 7 | High |
| FR-039 | System shall automatically resolve separation alert when rider returns within 300 m of group continuously for 30 seconds. | 7 | High |
| FR-040 | System shall pause/reset confirmation timer when rider returns below the relevant distance threshold. | 7 | High |
| FR-041 | System shall prevent repeated alerts for the same unresolved separation event. | 7 | High |
| FR-042 | System shall calculate group reference position using current locations of active ride participants. | 7 | High |
| FR-043 | System shall not classify a disconnected participant as separated solely because location updates have stopped. | 7 | High |
| FR-044 | System shall notify Ride Leader of separation alerts; affected rider shall receive appropriate feedback. | 7 | High |
| FR-023 | System shall recommend meeting points based on participant locations. | 8 | High |
| FR-024 | Ride Leader shall approve or reject meeting point recommendation. | 8 | High |
| FR-025 | Approved meeting point shall be shared with all participants via WebSocket. | 8 | High |
| FR-026 | Participant shall create ride events (fuel, food, rest, breakdown, emergency). | 9, 10 | High |
| FR-027 | Ride events shall be broadcast to participants via WebSocket. | 9 | High |
| FR-028 | Emergency reports shall trigger high-priority push notification + WebSocket alert. | 10 | Critical |
| FR-029 | System shall send push notifications for invitations, alerts, events, emergencies. | 11 | High |
| FR-030 | App shall detect network disconnection and show stale-location indicators. | 12 | Medium |
| FR-031 | App shall resume location sharing on reconnection. | 12 | Medium |
| FR-032 | Ride Leader shall mark ride as completed; ride archived to history. | 13 | High |
| FR-033 | Participant shall view completed ride history (duration, participants, events, alerts). | 13 | Medium |

## 2.2 Non-Functional Requirements

| ID | Requirement | Category | Target |
|----|-------------|----------|--------|
| NFR-001 | Location update frequency (foreground) | Performance | Every 5–10 seconds |
| NFR-002 | Location update frequency (background) | Performance | Every 30–60 seconds |
| NFR-003 | WebSocket message latency | Performance | < 500 ms |
| NFR-004 | API response time (95th percentile) | Performance | < 800 ms |
| NFR-005 | Concurrent rides supported | Scalability | 1,000+ |
| NFR-006 | Participants per ride | Scalability | 50 max |
| NFR-007 | Password hashing algorithm | Security | bcrypt / Argon2 |
| NFR-008 | JWT access token expiry | Security | 15 minutes |
| NFR-009 | JWT refresh token expiry | Security | 7 days |
| NFR-010 | Location data retention (active rides) | Data | Latest location only |
| NFR-011 | Location data retention (completed rides) | Data | 90 days |
| NFR-012 | Map tile attribution | Compliance | OpenStreetMap attribution displayed |
| NFR-013 | Offline behavior | Reliability | Show last known location + stale warning |
| NFR-014 | App crash rate | Reliability | < 1% of sessions |
| NFR-015 | Battery impact | Performance | < 15% drain per hour of tracking |
| NFR-016 | Separation detection evaluation interval | Performance | Every 30 seconds |
| NFR-017 | Separation alert delivery latency | Performance | < 1 second from detection |
| NFR-018 | GPS accuracy threshold for separation evaluation | Data | ≤ 30 metres |
| NFR-019 | Location staleness threshold for separation evaluation | Data | < 2 minutes |
| NFR-020 | Separation threshold configurability | Maintainability | Thresholds stored in configuration, not hardcoded |

## 2.8 Separation Threshold Configuration (Initial Values)

| Condition | Distance | Confirmation Duration | System Behaviour |
|-----------|----------|----------------------|------------------|
| Normal | 0–300 m | — | No separation alert |
| Monitoring | 300–500 m | — | Monitor rider's movement |
| Separation Warning | > 500 m | 60 seconds | Generate a separation warning |
| High Separation | > 1,000 m | 30 seconds | Generate a high-separation alert |
| Regrouped | Within 300 m | 30 seconds | Automatically resolve the separation alert |

**Additional configuration:**
- Primary separation threshold: 500 metres.
- Primary confirmation duration: 60 seconds.
- High separation threshold: 1,000 metres.
- High separation confirmation duration: 30 seconds.
- GPS accuracy requirement: 30 metres or better.
- Regrouping threshold: Within 300 metres for 30 seconds.

**Note:** These are initial configurable project values, not universal standards. The system shall allow future adjustments through configuration rather than hardcoding thresholds throughout the application.

## 2.3 User Requirements

- Users can register/login with email and password.
- Users can create rides and invite friends.
- Users can see friends on a map during a ride.
- Users get alerts if someone falls behind or stops unexpectedly.
- Ride Leader can end the ride and view history.

## 2.4 Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Mobile device | Android 8.0 / iOS 13 | Android 12+ / iOS 15+ |
| GPS | Built-in GPS | A-GPS + GLONASS |
| RAM | 2 GB | 4 GB |
| Storage | 1 GB free | 2 GB free |
| Network | 3G | 4G/5G or Wi-Fi |
| Backend server | 2 vCPU, 4 GB RAM | 4 vCPU, 8 GB RAM |
| Database | PostgreSQL 14 + PostGIS 3.x | PostgreSQL 15 + PostGIS 3.3+ |
| Cache | Redis 6.x | Redis 7.x |

## 2.5 Software Requirements

| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React Native (JavaScript) | 0.73+ |
| Frontend | Expo SDK | 50+ |
| Maps | MapLibre React Native | Latest |
| Backend | Python | 3.11 / 3.12 |
| Backend | Django | 4.2 LTS |
| Backend | Django REST Framework | 3.14+ |
| Backend | Django Channels | 4.0+ |
| Backend | Daphne | 4.0+ |
| Backend | Simple JWT | 5.3+ |
| Database | PostgreSQL | 14/15 |
| Database | PostGIS | 3.x |
| Cache | Redis | 6/7 |
| Routing | OSRM (self-hosted) | Latest |
| Notifications | Expo Notifications + FCM | Latest |

## 2.6 System Constraints

- Expo Go does not support full background location tracking; development build required.
- Public OSRM demo server rate-limited; not for production.
- Public Overpass API instances have strict rate limits.
- Public Nominatim limited to 1 request/second; requires attribution.
- Push notifications in production require EAS build and FCM project.
- Python 3.15 compatibility with Django not guaranteed; use 3.11/3.12.

## 2.7 Assumptions and Dependencies

| ID | Assumption / Dependency |
|----|-------------------------|
| ASSUMP-01 | Participants have smartphones with GPS and internet connectivity. |
| ASSUMP-02 | Ride Leader has stable internet during the ride. |
| ASSUMP-03 | OSRM road network data is available for the ride region. |
| ASSUMP-04 | Users grant location permissions (foreground + background). |
| DEP-01 | PostgreSQL with PostGIS extension installed and enabled. |
| DEP-02 | Redis running for Django Channels channel layer. |
| DEP-03 | OSRM backend running for route calculation. |
| DEP-04 | FCM project configured for production push notifications. |
| DEP-05 | Map tile source configured (public tiles for dev, paid/self-hosted for prod). |
