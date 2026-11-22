# Section 13: UI/UX Design Specification

## 13.1 Navigation Structure

```
Tab Navigator (bottom):
├── Map Tab (active ride screen)
├── Rides Tab (ride list, create ride, history)
├── Invitations Tab (sent/received)
└── Profile Tab (settings, profile, logout)

Stack Navigators (modals/screens):
├── Auth Stack: Login, Register, Forgot Password
├── Ride Stack: Create Ride, Ride Details, Ride Lobby
├── Map Stack: Map View, Route View, Event Detail
├── Alert Stack: Alert List, Meeting Point Review
└── Profile Stack: Profile Edit, Emergency Contact
```

## 13.2 Screen List by Module

| Screen | Module | Purpose | Key Components | User Actions |
|--------|--------|---------|----------------|--------------|
| Login | 1 | Authenticate | Email/password inputs, login button | Login, navigate to register |
| Register | 1 | Create account | Inputs, confirm password | Register |
| Profile | 1 | Manage profile | Name, phone, emergency contact, picture | Edit, save |
| Create Ride | 2 | New ride | Name, start/dest pickers, departure time | Create, cancel |
| Ride Details | 2 | Ride info | Ride info, participants, leader actions | Start, end, invite |
| User Search | 3 | Find users | Search bar, user list | Search, select to invite |
| Invitations | 3 | Invite list | Pending invitations, accept/reject | Accept, reject |
| Ride Lobby | 4 | Pre-ride | Participant list, status, start button | Approve, remove, start |
| Map View | 5 | Live tracking | MapLibre map, markers, **separation status indicators**, location button | View participants, view separation alerts |
| Route View | 6 | Route display | Polyline, distance, ETA, external nav button | Open external nav |
| Alert List | 7 | Alerts | Alert list, severity badges, **separation filter**, resolve | Resolve |
| Meeting Point Review | 8 | Regrouping | Candidate list, map markers, approve/reject | Approve, reject |
| Create Event | 9 | Ride event | Type picker, location, note, time | Create |
| Event List | 9 | Events | Event list, map markers | View, complete |
| Emergency Report | 10 | Emergency | Type, severity, description, location | Report |
| Emergency Alert | 10 | Emergency display | Emergency details, status, coordinate | Acknowledge |
| Notification Settings | 11 | Push prefs | Toggle permissions | Enable/disable |
| Ride History | 13 | Past rides | Ride list, detail | View detail |
| **Separation Status Screen** (new) | **7** | **Separation overview** | List of riders with separation status, distances, timer info | View details, acknowledge |

## 13.3 Map Interface Layout

```
+--------------------------------------------------+
| Ride Header: Ride Name | Status | ETA           |
+--------------------------------------------------+
|                                                  |
|                                                  |
|         [MapLibre Map - Full Screen]             |
|                                                  |
|   [Participant Markers]  [Route Polyline]        |
|   **[Separation Status Indicators]**             |
|   [Event Markers]        [Meeting Point Marker]  |
|                                                  |
+--------------------------------------------------+
| [Center on Me] [Layers] **[Alerts]** [Events]    |
+--------------------------------------------------+
| Bottom Sheet: Participant List | Location Status |
+--------------------------------------------------+
```

## 13.4 States

- **Loading:** Skeleton screens, spinner on map/data fetch
- **Empty:** "No rides yet", "No participants", "No alerts" with action button
- **Error:** Retry button, error message, offline indicator
- **Permission:** Location permission rationale dialog, settings redirect

## 13.5 Navigation Flow

Auth → (if new ride) Create Ride → Invite → Lobby → Start → Map → (alerts/events) → End → History
