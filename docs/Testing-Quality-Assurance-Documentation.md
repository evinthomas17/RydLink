# RydLink - Testing and Quality Assurance Documentation

## Table of Contents
1. [Testing Strategy](#1-testing-strategy)
2. [Unit Testing](#2-unit-testing)
3. [Backend API Testing](#3-backend-api-testing)
4. [Database Testing](#4-database-testing)
5. [WebSocket Testing](#5-websocket-testing)
6. [GPS Accuracy Testing](#6-gps-accuracy-testing)
7. [Background Location Testing](#7-background-location-testing)
8. [Rider Separation Detection Testing](#8-rider-separation-detection-testing)
9. [Regrouping Algorithm Testing](#9-regrouping-algorithm-testing)
10. [Notification Testing](#10-notification-testing)
11. [Integration Testing](#11-integration-testing)
12. [System Testing](#12-system-testing)
13. [Performance Testing](#13-performance-testing)
14. [Security Testing](#14-security-testing)
15. [User Acceptance Testing](#15-user-acceptance-testing)

---

## 1. Testing Strategy

### 1.1 Testing Objectives
- Verify all backend modules function correctly per specification
- Verify all frontend screens and components work as designed
- Verify frontend-backend integration is seamless
- Verify real-time WebSocket communication is reliable
- Verify GPS location tracking and separation detection accuracy
- Verify regrouping algorithm produces correct recommendations
- Verify emergency management and push notifications work correctly
- Verify data consistency and security across all modules
- Verify application performance under load
- Verify application meets all non-functional requirements

### 1.2 Testing Scope
**In Scope:**
- All backend Django apps and APIs
- All React Native screens and components
- WebSocket communication between frontend and backend
- PostgreSQL/PostGIS database operations
- Redis channel layer operations
- OSRM routing integration
- Overpass API integration
- Push notification integration
- GPS location tracking
- Separation detection algorithm
- Regrouping algorithm
- Emergency management
- Security and authentication

**Out of Scope:**
- Third-party service internal operations (OSRM, Overpass, FCM)
- Mobile device hardware testing
- Mobile network carrier testing
- Apple App Store / Google Play Store submission process

### 1.3 Testing Environment

| Environment | Purpose | Tools |
|-------------|---------|-------|
| Development | Unit testing, developer testing | Local Django, Expo, SQLite/PostgreSQL |
| Staging | Integration testing, UAT | Docker, PostgreSQL, Redis, OSRM |
| Production | Performance, security testing | Production-like environment |

### 1.4 Testing Tools

| Category | Tool | Purpose |
|----------|------|---------|
| Unit Testing (Backend) | pytest | Python unit tests |
| Unit Testing (Frontend) | Jest | JavaScript unit tests |
| API Testing | Postman | REST API manual testing |
| E2E Testing (Frontend) | Detox | End-to-end mobile testing |
| WebSocket Testing | websocket-client | WebSocket protocol testing |
| Database Testing | pgAdmin | Database inspection |
| Load Testing | Locust | Performance/load testing |
| Security Testing | OWASP ZAP | Security vulnerability scanning |
| Coverage | coverage.py | Code coverage measurement |
| Linting | ruff | Python linting |
| Type Checking | mypy | Python type checking |

### 1.5 Testing Methodology
- **Test Pyramid**: Unit tests (70%), Integration tests (20%), E2E tests (10%)
- **Test-Driven Development**: Write tests before implementation for critical algorithms
- **Continuous Integration**: GitHub Actions runs tests on every push
- **Regression Testing**: Run full test suite before each release

### 1.6 Testing Responsibilities

| Role | Responsibilities |
|------|-----------------|
| Backend Developer | Write unit tests for backend modules |
| Frontend Developer | Write unit tests for frontend components |
| QA Engineer | Write integration and E2E tests |
| DevOps Engineer | Configure CI/CD pipeline |
| Product Owner | Review UAT test cases |

### 1.7 Entry Criteria
- All code merged to feature branch
- Code review completed
- Build passes successfully
- Test environment ready
- Test data prepared

### 1.8 Exit Criteria
- All critical and high-priority test cases pass
- Code coverage ≥ 80% (backend), ≥ 70% (frontend)
- No critical or high-priority bugs open
- Performance benchmarks met
- Security vulnerabilities resolved

---

## 2. Unit Testing

### 2.1 Test Specifications

#### Authentication Logic
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-AUTH-01 | Authentication | User registration with valid data | 201 Created, user object returned |
| UT-AUTH-02 | Authentication | User registration with duplicate email | 400 Bad Request, validation error |
| UT-AUTH-03 | Authentication | User login with valid credentials | 200 OK, JWT tokens returned |
| UT-AUTH-04 | Authentication | User login with invalid credentials | 400 Bad Request, error message |
| UT-AUTH-05 | Authentication | Token refresh with valid refresh token | 200 OK, new access token |
| UT-AUTH-06 | Authentication | Token refresh with invalid token | 401 Unauthorized |
| UT-AUTH-07 | Authentication | Password hashing verification | Password matches hash |
| UT-AUTH-08 | Authentication | JWT access token expiry (15 min) | Token expires after 15 minutes |
| UT-AUTH-09 | Authentication | JWT refresh token expiry (7 days) | Token expires after 7 days |

#### Ride Creation
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-RIDE-01 | Rides | Create ride with valid data | Ride created, status=planned |
| UT-RIDE-02 | Rides | Create ride with invalid latitude (>90) | 400 Bad Request |
| UT-RIDE-03 | Rides | Create ride with invalid longitude (>180) | 400 Bad Request |
| UT-RIDE-04 | Rides | Create ride without name | 400 Bad Request, missing field |
| UT-RIDE-05 | Rides | Get ride details for participant | 200 OK, ride details |
| UT-RIDE-06 | Rides | Get ride details for non-participant | 403 Forbidden |

#### Invitation Management
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-INV-01 | Invitations | Send invitation to valid user | Invitation created, status=pending |
| UT-INV-02 | Invitations | Send duplicate invitation | 409 Conflict |
| UT-INV-03 | Invitations | Accept valid invitation | Invitation status=accepted |
| UT-INV-04 | Invitations | Reject invitation | Invitation status=rejected |
| UT-INV-05 | Invitations | Expired invitation (24h) | Invitation status=expired |
| UT-INV-06 | Invitations | Invite non-existent user | 404 Not Found |

#### Participant Management
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-PART-01 | Participants | Join ride as participant | Participant added, status=joined |
| UT-PART-02 | Participants | Leave ride | Participant status=left |
| UT-PART-03 | Participants | Update participant role (leader only) | Role updated |
| UT-PART-04 | Participants | Non-leader updating role | 403 Forbidden |
| UT-PART-05 | Participants | Get participant list for ride | 200 OK, list of participants |

#### Route Calculations
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-ROUTE-01 | Routes | Calculate route between valid coordinates | Route returned with distance/duration |
| UT-ROUTE-02 | Routes | Calculate route with invalid coordinates | None returned |
| UT-ROUTE-03 | Routes | Decode polyline geometry | Array of coordinate pairs |
| UT-ROUTE-04 | Routes | Store route in PostGIS | Route geometry stored correctly |

#### GPS Validation
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-GPS-01 | Location | Validate latitude within [-90, 90] | Valid accepted |
| UT-GPS-02 | Location | Validate latitude outside [-90, 90] | Rejected |
| UT-GPS-03 | Location | Validate longitude within [-180, 180] | Valid accepted |
| UT-GPS-04 | Location | Validate longitude outside [-180, 180] | Rejected |
| UT-GPS-05 | Location | Validate accuracy ≤ 30m | Accepted |
| UT-GPS-06 | Location | Validate accuracy > 30m | Rejected (use last valid) |
| UT-GPS-07 | Location | Validate timestamp freshness < 2 min | Accepted |
| UT-GPS-08 | Location | Validate timestamp > 2 min old | Marked stale |

#### Distance Calculations
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-DIST-01 | Monitoring | Calculate Haversine distance between two points | Distance in metres |
| UT-DIST-02 | Monitoring | Calculate distance using PostGIS ST_Distance | Distance in metres |
| UT-DIST-03 | Monitoring | Distance between same point = 0 | 0 metres |
| UT-DIST-04 | Monitoring | Distance between antipodal points | ~20,000 km |

#### Separation Detection
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-SEP-01 | Monitoring | Rider within 300m of group | Status=normal, no alert |
| UT-SEP-02 | Monitoring | Rider 300-500m of group | Status=monitoring, no alert |
| UT-SEP-03 | Monitoring | Rider >500m for <60s | Status=separation_warning, no alert yet |
| UT-SEP-04 | Monitoring | Rider >500m for ≥60s | Separation Warning alert generated |
| UT-SEP-05 | Monitoring | Rider >1000m for <30s | Status=high_separation, no alert yet |
| UT-SEP-06 | Monitoring | Rider >1000m for ≥30s | High Separation alert generated |
| UT-SEP-07 | Monitoring | Rider returns within 300m for 30s | Auto-resolve alert, status=normal |
| UT-SEP-08 | Monitoring | GPS accuracy >30m | Reading rejected, use last valid |
| UT-SEP-09 | Monitoring | Location timestamp >2min old | Location skipped in evaluation |

#### Regrouping Algorithm
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-REGP-01 | Regrouping | Get candidate POIs from Overpass API | List of POIs returned |
| UT-REGP-02 | Regrouping | Filter POIs by type (fuel, restaurant, parking) | Filtered list |
| UT-REGP-03 | Regrouping | Calculate OSRM travel time for each participant | ETA in seconds |
| UT-REGP-04 | Regrouping | Rank candidates by worst-case ETA | Sorted list |
| UT-REGP-05 | Regrouping | No suitable meeting point found | Empty list |
| UT-REGP-06 | Regrouping | Duplicate POI removal | Unique POIs only |

#### Emergency Management
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-EMER-01 | Emergencies | Create emergency report | Emergency created, status=active |
| UT-EMER-02 | Emergencies | Create breakdown report | Emergency created, type=breakdown |
| UT-EMER-03 | Emergencies | Acknowledge emergency | acknowledged_at set |
| UT-EMER-04 | Emergencies | Resolve emergency | status=resolved, resolved_at set |

#### Notification Logic
| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| UT-NOT-01 | Notifications | Register device token | Token stored in database |
| UT-NOT-02 | Notifications | Create notification record | Notification saved |
| UT-NOT-03 | Notifications | Mark notification as read | read_at set |
| UT-NOT-04 | Notifications | Get unread notifications | Only unread returned |

---

## 3. Backend API Testing

### 3.1 Test Cases

#### Registration
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-REG-01 | Auth | Valid registration | None | POST /api/auth/register/ with valid data | {email, password, phone, first_name, last_name} | 201 Created, user object | | |
| TC-API-REG-02 | Auth | Duplicate email | User already registered | POST /api/auth/register/ with existing email | {email: existing} | 400 Bad Request | | |
| TC-API-REG-03 | Auth | Missing required fields | None | POST /api/auth/register/ with missing email | {password: "test"} | 400 Bad Request, ERR_001 | | |
| TC-API-REG-04 | Auth | Invalid email format | None | POST /api/auth/register/ with invalid email | {email: "invalid"} | 400 Bad Request | | |

#### Login
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-LOG-01 | Auth | Valid login | User registered | POST /api/auth/login/ with valid credentials | {email, password} | 200 OK, tokens returned | | |
| TC-API-LOG-02 | Auth | Invalid password | User registered | POST /api/auth/login/ with wrong password | {email, wrong_password} | 400 Bad Request | | |
| TC-API-LOG-03 | Auth | Non-existent user | None | POST /api/auth/login/ with unknown email | {email: "nonexistent"} | 400 Bad Request | | |

#### Profile Management
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-PRF-01 | Auth | Get profile with valid token | User authenticated | GET /api/auth/profile/ with Bearer token | None | 200 OK, user data | | |
| TC-API-PRF-02 | Auth | Get profile without token | None | GET /api/auth/profile/ without Authorization | None | 401 Unauthorized | | |
| TC-API-PRF-03 | Auth | Update profile with valid data | User authenticated | PUT /api/auth/profile/ | {first_name: "New"} | 200 OK, updated user | | |

#### Ride Creation
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-RIDE-01 | Rides | Create ride with valid data | User authenticated | POST /api/rides/ | {name, start_lat, start_lng, dest_lat, dest_lng} | 201 Created, ride object | | |
| TC-API-RIDE-02 | Rides | Create ride with invalid lat | User authenticated | POST /api/rides/ | {start_lat: 91} | 400 Bad Request, ERR_007 | | |
| TC-API-RIDE-03 | Rides | Create ride without auth | None | POST /api/rides/ without token | {name: "Test"} | 401 Unauthorized | | |
| TC-API-RIDE-04 | Rides | Get ride details as participant | Ride exists, user is participant | GET /api/rides/{id}/ | None | 200 OK, ride details | | |
| TC-API-RIDE-05 | Rides | Get ride details as non-participant | Ride exists, user not participant | GET /api/rides/{id}/ | None | 403 Forbidden | | |

#### Invitations
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-INV-01 | Invitations | Send invitation as leader | Ride exists, user is leader | POST /api/rides/{id}/participants/ | {invitee_id} | 201 Created, invitation | | |
| TC-API-INV-02 | Invitations | Send invitation as participant | Ride exists, user is participant | POST /api/rides/{id}/participants/ | {invitee_id} | 403 Forbidden | | |
| TC-API-INV-03 | Invitations | Duplicate invitation | Invitation already sent | POST /api/rides/{id}/participants/ | {invitee_id: existing} | 409 Conflict | | |

#### Ride Participants
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-PART-01 | Participants | Get participant list | Ride exists, user is participant | GET /api/rides/{id}/participants/ | None | 200 OK, list | | |
| TC-API-PART-02 | Participants | Update status as participant | Ride exists, user is participant | PUT /api/participants/{ride_id}/status/ | {status: "joined"} | 200 OK, updated | | |
| TC-API-PART-03 | Participants | Update role as non-leader | Ride exists, user is participant | PUT /api/rides/{id}/participants/{user_id}/ | {role: "leader"} | 403 Forbidden | | |

#### Routes
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-ROUTE-01 | Routes | Get route for ride | Ride exists with route | GET /api/rides/{id}/route/ | None | 200 OK, route geometry | | |
| TC-API-ROUTE-02 | Routes | Route for ride without route | Ride exists, no route calculated | GET /api/rides/{id}/route/ | None | 404 Not Found | | |

#### Location Updates
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-LOC-01 | Location | Send valid location via WebSocket | Ride active, WebSocket connected | WS: location_update message | {lat, lng, accuracy: 5} | location_broadcast received | | |
| TC-API-LOC-02 | Location | Send invalid accuracy (>30m) | Ride active | WS: location_update with accuracy: 50 | {lat, lng, accuracy: 50} | Rejected, last valid used | | |
| TC-API-LOC-03 | Location | Send stale location | Ride active | WS: location_update with old timestamp | {lat, lng, timestamp: old} | Marked stale | | |

#### Separation Alerts
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-ALERT-01 | Alerts | Get alerts for ride | Ride has alerts | GET /api/alerts/?ride_id={id} | None | 200 OK, alerts list | | |
| TC-API-ALERT-02 | Alerts | Get specific alert | Alert exists | GET /api/alerts/{id}/ | None | 200 OK, alert details | | |

#### Regrouping
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-MEET-01 | Meeting Points | Get recommendations for ride | Ride has separation | GET /api/meeting-points/?ride_id={id} | None | 200 OK, recommendations | | |
| TC-API-MEET-02 | Meeting Points | Approve meeting point as leader | Recommendation exists, user is leader | PUT /api/meeting-points/{id}/ | {approved: true} | 200 OK, approved | | |
| TC-API-MEET-03 | Meeting Points | Approve as non-leader | Recommendation exists, user is participant | PUT /api/meeting-points/{id}/ | {approved: true} | 403 Forbidden | | |

#### Events
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-EVENT-01 | Events | Create ride event | Ride active | POST /api/events/ | {ride_id, type, lat, lng} | 201 Created, event | | |
| TC-API-EVENT-02 | Events | List events for ride | Events exist | GET /api/events/{ride_id}/ | None | 200 OK, events list | | |

#### Emergencies
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-EMER-01 | Emergencies | Create emergency report | Ride active | POST /api/emergencies/ | {ride_id, type, severity, lat, lng} | 201 Created | | |
| TC-API-EMER-02 | Emergencies | List emergencies for ride | Emergencies exist | GET /api/emergencies/{ride_id}/ | None | 200 OK, emergencies | | |

#### Notifications
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-NOT-01 | Notifications | Register device token | User authenticated | POST /api/device-token/ | {token, platform} | 200/201 Created | | |
| TC-API-NOT-02 | Notifications | Get notifications | User has notifications | GET /api/notifications/ | None | 200 OK, list | | |

#### Ride History
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-API-HIST-01 | History | Get ride history | Completed rides exist | GET /api/rides/{id}/history/ | None | 200 OK, history | | |
| TC-API-HIST-02 | History | Get history for non-participant | Ride exists, user not participant | GET /api/rides/{id}/history/ | None | 403 Forbidden | | |

### 3.2 Boundary Conditions
| Test Case ID | Module | Boundary Condition | Test Input | Expected Result |
|--------------|--------|-------------------|------------|-----------------|
| TC-BOUND-01 | Rides | Maximum participants (50) | 50 participants in ride | 200 OK, all listed |
| TC-BOUND-02 | Rides | Ride name max length (200 chars) | 200 character name | 201 Created |
| TC-BOUND-03 | Rides | Ride name 201 chars | 201 character name | 400 Bad Request |
| TC-BOUND-04 | Location | Accuracy exactly 30m | accuracy: 30 | Accepted |
| TC-BOUND-05 | Location | Accuracy 30.1m | accuracy: 30.1 | Rejected |
| TC-BOUND-06 | Separation | Distance exactly 500m | distance_m: 500 | Status=monitoring |
| TC-BOUND-07 | Separation | Distance 500.1m | distance_m: 500.1 | Status=separation_warning |
| TC-BOUND-08 | Separation | Timer exactly 60s | timer: 60 seconds | Alert generated |
| TC-BOUND-09 | Separation | Timer 59.9s | timer: 59.9 seconds | No alert |

---

## 4. Database Testing

### 4.1 Model Validation

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-01 | Models | User model validation | Email unique, password hashed |
| TC-DB-02 | Models | Ride model validation | Status constraint enforced |
| TC-DB-03 | Models | Participant unique constraint | (ride_id, user_id) unique |
| TC-DB-04 | Models | Invitation token unique | Token unique constraint |
| TC-DB-05 | Models | Device token unique | Token unique constraint |
| TC-DB-06 | Models | Route ride_id unique | One route per ride |

### 4.2 Primary and Foreign Key Constraints

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-FK-01 | DB | Delete ride cascades to participants | All participants deleted |
| TC-DB-FK-02 | DB | Delete ride cascades to invitations | All invitations deleted |
| TC-DB-FK-03 | DB | Delete ride cascades to location updates | All locations deleted |
| TC-DB-FK-04 | DB | Delete ride cascades to routes | Route deleted |
| TC-DB-FK-05 | DB | Delete ride cascades to alerts | All alerts deleted |
| TC-DB-FK-06 | DB | Delete ride cascades to events | All events deleted |
| TC-DB-FK-07 | DB | Delete ride cascades to emergencies | All emergencies deleted |

### 4.3 Relationship Validation

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-REL-01 | DB | User has many ride_participants | Query returns all participant records |
| TC-DB-REL-02 | DB | Ride has many location_updates | Query returns all location records |
| TC-DB-REL-03 | DB | Ride has one ride_route | Query returns single route |
| TC-DB-REL-04 | DB | User has many device_tokens | Query returns all device tokens |

### 4.4 Unique Constraints

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-UNQ-01 | DB | Duplicate user email | IntegrityError raised |
| TC-DB-UNQ-02 | DB | Duplicate participant (ride_id, user_id) | IntegrityError raised |
| TC-DB-UNQ-03 | DB | Duplicate invitation token | IntegrityError raised |
| TC-DB-UNQ-04 | DB | Duplicate device token | IntegrityError raised |
| TC-DB-UNQ-05 | DB | Duplicate route ride_id | IntegrityError raised |

### 4.5 Spatial Field Validation

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-GIS-01 | PostGIS | Insert location with valid geography | Point stored with SRID 4326 |
| TC-DB-GIS-02 | PostGIS | Insert route with valid LineString | LineString stored with SRID 4326 |
| TC-DB-GIS-03 | PostGIS | Calculate distance between two points | Distance in metres returned |
| TC-DB-GIS-04 | PostGIS | Spatial index usage | Query uses GiST index |

### 4.6 Database Transactions

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-TXN-01 | DB | Ride creation with participants (all succeed) | All records created |
| TC-DB-TXN-02 | DB | Ride creation with participants (one fails) | All records rolled back |
| TC-DB-TXN-03 | DB | Concurrent location updates | No data corruption |

### 4.7 Data Consistency

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-CONS-01 | DB | Participant status consistency | status matches joined_at/left_at |
| TC-DB-CONS-02 | DB | Alert status consistency | active alerts have no resolved_at |
| TC-DB-CONS-03 | DB | Emergency status consistency | resolved emergencies have resolved_at |

### 4.8 Migration Testing

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-MIG-01 | DB | Apply all migrations | All 12 tables created |
| TC-DB-MIG-02 | DB | Rollback migration | Tables removed in reverse order |
| TC-DB-MIG-03 | DB | PostGIS extension enabled | PostGIS functions available |

### 4.9 Index Testing

| Test Case ID | Module | Test Scenario | Expected Result |
|--------------|--------|---------------|-----------------|
| TC-DB-IDX-01 | DB | Query location_updates by ride_id | Uses lu_ride_user_time_idx |
| TC-DB-GIS-05 | PostGIS | Spatial query on location | Uses lu_location_gist GiST index |
| TC-DB-GIS-06 | PostGIS | Spatial query on route | Uses rr_geometry_gist GiST index |

---

## 5. WebSocket Testing

### 5.1 Connection Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-01 | WebSocket | Establish connection with valid token | Ride exists, user is participant | Connect WS with ?ride_id=&user_id=&token= | Valid JWT | Connection established, welcome message | | |
| TC-WS-02 | WebSocket | Connection with invalid token | None | Connect WS with invalid token | Invalid JWT | Connection closed, code 4001 | | |
| TC-WS-03 | WebSocket | Connection with missing ride_id | None | Connect WS without ride_id parameter | None | Connection rejected, error message | | |
| TC-WS-04 | WebSocket | Connection as non-participant | User not in ride | Connect WS with valid token, wrong ride_id | Valid JWT | Connection rejected, "Not a participant" | | |

### 5.2 Authentication Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-AUTH-01 | WebSocket | JWT token verification | None | Connect WS with expired token | Expired JWT | Connection closed, auth error | | |
| TC-WS-AUTH-02 | WebSocket | JWT token missing | None | Connect WS without token parameter | None | Connection rejected | | |

### 5.3 Ride Group Isolation Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-GRP-01 | WebSocket | Messages isolated by ride | Two rides, users in different rides | User A sends location_update in Ride 1 | Ride 1 location | User B in Ride 2 does not receive | | |
| TC-WS-GRP-02 | WebSocket | Broadcast within ride | Multiple users in same ride | User A sends location_update | Ride 1 location | All users in Ride 1 receive location_broadcast | | |

### 5.4 Location Broadcasting Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-LOC-01 | WebSocket | Location broadcast delivery | Ride active, WebSocket connected | Send location_update | {lat, lng, accuracy: 5} | location_broadcast received by all | | |
| TC-WS-LOC-02 | WebSocket | Location broadcast frequency | Ride active | Send 10 location updates per second | {lat, lng} | Server throttles, processes max 10/s | | |

### 5.5 Multiple Participants Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-MULT-01 | WebSocket | 10 participants in ride | 10 WebSocket connections | All send location updates | 10 sets of coordinates | All receive broadcast with 10 participants | | |
| TC-WS-MULT-02 | WebSocket | 50 participants (max) | 50 WebSocket connections | All connected simultaneously | 50 connections | All connections maintained | | |

### 5.6 Message Validation Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-VAL-01 | WebSocket | Invalid message type | WebSocket connected | Send {type: "invalid"} | {type: "invalid"} | Error response, connection continues | | |
| TC-WS-VAL-02 | WebSocket | Missing required fields | WebSocket connected | Send {type: "location_update"} without lat/lng | {type: "location_update"} | Error response | | |
| TC-WS-VAL-03 | WebSocket | Out of range coordinates | WebSocket connected | Send lat: 91 | {lat: 91, lng: 0} | Error response | | |

### 5.7 Disconnection Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-DISC-01 | WebSocket | Client disconnects | WebSocket connected | Close WebSocket connection | None | Participant marked as left | | |
| TC-WS-DISC-02 | WebSocket | Server closes connection | WebSocket connected | Server initiates close | None | Client receives close event | | |

### 5.8 Reconnection Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-RECON-01 | WebSocket | Reconnect after network loss | WebSocket connected | Disconnect network, wait, reconnect | None | Connection re-established, group rejoined | | |
| TC-WS-RECON-02 | WebSocket | Max reconnection attempts | WebSocket disconnected | Attempt reconnection 5+ times | None | Stop after 5 attempts, show manual retry | | |

### 5.9 Unauthorized Connection Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-UNAUTH-01 | WebSocket | Unauthorized access attempt | None | Connect WS without authentication | None | Connection rejected | | |

### 5.10 Redis Channel Layer Testing

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-WS-REDIS-01 | WebSocket | Message broadcast via Redis | Redis running, multiple workers | Publish message to ride group | Message | All workers receive message | | |
| TC-WS-REDIS-02 | WebSocket | Redis unavailable fallback | Redis stopped | Start Django with single worker | None | In-memory channel layer used | | |

---

## 6. GPS Accuracy Testing

### 6.1 Valid GPS Readings

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-01 | GPS | Valid coordinates with accuracy ≤ 30m | GPS enabled | Send location with accuracy: 5m | {lat: 40.7128, lng: -74.0060, accuracy: 5} | Accepted, stored | | |
| TC-GPS-02 | GPS | Valid coordinates with accuracy exactly 30m | GPS enabled | Send location with accuracy: 30m | {lat: 40.7128, lng: -74.0060, accuracy: 30} | Accepted, stored | | |

### 6.2 Inaccurate Readings

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-03 | GPS | Accuracy > 30m | GPS enabled | Send location with accuracy: 45m | {lat: 40.7128, lng: -74.0060, accuracy: 45} | Rejected, last valid used | | |
| TC-GPS-04 | GPS | Negative accuracy | GPS enabled | Send location with accuracy: -5 | {lat: 40.7128, lng: -74.0060, accuracy: -5} | Rejected | | |

### 6.3 Missing Coordinates

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-05 | GPS | Missing latitude | GPS enabled | Send location without lat | {lng: -74.0060} | Rejected | | |
| TC-GPS-06 | GPS | Missing longitude | GPS enabled | Send location without lng | {lat: 40.7128} | Rejected | | |
| TC-GPS-07 | GPS | NaN coordinates | GPS enabled | Send location with NaN | {lat: NaN, lng: NaN} | Rejected | | |

### 6.4 Delayed Updates

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-08 | GPS | Delayed location update | GPS enabled | Send location with 5 min old timestamp | {timestamp: now-5min} | Marked stale | | |

### 6.5 Stale Locations

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-09 | GPS | Stale location (>2 min) | GPS enabled | Send location with 3 min old timestamp | {timestamp: now-3min} | Skipped in evaluation | | |
| TC-GPS-10 | GPS | Fresh location (<2 min) | GPS enabled | Send location with 1 min old timestamp | {timestamp: now-1min} | Accepted | | |

### 6.6 GPS Permission Denial

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-11 | GPS | Permission denied | User denies location permission | Attempt to get location | None | Error, location unavailable | | |

### 6.7 Background Location

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-12 | GPS | Background location updates | App in background, permission granted | Minimize app, wait 60s | None | Location updates every 30-60s | | |

### 6.8 Network Interruption

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-GPS-13 | GPS | Network interruption during tracking | GPS tracking active | Disable network, wait, re-enable | None | Locations stored locally, synced later | | |

---

## 7. Background Location Testing

### 7.1 Application in Foreground

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-01 | Background | App in foreground, ride active | Ride started | Use app normally | None | Location updates every 5-10s | | |

### 7.2 Application Minimized

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-02 | Background | App minimized, ride active | Ride started | Press home button | None | Location updates every 30-60s | | |

### 7.3 Screen Locked

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-03 | Background | Screen locked, ride active | Ride started | Lock device screen | None | Background tracking continues | | |

### 7.4 Application Restarted

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-04 | Background | App restarted during ride | Ride active | Close and reopen app | None | Tracking resumes, WebSocket reconnects | | |

### 7.5 Network Unavailable

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-05 | Background | Network unavailable during ride | Ride active | Enable airplane mode | None | Locations stored locally, synced later | | |

### 7.6 Location Permission Revoked

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-06 | Background | Location permission revoked | Ride active | Revoke location permission in settings | None | Tracking stops, user notified | | |

### 7.7 Android Battery Restrictions

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-07 | Background | Android battery optimization enabled | Android device | Enable battery optimization for app | None | Tracking may be limited by OS | | |

### 7.8 Recovery After Interruption

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-BG-08 | Background | Recovery after network restored | Network was unavailable | Re-enable network | None | Pending locations synced, tracking resumes | | |

---

## 8. Rider Separation Detection Testing

### Approved Threshold Configuration

| Condition | Distance | Confirmation Duration |
|-----------|----------|----------------------|
| Normal | 0–300 m | — |
| Monitoring | 300–500 m | — |
| Separation Warning | >500 m | 60 seconds |
| High Separation | >1,000 m | 30 seconds |
| Regrouped | Within 300 m | 30 seconds |

### 8.1 Test Cases

#### Normal Distance
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-01 | Separation | Rider within 300m | Ride active, group reference known | Set rider distance to 200m | distance: 200m | Status=normal, no alert | | |

#### Monitoring Distance
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-02 | Separation | Rider 300-500m, moving | Ride active | Set rider distance to 400m | distance: 400m | Status=monitoring, no alert | | |

#### Separation Warning
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-03 | Separation | Rider >500m, <60s | Ride active | Set distance to 520m, wait 30s | distance: 520m, time: 30s | Status=separation_warning, no alert yet | | |
| TC-S-04 | Separation | Rider >500m, ≥60s | Ride active | Set distance to 520m, wait 60s+ | distance: 520m, time: 60s+ | Separation Warning alert generated | | |

#### High Separation
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-05 | Separation | Rider >1000m, <30s | Ride active | Set distance to 1100m, wait 15s | distance: 1100m, time: 15s | Status=high_separation, no alert yet | | |
| TC-S-06 | Separation | Rider >1000m, ≥30s | Ride active | Set distance to 1100m, wait 30s+ | distance: 1100m, time: 30s+ | High Separation alert generated | | |

#### Confirmation Duration
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-07 | Separation | Timer pauses below threshold | Rider in warning zone, moves closer | Move rider from 520m to 400m | distance: 400m | Timer pauses | | |
| TC-S-08 | Separation | Timer resumes above threshold | Rider was in warning, moves away | Move rider from 400m to 520m | distance: 520m | Timer resumes | | |

#### GPS Accuracy Failure
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-09 | Separation | GPS accuracy >30m | Ride active | Send location with accuracy: 45m | accuracy: 45m | Reading rejected, use last valid | | |

#### Stale Location
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-10 | Separation | Stale location >2min | Ride active | Send location with old timestamp | timestamp: now-3min | Location skipped in evaluation | | |

#### Disconnection
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-11 | Separation | Disconnected participant | Rider WebSocket disconnected | Disconnect WebSocket | None | Not classified as separated, use last known | | |

#### Duplicate Alerts
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-12 | Separation | Duplicate alert prevention | Alert already active | Keep rider >500m for 120s | distance: 520m, time: 120s | Only one alert generated | | |

#### Automatic Regrouping
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-13 | Separation | Auto-resolve on regroup | Alert active, rider returns | Move rider within 300m for 30s | distance: 200m, time: 30s | Alert resolved, status=normal | | |

#### Multiple Participants
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-14 | Separation | Multiple riders different zones | 3 riders, different distances | Rider A: 200m, Rider B: 400m, Rider C: 600m | Multiple distances | Each rider correct status | | |

#### Boundary Conditions
| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-S-15 | Separation | Exactly 300m boundary | Ride active | Set distance to exactly 300m | distance: 300m | Status=normal | | |
| TC-S-16 | Separation | Exactly 500m boundary | Ride active | Set distance to exactly 500m | distance: 500m | Status=monitoring | | |
| TC-S-17 | Separation | Exactly 1000m boundary | Ride active | Set distance to exactly 1000m | distance: 1000m | Status=separation_warning | | |

---

## 9. Regrouping Algorithm Testing

### 9.1 Test Cases

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-REGP-01 | Regrouping | Candidate meeting point retrieval | Overpass API available | Query POIs near group centroid | centroid coordinates | List of POIs returned | | |
| TC-REGP-02 | Regrouping | POI filtering by type | POIs retrieved | Filter by fuel, restaurant, parking | List of POIs | Filtered list by type | | |
| TC-REGP-03 | Regrouping | OSRM travel time calculation | OSRM available | Calculate route for each participant | Participant coords, POI coords | ETA in seconds | | |
| TC-REGP-04 | Regrouping | Ranking algorithm | Multiple candidates | Rank by worst-case ETA | Candidates with ETAs | Sorted list (best first) | | |
| TC-REGP-05 | Regrouping | Multiple participant scenarios | 3+ participants | Calculate ETAs for all | Multiple participants | Worst-case ETA per candidate | | |
| TC-REGP-06 | Regrouping | Equal travel time | Two candidates with same ETA | Compare ranking | Equal ETAs | Secondary sort by average ETA | | |
| TC-REGP-07 | Regrouping | Unavailable routes | OSRM returns no route | Candidate with no valid route | Candidate unreachable | Candidate filtered out | | |
| TC-REGP-08 | Regrouping | No suitable meeting point | All candidates unavailable | No valid routes | All candidates unreachable | Empty recommendation list | | |
| TC-REGP-09 | Regrouping | Ride Leader approval | Recommendation generated | Leader approves candidate | {approved: true} | Recommendation status=approved | | |

---

## 10. Notification Testing

### 10.1 Test Cases

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-NOT-01 | Notifications | In-app notification display | App foreground | Trigger separation alert | Alert generated | Notification shown in app | | |
| TC-NOT-02 | Notifications | WebSocket notification delivery | WebSocket connected | Generate separation alert | Alert created | WS message received | | |
| TC-NOT-03 | Notifications | Push notification delivery | App background | Generate emergency | Emergency created | Push notification received | | |
| TC-NOT-04 | Notifications | Invitation notification | Invitation sent | Send ride invitation | Invitation created | Notification to invitee | | |
| TC-NOT-05 | Notifications | Separation alert notification | Rider separated | Generate separation warning | Rider >500m | Notification to leader + rider | | |
| TC-NOT-06 | Notifications | Emergency alert notification | Emergency reported | Generate emergency | Emergency created | High-priority notification | | |
| TC-NOT-07 | Notifications | Read/unread status | Notification exists | Mark notification as read | Notification ID | read_at set | | |
| TC-NOT-08 | Notifications | Duplicate notification prevention | Alert already sent | Generate same alert twice | Same alert conditions | Only one notification | | |
| TC-NOT-09 | Notifications | Delivery failure | Push service unavailable | Send push notification | Invalid token | Error logged, retry attempted | | |

---

## 11. Integration Testing

### 11.1 Test Cases

#### React Native and Django REST APIs

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-01 | Integration | Register user via frontend | Frontend + Backend | Submit registration form | Valid user data | User created, redirected to login | | |
| TC-INT-02 | Integration | Login via frontend | User registered | Submit login form | Valid credentials | Tokens stored, dashboard shown | | |
| TC-INT-03 | Integration | Create ride via frontend | User authenticated | Fill ride creation form | Valid ride data | Ride created, lobby shown | | |
| TC-INT-04 | Integration | Invite participant via frontend | Ride exists, user is leader | Select user, send invitation | Invitee ID | Invitation sent, status pending | | |
| TC-INT-05 | Integration | Accept invitation via frontend | Invitation received | Tap accept on invitation | Invitation token | Invitation accepted, ride joined | | |

#### React Native and Django Channels

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-06 | Integration | WebSocket connection from app | App installed, backend running | Connect WebSocket from app | Valid token | Connection established | | |
| TC-INT-07 | Integration | Location sharing via WebSocket | Ride active | Send location from app | GPS coordinates | Broadcast to all participants | | |
| TC-INT-08 | Integration | Separation alert via WebSocket | Rider separated | Trigger separation detection | Rider >500m | Alert broadcast to leader+rider | | |

#### Django and PostgreSQL

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-09 | Integration | Store ride in database | Ride created | Save ride via API | Ride data | Ride persisted in PostgreSQL | | |
| TC-INT-10 | Integration | Store location with PostGIS | Location update received | Save location with geography | Coordinates | Point stored with SRID 4326 | | |
| TC-INT-11 | Integration | Query spatial data | Locations stored | Query nearby participants | Reference point | Results within radius | | |

#### Django Channels and Redis

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-12 | Integration | Broadcast via Redis channel layer | Redis running, multiple workers | Publish to ride group | Message | All workers receive message | | |
| TC-INT-13 | Integration | WebSocket group management | Redis running | Join/leave ride group | ride_id | Channel added/removed from group | | |

#### MapLibre and Map Tile Providers

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-14 | Integration | Load OSM tiles | Internet available | Initialize MapLibre with OSM | None | Map tiles load, attribution shown | | |
| TC-INT-15 | Integration | Offline map behavior | No internet | Open map screen | None | Error state, cached data if available | | |

#### Backend and OSRM

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-16 | Integration | Calculate route via OSRM | OSRM running | Request route between coordinates | Start/dest coords | Route with geometry, distance, duration | | |
| TC-INT-17 | Integration | OSRM unavailable | OSRM down | Request route | Start/dest coords | Error handled gracefully | | |

#### Backend and Overpass API

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-18 | Integration | Discover POIs via Overpass | Overpass running | Query nearby POIs | Centroid, radius | List of POIs returned | | |
| TC-INT-19 | Integration | Overpass unavailable | Overpass down | Query nearby POIs | Centroid, radius | Empty list, error logged | | |

#### Backend and Push Notification Services

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-INT-20 | Integration | Send push notification via Expo | Expo configured | Trigger emergency | Emergency data | Push notification sent to device | | |
| TC-INT-21 | Integration | Send push notification via FCM | FCM configured (Android) | Trigger emergency | Emergency data | Push notification sent via FCM | | |

---

## 12. System Testing

### 12.1 End-to-End Test Scenarios

#### Complete Ride Lifecycle

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-E2E-01 | System | User registration and login | None | Register, then login | Valid user data | User authenticated, tokens stored | | |
| TC-E2E-02 | System | Ride creation | User logged in | Create ride with start/dest | Valid ride data | Ride created, status=planned | | |
| TC-E2E-03 | System | Invitation | Ride exists, leader | Invite participant | Invitee ID | Invitation sent, status=pending | | |
| TC-E2E-04 | System | Invitation acceptance | Invitation pending | Accept invitation | Invitation token | Participant joined ride | | |
| TC-E2E-05 | System | Ride lobby | Participants joined | View ride lobby | None | Participant list displayed | | |
| TC-E2E-06 | System | Ride starting | Ride in lobby, leader | Start ride | None | Ride status=active, tracking starts | | |
| TC-E2E-07 | System | Live location sharing | Ride active | Share GPS location | Coordinates | Location broadcast to all | | |
| TC-E2E-08 | System | Participant tracking | Multiple participants | View map with markers | None | All participants visible on map | | |
| TC-E2E-09 | System | Separation detection | Rider moves away | Monitor separation | Rider >500m | Separation warning after 60s | | |
| TC-E2E-10 | System | Alert generation | Separation detected | Verify alert generated | Rider >500m | Alert broadcast, notification sent | | |
| TC-E2E-11 | System | Regrouping recommendation | Separation active | Request meeting point | None | Candidates displayed with ETAs | | |
| TC-E2E-12 | System | Ride events | Ride active | Create fuel stop event | Event data | Event created, broadcast | | |
| TC-E2E-13 | System | Emergency reporting | Emergency occurs | Report emergency | Emergency data | Emergency broadcast, notification | | |
| TC-E2E-14 | System | Ride completion | Ride active, leader | End ride | None | Ride status=completed, archived | | |
| TC-E2E-15 | System | Ride history | Ride completed | View ride history | None | Historical ride details displayed | | |

---

## 13. Performance Testing

### 13.1 Test Cases

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-PERF-01 | Performance | Multiple simultaneous participants | 50 participants | All send location updates | 50 connections | All connections maintained, <500ms latency | | |
| TC-PERF-02 | Performance | GPS update load | 10 participants, 10s interval | Continuous location updates | 10 participants × 6 updates/min | Database handles load, no errors | | |
| TC-PERF-03 | Performance | WebSocket message load | 50 participants | Broadcast location updates | 50 messages/sec | All messages delivered, no loss | | |
| TC-PERF-04 | Performance | Database query performance | 1000 rides | Query ride participants | ride_id | Query <800ms (95th percentile) | | |
| TC-PERF-05 | Performance | API response time | 100 concurrent users | Various API endpoints | Standard requests | Response <800ms (95th percentile) | | |
| TC-PERF-06 | Performance | Redis performance | 100 WebSocket connections | Broadcast messages | 100 messages/sec | Redis handles load, no errors | | |
| TC-PERF-07 | Performance | Map rendering performance | 50 markers on map | Render map with markers | 50 participants | Map renders smoothly, <2s load | | |
| TC-PERF-08 | Performance | Battery consumption | 2 hours tracking | Measure battery drain | Continuous tracking | <15% drain per hour | | |
| TC-PERF-09 | Performance | Network usage | 1 hour ride | Measure data transferred | Location updates | <5MB data per hour | | |

### 13.2 Performance Benchmarks

| Metric | Target | Measurement Tool |
|--------|--------|-----------------|
| WebSocket message latency | <500ms | WebSocket client timing |
| API response time (95th percentile) | <800ms | Postman/Newman |
| Concurrent rides supported | 1000+ | Load testing |
| Participants per ride | 50 max | WebSocket connection test |
| Separation alert delivery latency | <1 second | Alert generation timing |
| Location update frequency (foreground) | 5-10 seconds | GPS tracking |
| Location update frequency (background) | 30-60 seconds | Background tracking |
| Battery drain (1 hour tracking) | <15% | Battery monitoring |
| Map load time | <2 seconds | App startup timing |

---

## 14. Security Testing

### 14.1 Test Cases

#### Authentication

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-01 | Security | JWT token validation | Token generated | Use expired token | Expired JWT | 401 Unauthorized | | |
| TC-SEC-02 | Security | JWT token tampering | Token generated | Modify token payload | Tampered JWT | 401 Unauthorized | | |
| TC-SEC-03 | Security | Missing authentication | None | Access protected endpoint without token | None | 401 Unauthorized | | |

#### Authorization

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-04 | Security | Participant accessing leader-only endpoint | User is participant | Try to start ride as participant | Ride ID | 403 Forbidden | | |
| TC-SEC-05 | Security | User accessing another user's ride | User not in ride | Access ride details | Another ride's ID | 403 Forbidden | | |
| TC-SEC-06 | Security | Ride membership verification | User not participant | Send location update for ride | Ride ID | Rejected, not participant | | |

#### JWT Validation

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-07 | Security | Expired access token | Token expired | Access API with expired token | Expired JWT | 401 Unauthorized | | |
| TC-SEC-08 | Security | Refresh token rotation | Token refreshed | Use old refresh token | Old refresh token | 401 Unauthorized | | |

#### Password Security

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-09 | Security | Password hashing | User registers | Verify password stored as hash | Plain password | Hash stored, not plain text | | |
| TC-SEC-10 | Security | Weak password rejection | None | Register with weak password | {password: "123"} | 400 Bad Request | | |

#### API Access Control

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-11 | Security | SQL injection attempt | None | Send malicious SQL in request | {name: "'; DROP TABLE"} | 400 Bad Request, no DB damage | | |
| TC-SEC-12 | Security | XSS attempt | None | Send script in user input | {name: "<script>alert(1)</script>"} | 200 OK, script escaped in response | | |

#### WebSocket Security

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-13 | Security | WebSocket without authentication | None | Connect WS without token | None | Connection rejected | | |
| TC-SEC-14 | Security | Cross-ride message access | User in Ride 1 | Try to receive Ride 2 messages | Ride 2 ID | Messages not received | | |

#### Location Privacy

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-15 | Security | Unauthorized location access | User not in ride | Request location data for ride | Ride ID | 403 Forbidden | | |
| TC-SEC-16 | Security | Location data retention | Ride completed | Verify location data purged | Completed ride | Locations older than 90 days removed | | |

#### Secret Management

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-17 | Security | Secret not in code | None | Search codebase for hardcoded secrets | None | No secrets in source code | | |
| TC-SEC-18 | Security | Environment variables used | None | Verify .env used for configuration | None | All secrets in .env | | |

#### Unauthorized Location Access

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-19 | Security | Location spoofing | None | Send location with fake timestamp | {timestamp: future} | Rejected, timestamp validated | | |

#### Data Protection

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-SEC-20 | Security | Data encryption in transit | None | Access API over HTTPS | None | All traffic encrypted | | |
| TC-SEC-21 | Security | Sensitive data not logged | None | Check application logs | None | No passwords/tokens in logs | | |

---

## 15. User Acceptance Testing

### 15.1 Test Scenarios

#### Ride Leader

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-01 | UAT | Create a ride | User logged in | Fill ride creation form | Valid ride data | Ride created, appears in lobby | | |
| TC-UAT-02 | UAT | Invite participants | Ride created | Select users from search | Invitee IDs | Invitations sent | | |
| TC-UAT-03 | UAT | Start the ride | Ride in lobby | Tap start ride | None | Ride status=active | | |
| TC-UAT-04 | UAT | Monitor participants | Ride active | View map | None | All participants visible | | |
| TC-UAT-05 | UAT | Receive separation alerts | Rider separated | Wait for alert | Separation detected | Alert received, banner shown | | |
| TC-UAT-06 | UAT | Approve meeting point | Recommendation generated | Tap approve | Candidate selected | Meeting point approved, shared | | |
| TC-UAT-07 | UAT | Handle emergency | Emergency reported | View emergency banner | Emergency active | Acknowledge/resolve options | | |
| TC-UAT-08 | UAT | End the ride | Ride active | Tap end ride | None | Ride completed, archived | | |

#### Ride Participant

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-09 | UAT | Receive invitation | Invitation sent | View invitations | None | Invitation displayed | | |
| TC-UAT-10 | UAT | Accept invitation | Invitation pending | Tap accept | Invitation token | Joined ride | | |
| TC-UAT-11 | UAT | Join ride lobby | Joined ride | View lobby | None | Ride details, participant list | | |
| TC-UAT-12 | UAT | Share location | Ride active | Location sharing enabled | None | Location broadcast to group | | |
| TC-UAT-13 | UAT | View participant tracking | Ride active | View map | None | All participants on map | | |
| TC-UAT-14 | UAT | Receive separation alert | Rider separated | Wait for notification | Separation detected | Alert banner, push notification | | |
| TC-UAT-15 | UAT | Report emergency | Emergency occurs | Tap emergency button | Emergency data | Emergency reported, notification | | |
| TC-UAT-16 | UAT | View ride history | Ride completed | View history | None | Past rides listed | | |

#### Creating and Joining Rides

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-17 | UAT | Create ride with valid data | User logged in | Complete ride form | All fields valid | Ride created successfully | | |
| TC-UAT-18 | UAT | Join ride via invitation | Invitation received | Accept invitation | Token | Ride joined, lobby displayed | | |
| TC-UAT-19 | UAT | Leave ride | Participant in ride | Tap leave ride | None | Left ride, status updated | | |

#### Tracking Participants

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-20 | UAT | View all participants on map | Ride active | Open live map | None | All participant markers visible | | |
| TC-UAT-21 | UAT | View separation status | Ride active | Check participant status | None | Status indicators shown | | |
| TC-UAT-22 | UAT | Handle stale location | Participant offline | Wait >2 min without update | None | Stale indicator shown | | |

#### Receiving Alerts

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-23 | UAT | Receive separation warning | Rider >500m for 60s | Wait for alert | Separation detected | Warning banner shown | | |
| TC-UAT-24 | UAT | Receive high separation alert | Rider >1000m for 30s | Wait for alert | Separation detected | Critical alert shown | | |
| TC-UAT-25 | UAT | Receive regrouped notification | Rider returns within 300m | Wait for regroup | Rider back in range | Regrouped notification shown | | |

#### Finding Meeting Points

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-26 | UAT | View meeting point recommendations | Separation active | Request meeting point | None | Candidates with ETAs displayed | | |
| TC-UAT-27 | UAT | Approve meeting point | Recommendation displayed | Tap approve | Candidate selected | Point shared with all | | |
| TC-UAT-28 | UAT | Navigate to meeting point | Point approved | Tap navigate | None | Navigation started | | |

#### Reporting Emergencies

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-29 | UAT | Report emergency | Emergency occurs | Fill emergency form | Type, severity, description | Emergency reported | | |
| TC-UAT-30 | UAT | Acknowledge emergency | Emergency active | Tap acknowledge | None | Acknowledgment recorded | | |
| TC-UAT-31 | UAT | Resolve emergency | Emergency acknowledged | Tap resolve | None | Emergency resolved | | |

#### Completing Rides

| Test Case ID | Module | Test Scenario | Preconditions | Test Steps | Test Input | Expected Result | Actual Result | Pass/Fail |
|--------------|--------|---------------|---------------|------------|------------|-----------------|---------------|-----------|
| TC-UAT-32 | UAT | Complete ride | Ride active | Leader ends ride | None | Ride marked completed | | |
| TC-UAT-33 | UAT | View ride summary | Ride completed | Open ride details | None | Summary with duration, events | | |
| TC-UAT-34 | UAT | View ride history | Multiple completed rides | Open history screen | None | List of past rides | | |

### 15.2 UAT Exit Criteria
- All critical user scenarios pass
- No showstopper bugs
- Performance acceptable for target devices
- User feedback incorporated
- Product owner sign-off

---