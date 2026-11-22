# RydLink – Intelligent Group Ride Coordination and Safety System

## Master Project Documentation

This repository contains the complete technical documentation for **RydLink**, a mobile application designed for people travelling together in separate vehicles. The documentation serves as the primary reference throughout the entire development process — frontend, backend, database, APIs, real-time communication, and intelligent functionalities.

## Documentation Structure

| Document | Description |
|----------|-------------|
| `docs/section-01-project-overview.md` | Section 1: Project Overview |
| `docs/section-02-requirement-analysis.md` | Section 2: Requirement Analysis |
| `docs/section-03-user-roles-permissions.md` | Section 3: User Roles and Permissions |
| `docs/section-04-system-modules.md` | Section 4: Detailed System Modules (1–13) |
| `docs/section-05-system-workflow.md` | Section 5: Complete System Workflow |
| `docs/section-06-system-architecture.md` | Section 6: System Architecture |
| `docs/section-07-technology-stack.md` | Section 7: Complete Technology Stack and Dependencies |
| `docs/section-08-database-design.md` | Section 8: Complete Database Design |
| `docs/section-09-rest-api-documentation.md` | Section 9: Complete REST API Documentation |
| `docs/section-10-websocket-realtime.md` | Section 10: WebSocket and Real-Time Communication |
| `docs/section-11-gps-map-routing.md` | Section 11: GPS, Map and Routing Design |
| `docs/section-12-intelligent-algorithms.md` | Section 12: Intelligent Monitoring and Recommendation Algorithms |
| `docs/section-13-ui-ux-design.md` | Section 13: UI/UX Design Specification |
| `docs/section-14-security-privacy.md` | Section 14: Security and Privacy |
| `docs/section-15-error-handling.md` | Section 15: Error Handling and Special Scenarios |
| `docs/section-16-testing-strategy.md` | Section 16: Testing Strategy |
| `docs/section-17-deployment-plan.md` | Section 17: Development and Deployment Plan |
| `docs/section-18-future-enhancements.md` | Section 18: Future Enhancements and Limitations |
| `docs/section-19-design-decisions.md` | Section 19: System-Wide Design Decisions |
| `docs/section-20-readiness-checklist.md` | Section 20: Development Readiness Checklist |
| `docs/database-design-document.md` | Separate Detailed Database Design Document (DDD) |

## Recommended Implementation Sequence

1. Project and environment setup
2. Authentication and profile management
3. Ride creation and group management
4. In-app invitation management
5. Ride lobby and participant management
6. Live GPS tracking
7. Map visualization
8. Route management
9. Intelligent monitoring
10. Meeting point recommendations
11. Ride event management
12. Emergency and breakdown management
13. Push notifications
14. Network connectivity and recovery
15. Ride completion and history
16. Testing and deployment

## Technology Stack Summary

- **Frontend:** React Native (JavaScript), Expo SDK, MapLibre React Native, React Navigation, Axios
- **Backend:** Python 3.11/3.12, Django 4.2 LTS, Django REST Framework, Django Channels, Daphne, Simple JWT
- **Database:** PostgreSQL 14/15 + PostGIS 3.x
- **Real-time:** Redis (channel layer), WebSocket
- **Maps/Routing:** OpenStreetMap, OSRM, Overpass API, Nominatim (optional)
- **Notifications:** Expo Notifications, Firebase Cloud Messaging (FCM)

## Confirmed Design Decisions

| Decision | Value |
|----------|-------|
| Python version | 3.11 or 3.12 (LTS) |
| React Native | Expo development build (custom dev client) |
| Local DB/cache | Native install (PostgreSQL + PostGIS + Redis) |
| Map tiles (prod) | Self-hosted or paid provider (MapTiler/Mapbox) |
| OSRM | Self-hosted for prod; demo server OK for prototype |
| Target platforms | Android first, iOS later |
| Expo/FCM | Prototype-only push; EAS + FCM for production |
| **Separation detection thresholds** | 500m primary / 1000m high; 60s/30s confirmation; ≤30m GPS accuracy |

## Before You Begin

Review **Section 20: Development Readiness Checklist** and verify all 23 items before starting Module 1 (Authentication) implementation.

The **Database Design Document** must be finalized before backend model implementation begins.
