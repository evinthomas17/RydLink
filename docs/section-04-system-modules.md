# Section 4: Detailed System Modules

This section documents all 13 modules. For each module: purpose, working process, user interactions, inputs/outputs, functional requirements, backend/frontend/database responsibilities, REST API, WebSocket, technologies, dependencies, validation, error handling, security, and testing scenarios.

---

## Module 1: User Authentication and Profile Management

**Purpose:** Provide secure access to the application and manage user information.

**Working Process:**
1. User submits registration (email, password, phone). Backend validates uniqueness of email, hashes password (bcrypt/Argon2), creates user record.
2. User logs in with email/password. Backend verifies credentials, issues JWT access (15 min) + refresh (7 days) tokens.
3. Frontend stores tokens securely, includes access token in API headers.
4. Profile endpoints allow GET/PATCH on authenticated user's record.
5. Password reset: email-based token flow.

**User Interactions:** Register screen, Login screen, Profile screen, Forgot password screen.

**Inputs:** Registration payload, login credentials, profile update fields, reset token.

**Outputs:** JWT tokens, user profile data, success/error responses.

**Functional Requirements:** FR-001 to FR-004.

**Backend Responsibilities:** Validate input, hash passwords, issue/verify JWT, manage refresh token rotation, email verification (optional).

**Frontend Responsibilities:** Token storage (secure storage), auth header injection, token refresh interceptor, auth guards on screens.

**Database Requirements:** `users` table (id, email, password_hash, phone, first_name, last_name, profile_picture_url, is_active, email_verified, created_at, updated_at).

**REST API:** `POST /api/v1/auth/register`, `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/logout`, `GET/PATCH /api/v1/users/me`, `POST /api/v1/auth/password-reset`.

**WebSocket:** Not applicable.

**Technologies:** Django REST Framework, Simple JWT, bcrypt/Argon2, PostgreSQL.

**Dependencies:** None.

**Validation Rules:** Email unique, password ≥ 8 chars, phone format E.164, email format valid.

**Error Handling:** 400 validation errors, 401 invalid credentials, 409 duplicate email, 404 user not found.

**Security:** HTTPS only, password never logged, rate limiting on login/register, refresh token rotation.

**Testing Scenarios:** Valid/invalid registration, duplicate email, wrong password, expired token, refresh token rotation.

---

## Module 2: Ride Creation and Group Management

**Purpose:** Allow authenticated users to create group rides and become Ride Leader.

**Working Process:**
1. User submits ride creation form (name, start location, destination, departure time, ride type).
2. Backend validates inputs, geocodes start/destination if needed, creates `rides` record with `status = 'planned'`, `creator = current user`.
3. Backend automatically inserts creator into `ride_participants` with `role = 'leader'`, `status = 'joined'`.
4. System generates unique invitation code.
5. Ride Leader can view, update (before start), or cancel the ride.

**User Interactions:** Create Ride screen, Ride Details screen, Ride Settings screen.

**Inputs:** Ride name, start location (lat/lng + address), destination (lat/lng + address), departure datetime, ride type.

**Outputs:** Ride ID, invitation code, ride details object.

**Functional Requirements:** FR-005, FR-006.

**Backend Responsibilities:** Ride CRUD, geocoding integration (Nominatim optional), invitation code generation, leader assignment, ride status lifecycle.

**Frontend Responsibilities:** Ride creation form, location picker (map-based), ride details display.

**Database Requirements:** `rides` table (id, name, leader_id, start_lat, start_lng, start_address, dest_lat, dest_lng, dest_address, departure_time, status, created_at, started_at, ended_at).

**REST API:** `POST /api/v1/rides`, `GET /api/v1/rides/{id}`, `PATCH /api/v1/rides/{id}`, `DELETE /api/v1/rides/{id}`, `GET /api/v1/rides` (list my rides).

**WebSocket:** Not applicable for creation; ride status changes broadcasted.

**Technologies:** Django REST Framework, PostgreSQL, Nominatim (optional).

**Dependencies:** Module 1 (authentication).

**Validation Rules:** Start != destination, departure_time in future, ride name non-empty, leader must be authenticated user.

**Error Handling:** 400 invalid coordinates, 403 not leader, 404 ride not found, 409 ride already started.

