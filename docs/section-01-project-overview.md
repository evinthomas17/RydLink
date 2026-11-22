# Section 1: Project Overview

## 1.1 Project Title

**RydLink – Intelligent Group Ride Coordination and Safety System**

## 1.2 Abstract

RydLink is a cross-platform mobile application that enables groups of people travelling together in separate vehicles to coordinate their journey, share real-time GPS locations, monitor each other on a common interactive map, and receive intelligent safety alerts. The system detects group separation, route deviations, and unusual stationary activity, and recommends meeting points for regrouping. It supports ride events (fuel stops, food breaks, rest stops), emergency and breakdown reporting, push notifications, and ride history. RydLink complements—but does not replace—existing navigation apps such as Google Maps.

## 1.3 Introduction

Group travel in multiple vehicles faces common problems: vehicles get separated at traffic signals, participants take wrong turns, stops are uncoordinated, and emergencies go unnoticed until too late. Existing navigation tools are designed for individual drivers, not groups. RydLink fills this gap by providing a shared group context: live location sharing, intelligent monitoring, regrouping recommendations, and safety alerts—all within a single application.

## 1.4 Problem Statement

- No centralized view of all participants in a multi-vehicle group.
- No automatic detection of group separation or route deviation.
- No intelligent meeting point recommendation when separated.
- No standardized way to share ride events (fuel, food, breakdown, emergency).
- No ride history or post-ride summary for completed journeys.

## 1.5 Objectives

| ID | Objective |
|----|-----------|
| OBJ-01 | Enable users to create and join group rides. |
| OBJ-02 | Provide in-app invitations for registered users. |
| OBJ-03 | Enable real-time GPS location sharing. |
| OBJ-04 | Display all ride participants on a common interactive map. |
| OBJ-05 | Provide route visualization, distance, and estimated travel time. |
| OBJ-06 | Detect potential group separation. |
| OBJ-07 | Detect route deviations. |
| OBJ-08 | Detect unusual stationary activity. |
| OBJ-09 | Recommend suitable meeting points. |
| OBJ-10 | Support ride events and planned stops. |
| OBJ-11 | Provide emergency and breakdown reporting. |
| OBJ-12 | Send important notifications. |
| OBJ-13 | Maintain completed ride history. |

## 1.6 Project Scope

### In Scope

- User registration, login, profile management (Module 1).
- Ride creation, configuration, group management (Module 2).
- In-app ride invitation lifecycle (Module 3).
- Ride lobby and participant management (Module 4).
- Live GPS tracking and map visualization (Module 5).
- Route management and navigation via OSRM (Module 6).
- Intelligent ride monitoring and alert management (Module 7).
- Intelligent regrouping and meeting point recommendation (Module 8).
- Ride event and stop management (Module 9).
- Emergency and breakdown management (Module 10).
- Push notification management (Module 11).
- Network connectivity and location recovery (Module 12).
- Ride completion and history management (Module 13).

### Out of Scope

- Turn-by-turn navigation within RydLink (external app integration only).
- WhatsApp/SMS integration (not mandatory).
- Machine learning / AI features (rule-based only).
- Payment processing, vehicle telemetry (OBD-II), or hardware integration.

## 1.7 Existing System

Participants use separate navigation apps (Google Maps, Waze) and messaging apps (WhatsApp, SMS) to coordinate. Location sharing is manual and fragmented. There is no automated detection of separation, deviation, or stationary incidents. Emergency reporting relies on phone calls.

## 1.8 Proposed System

A unified mobile application with:

- **Backend:** Python, Django, Django REST Framework, Django Channels, Daphne, Redis, PostgreSQL + PostGIS.
- **Frontend:** React Native (JavaScript), Expo, MapLibre React Native.
- **Real-time:** WebSocket via Django Channels + Redis channel layer.
- **Maps/Routing:** MapLibre, OpenStreetMap, OSRM, Overpass API, Nominatim (optional).
- **Notifications:** Expo Notifications, Firebase Cloud Messaging (FCM).

## 1.9 Advantages

- Single shared map view for all ride participants.
- Automated safety alerts (separation, deviation, stationary).
- Intelligent meeting point recommendations based on travel time and distance.
- Ride event sharing and emergency reporting.
- Ride history for completed journeys.
- Role-based access control (Ride Leader vs. Participant).

## 1.10 Limitations

- Not a replacement for dedicated navigation apps.
- Background tracking requires Expo development build (not Expo Go).
- Public OSRM/Overpass/Nominatim instances have rate limits; production requires self-hosting.
- GPS accuracy depends on device hardware and environmental conditions.
- Push notifications require EAS build + FCM for production.
- Python 3.11/3.12 targeted (Django compatibility); Python 3.15 not guaranteed.
