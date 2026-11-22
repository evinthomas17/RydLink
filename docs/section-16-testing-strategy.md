# Section 16: Testing Strategy

## 16.1 Test Types

| Type | Scope | Tools |
|------|-------|-------|
| Unit | Individual functions (algorithms, validation) | Python unittest, Jest |
| API | REST endpoints, auth, validation | Postman, Django test client |
| Database | Schema, constraints, migrations, spatial queries | pgAdmin, Django test DB |
| WebSocket | Connection, auth, broadcast, reconnect | websocket-client, Channels test |
| GPS | Mock locations, accuracy, background | Expo location mocking |
| Map | Marker rendering, polyline, tile loading | MapLibre testing, visual |
| Integration | Cross-module flows | Django + React Native integration |
| System | Full app on device/emulator | Detox, Appium |
| Security | Auth bypass, injection, rate limiting | OWASP ZAP, manual |
| UAT | End-to-end user scenarios | Beta testers |

## 16.2 Test Case Table

| Test Case ID | Module | Scenario | Steps | Expected Result | Status |
|--------------|--------|----------|-------|-----------------|--------|
| TC-001 | 1 | Valid registration | Submit unique email + password | 201, user created, tokens issued | — |
| TC-002 | 1 | Duplicate registration | Register with existing email | 409, error message | — |
| TC-003 | 1 | Invalid login | Wrong password | 401, invalid credentials | — |
| TC-004 | 2 | Create ride | Fill valid ride form | 201, ride created, leader assigned | — |
| TC-005 | 2 | Past departure time | Set departure_time in past | 400, validation error | — |
| TC-006 | 3 | Send invitation | Leader invites existing user | 201, invitation created | — |
| TC-007 | 3 | Accept invitation | Invitee accepts | 200, participant created | — |
| TC-008 | 3 | Expired invitation | Use token after 24h | 410, expired | — |
| TC-009 | 5 | Location broadcast | Send GPS via WebSocket | Other participants receive update | — |
| TC-010 | 5 | GPS permission denied | Deny location permission | Tracking disabled, prompt shown | — |
| TC-011 | 6 | Route calculation | Valid start + dest | OSRM returns polyline, distance, ETA | — |
| TC-012 | 6 | OSRM unavailable | Mock OSRM failure | Fallback to straight-line, error toast | — |
| TC-013 | 7 | Separation detection | Participant > 300m from group | Separation alert generated | — |
| TC-014 | 7 | False positive prevention | Participant within threshold | No alert | — |
| TC-015 | 8 | Meeting point recommendation | Separation alert triggered | Ranked candidates returned | — |
| TC-016 | 8 | Leader approves meeting point | Leader approves | All participants notified | — |
| TC-017 | 10 | Emergency report | Report breakdown | Immediate WebSocket + push alert | — |
| TC-018 | 11 | Push notification | Event occurs offline | Notification received on reconnect | — |
| TC-019 | 12 | Network disconnect | Turn off airplane mode | Offline indicator, cache locations | — |
| TC-020 | 13 | End ride | Leader ends active ride | status='completed', broadcast ride.ended | — |
| **TC-S01** | **7** | **Normal movement (200m from group)** | **Rider 200m from group, stationary** | **No alert; status = Normal** | — |
| **TC-S02** | **7** | **Monitoring zone (400m from group)** | **Rider 400m from group for 2 monitoring cycles** | **Monitoring status; no alert yet** | — |
| **TC-S03** | **7** | **Separation Warning triggered** | **Rider 600m from group continuously for 60s** | **Separation Warning alert generated** | — |
| **TC-S04** | **7** | **High Separation Alert triggered** | **Rider 1200m from group continuously for 30s** | **High Separation Alert generated** | — |
| **TC-S05** | **7** | **No high alert yet (1200m for 20s)** | **Rider 1200m from group for 20s only** | **No high-separation alert** | — |
| **TC-S06** | **7** | **Regrouping resolution** | **Rider returns within 300m for 15s** | **Alert remains active** | — |
| **TC-S07** | **7** | **Auto-resolve on regrouping** | **Rider remains within 300m for 30s** | **Alert auto-resolves; status = Normal** | — |
| **TC-S08** | **7** | **GPS accuracy below threshold** | **GPS accuracy = 50m** | **Reading ignored for separation evaluation** | — |
| **TC-S09** | **7** | **Stale location used** | **Location timestamp > 2 minutes old** | **Do not confirm separation using stale data** | — |
| **TC-S10** | **7** | **Participant disconnect** | **Participant loses connection** | **Shown as location unavailable; NOT classified as separated** | — |
| **TC-S11** | **7** | **Multiple riders at different distances** | **Rider A: 200m, Rider B: 600m** | **Each evaluated independently; A=Normal, B=Monitoring/Warning** | — |
| **TC-S12** | **7** | **Repeated threshold crossing** | **Rider crosses 500m threshold repeatedly** | **No duplicate alerts; stable alert generation** | — |
| **TC-S13** | **7** | **Group reference position calculation** | **3 participants in different locations** | **Group reference calculated from all active participants** | — |
| TC-021 | 13 | End ride | Leader ends active ride | status='completed', broadcast ride.ended | — |