**Security:** Only leader can modify ride; ride membership required to view details.

**Testing Scenarios:** Successful creation, missing fields, past departure time, non-leader modification attempt, ride cancellation.

---

## Module 3: In-App Ride Invitation Management

**Purpose:** Ride Leader invites registered users; invitees accept/reject.

**Working Process:**
1. Ride Leader searches for users by username/email.
2. Leader selects users to invite; backend creates `ride_invitations` records (ride_id, inviter_id, invitee_id, token, status = 'pending', expires_at = now + 24h).
3. Invitee receives notification (WebSocket if online, push if offline).
4. Invitee views invitation, accepts or rejects.
5. On accept: backend creates `ride_participants` record (role = 'participant', status = 'joined'), updates invitation status to 'accepted'.
6. On reject: invitation status = 'rejected'.

**User Interactions:** User Search screen, Invite screen, Invitations screen (received), Invitation detail.

**Inputs:** Invitee identifier (username/email), ride ID, accept/reject action, invitation token.

**Outputs:** Invitation status, participant created (on accept).

**Functional Requirements:** FR-007, FR-008, FR-009.

**Backend Responsibilities:** Invitation CRUD, token generation (UUID), expiry enforcement, single-use validation, duplicate invitation prevention, status transitions.

**Frontend Responsibilities:** Search UI, invitation list, accept/reject buttons.

**Database Requirements:** `ride_invitations` table (id, ride_id, inviter_id, invitee_id, token, status, expires_at, created_at, responded_at).

**REST API:** `POST /api/v1/rides/{id}/invitations`, `GET /api/v1/invitations` (received), `POST /api/v1/invitations/{token}/respond`.

**WebSocket:** Invitation notification sent to invitee's ride group or user-specific channel.

**Technologies:** Django REST Framework, UUID, PostgreSQL.

**Dependencies:** Module 1 (users), Module 2 (rides).

**Validation Rules:** Inviter must be ride leader, invitee must exist, invitee not already participant, invitation not expired, token single-use, no duplicate pending invitation.

**Error Handling:** 403 not leader, 404 user/ride/invitation not found, 409 duplicate invitation, 410 expired invitation.

**Security:** Tokens unguessable (UUID v4), leader-only invitation sending, invitee-only response.

**Testing Scenarios:** Send invitation, accept, reject, expired invitation, duplicate invitation, non-leader sending, invalid token.

---

## Module 4: Ride Lobby and Participant Management

**Purpose:** Pre-ride space where participants view ride details and leader manages the group.

**Working Process:**
1. Participants join ride; appear in lobby with status.
2. Ride Leader views participant list, can approve pending join requests or remove participants.
3. Leader starts ride: `rides.status` changes to 'active', `started_at` set.
4. All participants receive WebSocket notification that ride started; location sharing begins.
5. Lobby transitions to active ride screen.

**User Interactions:** Ride Lobby screen, Participant List screen, Start Ride button (leader only).

**Inputs:** Ride ID, participant action (approve/remove), start ride action.

**Outputs:** Participant list, ride status change.

**Functional Requirements:** FR-010, FR-011, FR-012.

**Backend Responsibilities:** Participant status management, ride start/end transitions, broadcast ride state changes.

**Frontend Responsibilities:** Lobby UI, participant avatars/status badges, start ride button (conditional on leader role).

**Database Requirements:** `ride_participants` table (id, ride_id, user_id, role, status, joined_at, left_at, is_active); `rides.status` field.

**REST API:** `GET /api/v1/rides/{id}/participants`, `PATCH /api/v1/rides/{id}/participants/{user_id}`, `POST /api/v1/rides/{id}/start`, `POST /api/v1/rides/{id}/end`.

**WebSocket:** Ride state broadcast (started/ended), participant joined/left events.

**Technologies:** Django REST Framework, Django Channels, Redis, PostgreSQL.

**Dependencies:** Module 2 (rides), Module 3 (invitations).

**Validation Rules:** Only leader can start/end/modify participants, participant must be active member, ride must be 'planned' to start, 'active' to end.

**Error Handling:** 403 not leader, 409 wrong ride status, 404 participant not found.

**Security:** Ride membership verification on all endpoints; WebSocket group authorization.

