# Section 14: Security and Privacy

## 14.1 Authentication

- Email/password registration and login
- JWT access token (15 min expiry) + refresh token (7 days)
- Refresh token rotation on use
- Access token in `Authorization: Bearer` header for REST
- JWT as query param or subprotocol for WebSocket

## 14.2 JWT Handling

- Access tokens: short-lived, stored in memory or secure storage
- Refresh tokens: stored in secure storage (Keychain/EncryptedSharedPreferences)
- Token refresh interceptor: automatically refresh on 401
- Logout: blacklist refresh token, clear local storage

## 14.3 Password Hashing

- bcrypt or Argon2id with cost factor ≥ 12
- Passwords never logged or returned in API responses
- Minimum 8 characters enforced

## 14.4 Authorization

- Ride membership check on all ride-scoped endpoints
- Leader-only actions: ride config, participant management, start/end, meeting point approval
- Self-only actions: profile, own location, own device tokens
- WebSocket: join ride group only if valid JWT + active membership

## 14.5 Ride Membership Verification

- Middleware checks `ride_participants` table for `user_id` + `ride_id` + `is_active = true`
- Applied to: locations, route, alerts, events, emergencies, meeting points

## 14.6 Invitation Security

- UUID v4 tokens (unguessable)
- Single-use: consumed on accept/reject
- 24-hour expiry
- Only ride leader can send
- Invitee can only respond to own invitations

## 14.7 WebSocket Authentication

- JWT validated on connect via custom middleware
- Ride membership verified before joining channel group
- Invalid token → close code 4003
- Message `ride_id` must match connected group

## 14.8 Location Privacy

- Location shared only within ride group members
- No public location exposure
- Last known location retained with staleness indicator
- Location data deleted per retention policy (90 days for completed rides)

## 14.9 Secure Communication

- HTTPS for all REST API calls
- WSS (secure WebSocket) for real-time
- Certificate pinning optional for production

## 14.10 Device Token Protection

- Tokens tied to authenticated user
- Unique constraint on token
- Invalid/revoked tokens removed on push failure
- Tokens never exposed in API responses

## 14.11 Data Retention

- Active ride locations: latest per user only
- Completed ride locations: 90 days, then purged
- Alerts, events, emergencies: 90 days
- Ride metadata: indefinite (history)

## 14.12 Input Validation

- Email format, phone E.164
- Coordinate range checks (-90 to 90, -180 to 180)
- Enum validation for status, type, role fields
- Required field enforcement

## 14.13 Rate Limiting

- Login: max 5 attempts per 15 minutes per IP
- Registration: max 10 per hour per IP
- Invitation: max 20 per hour per ride
- Location updates: max 1 per 3 seconds per connection

## 14.14 Abuse Prevention

- Duplicate invitation prevention (same ride + invitee + pending)
- Fake location rejection: accuracy > 50m or speed > 150 m/s rejected
- Emergency report rate limit: max 5 per hour per user
