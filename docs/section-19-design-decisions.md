# Section 19: System-Wide Design Decisions

## 19.1 Consolidated Technical Rules

| ID | Decision | Value | Rationale |
|----|----------|-------|-----------|
| TD-01 | Frontend language | JavaScript (ES6+), not TypeScript | Per requirements |
| TD-02 | Backend framework | Django 4.2 LTS | Per requirements; no FastAPI |
| TD-03 | Database | PostgreSQL + PostGIS | Per requirements |
| TD-04 | Auth | Simple JWT (access 15 min, refresh 7 days) | Stateless, scalable |
| TD-05 | Real-time | Django Channels + Daphne + Redis | Per requirements |
| TD-06 | Map | MapLibre + OpenStreetMap tiles | Per requirements |
| TD-07 | Routing | OSRM (self-hosted for prod) | Per requirements |
| TD-08 | POI search | Overpass API (self-hosted for prod) | Per requirements |
| TD-09 | Notifications | Expo Notifications + FCM | Per requirements |
| TD-10 | Invitations | In-app only; no WhatsApp mandatory | Per requirements |
| TD-11 | Algorithms | Rule-based Python; no ML | Per requirements |
| TD-12 | Primary keys | UUID | Distributed-safe, no sequential IDs |
| TD-13 | Location storage | geography(Point,4326) PostGIS | Spatial queries in meters |
| TD-14 | Location retention | Latest only for active; 90 days for completed | Privacy + DB growth |
| TD-15 | Roles | Per-ride (leader/participant), not global | Flexibility |
| TD-16 | Platform | Android first, iOS later | Background tracking simplicity |
| TD-17 | Python version | 3.11 or 3.12 LTS | Django compatibility |
| TD-18 | React Native | Expo development build (custom dev client) | Background tracking |
| TD-19 | Rate limiting | Login 5/15min, invites 20/hr/ride | Abuse prevention |
| TD-20 | Emergency | Immediate WebSocket + high-priority push | Safety critical |

## 19.2 Consolidated Business Rules

| ID | Rule |
|----|------|
| BR-01 | Ride creator is automatically Ride Leader |
| BR-02 | Only Ride Leader can start/end a ride |
| BR-03 | Only Ride Leader can approve meeting points |
| BR-04 | Invitations are single-use, 24-hour expiry |
| BR-05 | Only Ride Leader can send invitations |
| BR-06 | Location shared only with ride members |
| BR-07 | Emergency alerts bypass normal alert throttling |
| BR-08 | Ride Leader can transfer leadership to participant |
| BR-09 | Completed rides are archived, not deleted |
| BR-10 | Stale location (>2 min) shown as gray marker |

## 19.3 Unresolved Design Questions

| Question | Status | Needed For |
|----------|--------|------------|
| Production tile provider (MapTiler vs Mapbox vs self-hosted) | Undecided | Section 6.6 |
| FCM project setup for production push | Undecided | Module 11 production |
| Target OSRM region (.pbf file selection) | Undecided | OSRM self-hosting |
| iOS support timeline | Undecided | Platform expansion |
| Separation threshold finalization for production use | **Threshold values documented and configured; to be validated with real-world testing** | Module 7 production tuning |