**Testing Scenarios:** Leader starts ride, participant joins, leader removes participant, non-leader start attempt, ride already started.

---

## Module 5: Live Location Tracking and Map Visualization

**Purpose:** Real-time GPS sharing and map display of all participants.

**Working Process:**
1. App requests location permissions (foreground + background).
2. Foreground: `expo-location` gets position every 5–10 seconds; background: every 30–60 seconds via task manager.
3. Each location update sent via WebSocket to backend (`location.update` message with ride_id, lat, lng, accuracy, timestamp, speed).
4. Backend validates ride membership, validates GPS accuracy (≤ 30 metres) and timestamp freshness, updates `location_updates` table (or latest location cache), broadcasts to all ride participants via WebSocket group.
5. Frontend receives updates, updates map markers via MapLibre.
6. Markers show participant avatar, status, and last-updated timestamp.

**User Interactions:** Map screen (main), location permission dialog, tracking toggle.

**Inputs:** GPS coordinates, accuracy, timestamp, speed, ride ID.

**Outputs:** Broadcast location updates, map marker positions, separation status per participant.

**Functional Requirements:** FR-013, FR-014, FR-015, FR-016, FR-034, FR-035, FR-042.

**Backend Responsibilities:** WebSocket connection handling, ride membership validation, GPS accuracy validation (≤ 30 m), timestamp freshness validation, location storage/update, broadcast to ride group, group reference position calculation.

**Frontend Responsibilities:** Location permission requests, foreground/background tracking setup, WebSocket connection, map rendering, marker updates, separation status indicator display.

**Database Requirements:** `location_updates` table (id, ride_id, user_id, latitude, longitude, accuracy, speed, heading, recorded_at, server_received_at) with PostGIS `geography` point; `ride_participants` separation status fields.

**REST API:** `GET /api/v1/rides/{id}/locations` (latest locations of all participants with separation status).

**WebSocket:** `client → server: location.update`, `server → client: location.broadcast`, `server → client: separation.alert`, `server → client: separation.resolved`.

**Technologies:** Expo Location, Expo Task Manager, MapLibre React Native, Django Channels, Redis, PostgreSQL + PostGIS.

**Dependencies:** Module 1 (auth), Module 4 (ride membership), Module 7 (monitoring).

**Validation Rules:** Coordinates within valid range (-90 to 90, -180 to 180), accuracy ≤ 30 metres for separation evaluation, timestamp not in future, user is active ride member.

**Error Handling:** GPS unavailable → show last known + stale warning; GPS accuracy > 30 m → ignore for separation evaluation but still display; permission denied → prompt again or disable tracking; WebSocket disconnect → reconnect + resubscribe.

**Security:** Location only shared within ride group; no public location exposure; WebSocket auth required.

**Testing Scenarios:** Location update broadcast, marker movement, GPS permission denied, stale location indicator, WebSocket reconnect, GPS accuracy below threshold.

---

## Module 6: Route Management and Navigation

**Purpose:** Planned route visualization, distance, ETA.

**Working Process:**
1. At ride creation, start and destination coordinates are stored.
2. Backend calls OSRM (`/route/v1/driving/{start_lng},{start_lat};{dest_lng},{dest_lat}`) to get polyline, distance, duration.
3. Route polyline (GeoJSON LineString) stored in `ride_routes` table with PostGIS geometry.
4. Frontend fetches route, renders polyline on MapLibre alongside participant markers.
5. User can tap "Open in external nav" to launch Google Maps/Waze for turn-by-turn.

**User Interactions:** Route preview (ride creation), Route view (active ride), External navigation button.

**Inputs:** Start coordinates, destination coordinates, OSRM response.

**Outputs:** Route polyline, distance (meters), duration (seconds), ETA.

**Functional Requirements:** FR-017, FR-018.

**Backend Responsibilities:** OSRM integration, route storage, route retrieval API.

**Frontend Responsibilities:** MapLibre polyline rendering, distance/ETA display, external nav deep-link.

**Database Requirements:** `ride_routes` table (id, ride_id, geometry PostGIS LineString, distance_m, duration_s, created_at).

**REST API:** `GET /api/v1/rides/{id}/route`.

**WebSocket:** Not applicable (route is static per ride).

