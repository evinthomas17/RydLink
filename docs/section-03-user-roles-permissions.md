# Section 3: User Roles and Permissions

## 3.1 Role Definitions

| Role | Description | Account Type? |
|------|-------------|---------------|
| Registered User | Base account. Can register, login, manage profile, create rides, send/receive invitations. | Yes (standalone account) |
| Ride Leader | The user who created a specific ride. Ride-specific role, not a separate account type. | No (derived per-ride) |
| Ride Participant | A user who joined a ride via invitation. Ride-specific role. | No (derived per-ride) |

**Key rule:** A Registered User can be Ride Leader in Ride A and Ride Participant in Ride B simultaneously. Roles are scoped per ride.

## 3.2 Role-Specific Functionalities

### Registered User (Global)

- Register, login, logout
- Manage profile (name, phone, profile picture, emergency contact)
- Change password, password reset
- Create group rides (becomes Ride Leader of that ride)
- Search registered users by username/email
- Send and receive ride invitations
- Join rides via invitation code/link
- Share live GPS location during active rides
- View other participants on map
- Share ride events (fuel, food, rest, breakdown)
- Report emergencies
- Receive notifications
- View completed ride history

### Ride Leader (Per Ride)

- Create and configure rides (name, start, destination, departure time)
- Generate invitation code/link
- Search and invite registered users to the ride
- Manage participants (approve join requests, remove participants)
- View live locations of all participants
- Start the ride
- End/complete the ride
- Monitor group alerts (separation, deviation, stationary)
- Review and approve/reject meeting point recommendations
- Coordinate ride events and emergencies

### Ride Participant (Per Ride)

- Receive and respond to ride invitations (accept/reject)
- Join group rides
- Share live location during journey
- View other participants' locations on map
- Receive alerts and regrouping recommendations
- Share ride events
- Report breakdowns and emergencies
- Follow approved meeting point recommendations
- View ride updates and completed ride information

## 3.3 Permission Matrix

| Action | Registered User | Ride Leader | Ride Participant | Auth Required | Ride Membership |
|--------|:---:|:---:|:---:|:---:|:---:|
| Register / Login | ✓ | – | – | No | No |
| Manage own profile | ✓ | – | – | Yes | No |
| Create a ride | ✓ | (becomes leader) | – | Yes | No |
| View ride details | ✓ | ✓ | ✓ | Yes | Yes (member) |
| Invite users to ride | – | ✓ | – | Yes | Yes (leader) |
| Accept/reject invitation | ✓ | ✓ | ✓ | Yes | No (until joined) |
| Join a ride | ✓ | ✓ | ✓ | Yes | Yes (via invite) |
| View participant list | ✓ | ✓ | ✓ | Yes | Yes |
| Approve/remove participants | – | ✓ | – | Yes | Yes (leader) |
| Share live location | ✓ | ✓ | ✓ | Yes | Yes (active ride) |
| View others' locations | ✓ | ✓ | ✓ | Yes | Yes |
| Start a ride | – | ✓ | – | Yes | Yes (leader) |
| End/complete a ride | – | ✓ | – | Yes | Yes (leader) |
| Receive group alerts | ✓ | ✓ | ✓ | Yes | Yes |
| Approve meeting point | – | ✓ | – | Yes | Yes (leader) |
| Share ride event | ✓ | ✓ | ✓ | Yes | Yes |
| Report emergency | ✓ | ✓ | ✓ | Yes | Yes |
| View ride history | ✓ | ✓ | ✓ | Yes | Yes (if participated) |

## 3.4 Ride-Specific Role Handling

- Roles are stored in the `ride_participants` table, not in the `users` table.
- A user record has no global `role` column.
- Ride membership is established via an accepted `ride_invitation`.
- The ride creator is inserted into `ride_participants` with `role = 'leader'` at ride creation.
- All other accepted invitees have `role = 'participant'`.
- On ride completion, participant records are preserved for history but `is_active` is set to false.

## 3.5 Authorization Rules

1. **Authentication:** All API endpoints except registration/login require a valid JWT access token in `Authorization: Bearer <token>` header.
2. **Ride Membership:** Any action targeting a specific ride (location sharing, viewing participants, events) requires the user to be an active member of that ride (record in `ride_participants` with `is_active = true`).
3. **Leader-Only Actions:** Ride configuration, participant management, ride start/end, meeting point approval require `role = 'leader'` for that ride.
4. **Self-Only Actions:** Profile updates, password changes, own location sharing are restricted to the authenticated user's own record.
5. **WebSocket Authorization:** A client may only join a ride's WebSocket group if it presents a valid JWT and is a member of that ride.
6. **Invitation Security:** Invitations can only be sent by the ride leader. Invitation tokens are single-use and expire after 24 hours.