**Technologies:** OSRM, MapLibre, PostgreSQL + PostGIS, Django REST Framework.

**Dependencies:** Module 2 (ride start/destination).

**Validation Rules:** OSRM response valid, coordinates within service region.

**Error Handling:** OSRM unavailable → show direct line or error message; route fetch failure → retry, fallback to straight-line distance.

**Security:** Route data only accessible to ride members.

**Testing Scenarios:** Route calculation success, OSRM unavailable, route display on map, external nav launch.

---

## Module 7: Intelligent Ride Monitoring and Alert Management

**Purpose:** Detect rider separation from group, route deviations, stationary activity; generate alerts; detect automatic regrouping.

**Working Process:**
1. Backend monitoring service runs periodically (every 30 seconds) on latest validated locations.
2. **GPS Validation:** Before evaluating separation, check GPS accuracy (≤ 30 metres) and timestamp freshness (< 2 minutes). Ignore inaccurate or stale readings.
3. **Group Reference Position:** Calculate using current locations of all active ride participants (geographic centroid or leader-based reference). Handle distributed groups across multiple locations.
4. **Distance Calculation:** For each active participant, calculate geographical distance from rider to group reference using Haversine formula or PostGIS `ST_Distance`.
5. **Separation Classification:**
   - **Normal (0–300 m):** No separation alert.
   - **Monitoring (300–500 m):** Monitor rider's movement; no alert generated.
   - **Separation Warning (> 500 m):** Start/continue 60-second confirmation timer. If rider remains > 500 m continuously for 60 seconds, generate Separation Warning alert.
   - **High Separation (> 1,000 m):** Start/continue 30-second confirmation timer. If rider remains > 1,000 m continuously for 30 seconds, generate High Separation Alert.
6. **Timer Management:** Pause or reset confirmation timer when rider returns below the relevant threshold. Prevent repeated alerts for the same unresolved separation event.
7. **Automatic Regrouping:** When separated rider returns within 300 m of group, start 30-second regrouping confirmation timer. If rider remains within 300 m continuously for 30 seconds, automatically resolve active separation alert, update status to Normal, notify participants.
8. Alerts stored in `ride_alerts` table, broadcast via WebSocket to ride group, push notification sent for high-severity alerts.
9. Alerts can be resolved by Ride Leader or auto-resolve on regrouping.

**User Interactions:** Alert display screen, Alert list, Acknowledge/Resolve button, Separation status indicator on map.

**Inputs:** Location updates, route geometry, thresholds, timestamps, group reference position.

**Outputs:** Alert objects, broadcast notifications, separation status per participant, regrouping notification.

**Functional Requirements:** FR-019, FR-020, FR-021, FR-022, FR-034 through FR-044.

**Backend Responsibilities:** Monitoring algorithm execution (separation, deviation, stationary), GPS validation, group reference position calculation, distance calculation, confirmation timer management, alert creation and resolution, broadcast, push notification triggering.

**Frontend Responsibilities:** Alert UI, acknowledge button, alert history, separation status indicator on map markers, regrouping notification display.

**Database Requirements:** `ride_alerts` table (id, ride_id, type, user_id, location_lat, location_lng, detected_distance, severity, message, status, created_at, resolved_at, resolved_by); `ride_participants` separation status fields (separation_status, separation_distance, separation_timer_started_at, regrouping_timer_started_at).

**REST API:** `GET /api/v1/rides/{id}/alerts`, `GET /api/v1/rides/{id}/separation-status`, `PATCH /api/v1/alerts/{id}/resolve`.

**WebSocket:** `server → client: alert.new`, `alert.resolved`, `separation.status.update`, `separation.regrouped`.

**Technologies:** Python, Django, Django Channels, PostgreSQL + PostGIS (ST_Distance), WebSocket.

**Dependencies:** Module 5 (locations), Module 6 (route), Module 11 (push notifications).

**Validation Rules:** Alert type in enum (separation_warning, high_separation, regrouped), status in (active, resolved), severity in (warning, high, info), user is ride participant, distance ≥ 0.

**Error Handling:** Algorithm errors logged, don't block location processing; missing route → skip deviation check; GPS unavailable → use last valid location, mark as stale; disconnected participant → mark as connectivity issue, not confirmed separation.

**Security:** Alerts visible only to ride members; separation status not exposed publicly.

**Testing Scenarios:** Separation detected, deviation detected, stationary detected, alert resolved, false positive prevention, GPS accuracy below threshold, stale location, disconnected participant, multiple riders at different distances, repeated threshold crossing, automatic regrouping.

---

## Module 8: Intelligent Regrouping and Meeting Point Recommendation

**Purpose:** Recommend meeting points when participants are separated from the group.

**Working Process:**
1. Separation Warning or High Separation Alert triggers regrouping flow (or leader requests manually).
2. Backend identifies separated participants (those beyond 500 m threshold from group).
3. Backend queries Overpass API for nearby POIs (fuel stations, restaurants, parking) around each separated participant's location.
4. For each candidate POI, OSRM calculates route + ETA from each participant.
5. Ranking algorithm: minimize max travel time (minimize worst-case reunite time), prefer POI with amenities.
6. Top recommendation presented to Ride Leader; leader approves/rejects.
7. Approved point shared with all participants via WebSocket; displayed on map.
8. Automatic regrouping detection (Module 7) monitors when separated rider returns within 300 m for 30 seconds to resolve alert.

**User Interactions:** Meeting point recommendation screen, Approve/Reject buttons, Map with candidate markers.

**Inputs:** Separated participant locations, ride ID, POI preferences.

**Outputs:** Ranked meeting point candidates, approved meeting point.

**Functional Requirements:** FR-023, FR-024, FR-025, FR-039.

**Backend Responsibilities:** POI search (Overpass), route calculation (OSRM), ranking algorithm, recommendation storage, leader approval workflow, regrouping detection integration.

**Frontend Responsibilities:** Candidate display, leader decision UI, map markers for candidates/approved point, regrouped status indicator.

**Database Requirements:** `meeting_point_recommendations` table (id, ride_id, alert_id, name, lat, lng, type, address, rank_score, max_eta, status, created_at, approved_by, approved_at, expires_at).

**REST API:** `GET /api/v1/rides/{id}/meeting-points`, `POST /api/v1/meeting-points/{id}/approve`, `POST /api/v1/meeting-points/{id}/reject`.

**WebSocket:** `server → client: meeting-point.recommended`, `meeting-point.approved`, `separation.regrouped`.

**Technologies:** Python, Overpass API, OSRM, PostgreSQL + PostGIS, Django REST Framework, Django Channels.

**Dependencies:** Module 7 (separation alerts), Module 5 (locations), Module 6 (route/OSRM).

**Validation Rules:** Recommendation tied to active ride, leader-only approve/reject, expiry enforcement.

**Error Handling:** Overpass/OSRM unavailable → return error, leader can manually enter meeting point; no candidates → notify leader.

**Security:** Recommendations visible only to ride members; approval restricted to leader.

**Testing Scenarios:** Recommendation generated, leader approves, leader rejects, no POIs found, OSRM unavailable, expiry, automatic regrouping resolution.

---

## Module 9: Ride Event and Stop Management

**Purpose:** Share ride events (fuel, food, rest) with group.

**Working Process:**
1. Participant creates event: type, location, optional note, optional planned time.
2. Backend stores in `ride_events` table, broadcasts via WebSocket to ride group.
3. All participants see event marker on map and in event list.
4. Events can be marked completed; history preserved after ride ends.

**User Interactions:** Create Event screen, Event List screen, Event map markers.

**Inputs:** Event type (enum), location, note, planned time.

**Outputs:** Event object, broadcast notification.

**Functional Requirements:** FR-026, FR-027.

**Backend Responsibilities:** Event CRUD, broadcast, status management.

**Frontend Responsibilities:** Event creation form, event list, map markers.

**Database Requirements:** `ride_events` table (id, ride_id, user_id, type, lat, lng, address, note, planned_time, status, created_at, completed_at).

**REST API:** `POST /api/v1/rides/{id}/events`, `GET /api/v1/rides/{id}/events`, `PATCH /api/v1/events/{id}`.

**WebSocket:** `server → client: event.new`, `event.updated`.

**Technologies:** Django REST Framework, Django Channels, PostgreSQL + PostGIS.

**Dependencies:** Module 4 (ride membership), Module 5 (locations).

**Validation Rules:** Event type in enum, user is active ride participant, location valid.

**Error Handling:** Event creation failure → retry; broadcast failure → event still stored, sync later.

**Security:** Events visible only to ride members; creator can edit/delete own events.

**Testing Scenarios:** Create fuel stop event, view event list, mark completed, event map marker, non-member access denied.

---

## Module 10: Emergency and Breakdown Management

**Purpose:** Report breakdowns and emergencies to group.

**Working Process:**
1. Participant reports emergency/breakdown: type, description, location (auto or manual), severity.
2. Backend creates `emergency_reports` record with `status = 'active'`, high priority.
3. Immediate WebSocket broadcast to all ride participants + high-priority push notification.
4. Ride Leader and participants can coordinate response; status updated (acknowledged, resolved, false_alarm).
5. Reporter can update status; incident logged in ride history.

**User Interactions:** Emergency Report screen, Emergency alert display, Status update.

**Inputs:** Emergency type, description, location, severity.

**Outputs:** Emergency report, broadcast alert, push notification.

**Functional Requirements:** FR-028.

**Backend Responsibilities:** Emergency report creation, immediate broadcast, push notification trigger, status lifecycle.

**Frontend Responsibilities:** Emergency reporting UI (prominent, accessible), alert display, status updates.

**Database Requirements:** `emergency_reports` table (id, ride_id, user_id, type, severity, description, lat, lng, status, created_at, acknowledged_at, resolved_at, resolved_by).

**REST API:** `POST /api/v1/rides/{id}/emergencies`, `GET /api/v1/rides/{id}/emergencies`, `PATCH /api/v1/emergencies/{id}`.

**WebSocket:** `server → client: emergency.new` (high priority), `emergency.updated`.

**Technologies:** Django REST Framework, Django Channels, Expo Notifications (push), PostgreSQL + PostGIS.

**Dependencies:** Module 4 (membership), Module 11 (notifications), Module 5 (location).

**Validation Rules:** Emergency type in enum, user is active ride participant, severity in (low, medium, high, critical).

**Error Handling:** Push notification failure → WebSocket still delivers; offline participants see alert on reconnect.

**Security:** Emergency visible to all ride members; status updates by reporter or leader.

**Testing Scenarios:** Emergency reported, broadcast received, status updated, push notification delivered, false alarm resolution.

---

## Module 11: Push Notification Management

**Purpose:** Notify participants of important events when app is backgrounded/closed.

**Working Process:**
1. Backend triggers notification on events: new invitation, separation warning, high separation alert, deviation alert, stationary alert, meeting point recommended/approved, regrouping notification, new event, emergency, ride started/ended.
2. Backend calls Expo Notifications API (or FCM directly) with device token, title, body, data payload.
3. Frontend registers device token on login, handles incoming notifications (foreground + background).
4. Notification tapped → deep link to relevant ride/screen.
5. Notification history optionally stored in `notifications` table.

**User Interactions:** Notification display, notification settings, permission dialog.

**Inputs:** Device token, notification payload, user preferences.

**Outputs:** Push notifications delivered to devices.

**Functional Requirements:** FR-029, FR-038 (high separation push notification).

**Backend Responsibilities:** Notification triggering, device token management, Expo/FCM API integration, separation alert notification routing (Ride Leader + affected rider).

**Frontend Responsibilities:** Token registration, notification handling, permission requests, deep-link handling, separation alert display.

**Database Requirements:** `device_tokens` table (id, user_id, token, platform, created_at, last_used_at); optional `notifications` table.

**REST API:** `POST /api/v1/devices/tokens`, `GET /api/v1/notifications` (user's notification history).

**WebSocket:** Not applicable (push is separate channel).

**Technologies:** Expo Notifications, Firebase Cloud Messaging (FCM), Django.

**Dependencies:** Module 1 (users), all event modules.

**Validation Rules:** Device token format valid, platform in (ios, android), user owns token.

**Error Handling:** Invalid token → remove from DB; FCM/Expo API failure → log and retry; user disabled notifications → respect preference.

**Security:** Device tokens tied to user, never exposed publicly; push payloads contain no sensitive location data (ride IDs only).

**Testing Scenarios:** Token registration, notification on invitation, notification on separation warning, notification on high separation alert, notification on regrouping, tapped notification deep-link, disabled notifications respected.

---

## Module 12: Network Connectivity and Location Recovery

**Purpose:** Handle network interruptions gracefully; maintain location context; distinguish disconnected participants from separated participants.

**Working Process:**
1. Frontend monitors network status (NetInfo).
2. On disconnect: WebSocket closes, app shows "Offline" indicator, continues collecting GPS locally (cached).
3. Last known location + timestamp retained; other participants see "stale" marker with age.
4. On reconnect: WebSocket reconnects, re-authenticates, resubscribes to ride groups, sends buffered location updates.
5. Backend marks disconnected participants; clears stale status after configurable threshold (e.g., 5 minutes no update).
6. **Separation distinction:** A disconnected participant (no location updates) is NOT classified as separated. Separation requires valid GPS readings exceeding distance thresholds. Disconnected participants are shown as "location unavailable" rather than "separated."
7. On reconnection, buffered locations are processed for separation evaluation using their original timestamps.

**User Interactions:** Connection status indicator, stale location marker, reconnecting UI, location unavailable indicator.

**Inputs:** Network status events, WebSocket connect/disconnect, buffered locations.

**Outputs:** Connection status UI, stale indicators, resync of locations, separation status (not triggered by disconnect alone).

**Functional Requirements:** FR-030, FR-031, FR-043.

**Backend Responsibilities:** WebSocket connection management, disconnect detection, stale location marking, reconnection handling, separation evaluation using buffered locations with original timestamps.

**Frontend Responsibilities:** Network monitoring, local cache, reconnection logic, stale UI, location unavailable indicator.

**Database Requirements:** `location_updates` (or latest location cache) with `server_received_at` for staleness calculation.

**REST API:** Not primary (WebSocket-driven).

**WebSocket:** Reconnect + resume, heartbeat/ping-pong, separation status update.

**Technologies:** React Native NetInfo, Django Channels, Redis.

**Dependencies:** Module 5 (location tracking), Module 7 (monitoring).

**Validation Rules:** Reconnection requires valid JWT; ride membership re-verified on reconnect; disconnected participant not classified as separated.

**Error Handling:** Prolonged disconnect → participant shown as offline/location unavailable; auto-reconnect with exponential backoff; buffered locations processed on reconnect with original timestamps.

**Security:** Re-authentication required on reconnect; no session hijacking via reconnect.

**Testing Scenarios:** Network disconnect, stale location shown, reconnect + resync, prolonged offline → offline status, disconnected participant not classified as separated, buffered location processing on reconnect.

---

## Module 13: Ride Completion and History Management

**Purpose:** End rides and maintain completed ride history.

**Working Process:**
1. Ride Leader ends ride: `rides.status = 'completed'`, `ended_at` set.
2. WebSocket broadcast ride ended; location sharing stops; WebSocket groups closed.
3. Ride moved to history; participants can view summary (duration, participants, events, alerts, route).
4. Historical location data subject to retention policy (NFR-010/NFR-011).

**User Interactions:** End Ride button (leader), Ride History screen, Completed ride detail.

**Inputs:** Ride ID, end ride action.

**Outputs:** Completed ride record, history list.

**Functional Requirements:** FR-012 (end), FR-032, FR-033.

**Backend Responsibilities:** Ride status transition, WebSocket group cleanup, history archival, retention enforcement.

**Frontend Responsibilities:** End ride confirmation, history list, ride detail view.

**Database Requirements:** `rides` (completed status), `ride_history` or query rides where status='completed'; participant records preserved.

**REST API:** `POST /api/v1/rides/{id}/end`, `GET /api/v1/rides/history`, `GET /api/v1/rides/{id}` (completed detail).

**WebSocket:** `server → client: ride.ended`.

**Technologies:** Django REST Framework, Django Channels, PostgreSQL.

**Dependencies:** Module 4 (ride management), Module 5/6/7/9/10 (historical data).

**Validation Rules:** Only leader can end, ride must be 'active' to end.

**Error Handling:** End ride failure → retry; WebSocket close errors handled gracefully.

**Security:** History visible only to ride participants.

**Testing Scenarios:** Leader ends ride, history view, participant cannot end ride, ride ended broadcast.
