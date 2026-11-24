# Feature Classification – Already Included and Not Yet Included Features

This document classifies RydLink features into **Already Included**, **Partially Included**, and **Not Yet Included**, with a technology classification and recommended implementation decision.

---

## 1. Already Included Features

### 1.1 Intelligent Rider Separation Detection

**Status: Already Included**

**Objective:** Identify when a rider becomes excessively separated from the group.

**Included technologies:**
- GPS Location Services.
- PostgreSQL and PostGIS.
- Python.
- Django Channels.
- WebSockets.
- Geographical Distance Calculation.

**Existing functionality:**
- Monitor participant locations.
- Calculate the distance between riders and the group reference position.
- Detect separation beyond configured thresholds.
- Generate separation warnings and high-separation alerts.
- Detect when participants regroup.

---

### 1.2 Intelligent Route Deviation Detection

**Status: Already Included**

**Objective:** Identify when a rider moves away from the planned group route.

**Included technologies:**
- GPS Tracking.
- PostGIS.
- OSRM.
- Python.
- WebSockets.
- MapLibre.

**Existing functionality:**
- Store the planned group route.
- Track actual participant locations.
- Compare participant locations with the planned route.
- Identify significant route deviations.
- Notify relevant participants.

---

### 1.3 Unusual Stopping Detection

**Status: Already Included**

**Objective:** Identify riders who remain stationary unusually long during a journey.

**Included technologies:**
- GPS Location Services.
- Python.
- Speed and Distance Calculations.
- Rule-Based Monitoring.
- Django Channels.
- WebSockets.

**Existing functionality:**
- Monitor participant movement.
- Identify stationary participants.
- Detect unusual stopping conditions.
- Generate notifications when configured conditions are satisfied.

---

### 1.4 Intelligent Regrouping and Meeting-Point Recommendation

**Status: Already Included**

**Objective:** Recommend suitable meeting locations when group members become separated.

**Included technologies:**
- PostgreSQL and PostGIS.
- Python.
- OpenStreetMap.
- Overpass API.
- OSRM.
- Geospatial Algorithms.

**Existing functionality:**
- Identify separated participants.
- Search for nearby fuel stations, restaurants and parking locations.
- Calculate travel times to candidate meeting points.
- Rank suitable meeting locations.
- Recommend meeting points.
- Allow the Ride Leader to approve recommendations.

---

### 1.5 Live Location Tracking

**Status: Already Included**

**Included technologies:**
- React Native.
- GPS Location Services.
- Expo Location.
- Django REST Framework.
- Django Channels.
- WebSockets.
- PostgreSQL and PostGIS.
- MapLibre.

**Existing functionality:**
- Collect participant GPS locations.
- Send location updates to the backend.
- Broadcast location information to ride participants.
- Display participant locations on a shared map.

---

### 1.6 Route Management and Visualization

**Status: Already Included**

**Included technologies:**
- MapLibre.
- OpenStreetMap.
- OSRM.
- PostGIS.
- React Native.

**Existing functionality:**
- Starting and destination selection.
- Route calculation.
- Route visualization.
- Distance and estimated travel time display.
- Route storage and retrieval.

---

### 1.7 Emergency and Breakdown Management

**Status: Already Included**

**Included technologies:**
- React Native.
- Django REST Framework.
- GPS Location Services.
- WebSockets.
- Push Notifications.
- MapLibre.

**Existing functionality:**
- Report emergencies and vehicle breakdowns.
- Share emergency location.
- Notify ride participants.
- Acknowledge emergency reports.
- Track emergency response status.

---

### 1.8 Ride Events and Stop Management

**Status: Already Included**

**Existing functionality:**
- Create fuel stops.
- Create food stops.
- Create rest stops.
- Share event locations.
- Notify participants about ride events.

---

### 1.9 Offline and Network Recovery

**Status: Already Included at the Functional Level**

**Included technologies:**
- React Native.
- Django REST Framework.
- WebSockets.
- Background Location Services.
- PostgreSQL.

**Existing functionality:**
- Detect network disconnection.
- Handle WebSocket disconnection.
- Reconnect when connectivity returns.
- Identify stale participant locations.
- Synchronize location information after connectivity is restored.

**Note:** Local SQLite-based offline storage and a complete offline-first synchronization mechanism are not yet finalized.

---

## 2. Partially Included Features

These features are related to existing RydLink modules, but their advanced implementations are not yet part of the finalized specification.

### 2.1 Intelligent Movement and Behaviour Monitoring

**Status: Partially Included**

**Already included:**
- GPS movement monitoring.
- Speed and direction monitoring.
- Unusual stopping detection.
- Route deviation monitoring.

**Not yet included:**
- Advanced movement pattern analysis.
- Isolation Forest anomaly detection.
- Machine learning-based behavioural classification.
- Historical movement pattern analysis.

---

### 2.2 Predictive ETA and Group Arrival Estimation

**Status: Partially Included**

**Already included:**
- Route distance calculation.
- Estimated travel time using OSRM.
- Participant location tracking.

**Not yet included:**
- Individual rider arrival prediction using historical journey data.
- Group-wide arrival time estimation.
- Dynamic ETA prediction based on movement patterns.
- Historical travel-time analysis.

---

### 2.3 GPS Distance Calculation Algorithms

**Status: Partially Included**

**Already included:**
- Geographical distance calculation.
- PostGIS spatial processing.

**Not yet finalized:**
- Explicit selection and documentation of the Haversine algorithm.
- Comparison of Haversine and PostGIS distance calculations.

---

### 2.4 Background GPS Tracking

**Status: Partially Included**

**Already included:**
- React Native GPS integration.
- Foreground and background location tracking requirements.
- Location recovery after interruptions.

**Not yet finalized:**
- Adaptive GPS update frequency.
- Detailed battery optimization strategy.
- Dynamic tracking intervals based on movement and network conditions.

---

### 2.5 Push Notification Integration

**Status: Partially Included**

**Already included:**
- In-app notifications.
- Real-time WebSocket notifications.
- Push notification requirements.

**Not yet finalized:**
- Complete Firebase Cloud Messaging implementation.
- Device token management.
- Push notification delivery and failure handling.

---

## 3. Not Yet Included Features

The following are proposed additional features that are not part of the current finalized RydLink scope.

### 3.1 Adaptive GPS Tracking

**Objective:** Dynamically adjust GPS update frequency based on rider movement and network conditions.

**Proposed technologies:**
- React Native Background Location APIs.
- GPS.
- Adaptive Sampling Algorithm.
- Speed and Distance Calculations.

**Proposed update intervals:**
- Stationary: Every 30 seconds.
- Normal movement: Every 5 seconds.
- High-speed movement: Every 2–3 seconds.
- Poor network: Store and synchronize pending updates.

---

### 3.2 Offline-First Location Management Using SQLite

**Objective:** Store location records locally during internet interruptions and synchronize them when connectivity returns.

**Proposed technologies:**
- SQLite.
- React Native NetInfo.
- Background Synchronization.
- Django REST Framework.
- PostgreSQL.

---

### 3.3 Machine Learning-Based Movement Anomaly Detection

**Objective:** Analyse rider movement patterns using machine learning techniques.

**Proposed technologies:**
- Python.
- NumPy.
- Pandas.
- Scikit-learn.
- Isolation Forest.

**Potential functionality:**
- Detect unusual speed changes.
- Identify abnormal movement patterns.
- Analyse repeated route deviations.
- Identify unusual stopping patterns using historical data.

---

### 3.4 K-Means-Based Regrouping Optimization

**Objective:** Use clustering algorithms to analyse participant locations and identify suitable meeting areas.

**Proposed technologies:**
- Python.
- Scikit-learn.
- K-Means Clustering.
- PostGIS.
- Geospatial Algorithms.

**Note:** The existing RydLink regrouping feature already uses geographical processing and route-based ranking. K-Means is an optional advanced extension, not a requirement for the current regrouping implementation.

---

### 3.5 Advanced Predictive ETA

**Objective:** Predict participant and group arrival times using historical movement information.

**Proposed technologies:**
- Python.
- Historical Journey Data.
- NumPy.
- Pandas.
- OSRM.
- Machine Learning (optional).

---

### 3.6 GPS-Based Ride Simulation Module

**Objective:** Simulate multiple riders and demonstrate RydLink's intelligent monitoring features without requiring actual group travel.

**Proposed technologies:**
- Python.
- NumPy.
- GeoPandas.
- PostgreSQL.
- PostGIS.
- Django REST Framework.
- WebSockets.
- React Native.

**Simulation scenarios:**
- Normal group movement.
- Excessive rider separation.
- Route deviation.
- Unexpected stopping.
- Network disconnection.
- Emergency situations.

---

### 3.7 Advanced Route Optimization

**Objective:** Optimize meeting-point and route selection using advanced optimization algorithms.

**Proposed technologies:**
- Google OR-Tools.
- OSRM or GraphHopper.
- Python.
- PostGIS.

---

### 3.8 On-Device Machine Learning

**Objective:** Run lightweight movement analysis directly on the mobile device.

**Proposed technologies:**
- TensorFlow Lite.
- React Native.
- GPS Location Services.

This is an optional future enhancement.

---

## 4. Technology Classification

| Technology | Current Status |
|---|---|
| React Native | Already Included |
| JavaScript | Already Included in current frontend specification |
| TypeScript | Not finalized; optional migration |
| Django REST Framework | Already Included |
| PostgreSQL | Already Included |
| PostGIS | Already Included |
| Django Channels | Already Included |
| WebSockets | Already Included |
| Redis | Already Included |
| JWT Authentication | Already Included |
| MapLibre | Already Included |
| OpenStreetMap | Already Included |
| OSRM | Already Included |
| Overpass API | Already Included |
| Expo Location | Already Included |
| Expo Task Manager | Already Included in the proposed frontend stack |
| SQLite | Not Yet Included |
| React Native NetInfo | Not Yet Included |
| NumPy | Not Yet Included in the finalized stack |
| Pandas | Not Yet Included in the finalized stack |
| Scikit-learn | Not Yet Included |
| Isolation Forest | Not Yet Included |
| K-Means Clustering | Not Yet Included |
| GraphHopper | Not Included; alternative to OSRM |
| Firebase Cloud Messaging | Partially Included |
| Celery | Not Yet Included |
| Google OR-Tools | Not Yet Included |
| TensorFlow Lite | Not Yet Included |
| GeoPandas | Not Yet Included |
| Postman | Already Included |
| Git and GitHub | Already Included |

---

## 5. Recommended Final Feature Classification

### Core Features to Implement

1. User Authentication and Profile Management.
2. Ride Creation and Group Management.
3. In-App Ride Invitation Management.
4. Ride Lobby and Participant Management.
5. Live GPS Location Tracking.
6. Route Management and Visualization.
7. Intelligent Rider Separation Detection.
8. Intelligent Route Deviation Detection.
9. Unusual Stopping Detection.
10. Intelligent Regrouping and Meeting-Point Recommendation.
11. Ride Events and Stop Management.
12. Emergency and Breakdown Management.
13. Real-Time Communication.
14. Push Notification Management.
15. Network Connectivity and Location Recovery.
16. Ride Completion and History Management.

### Advanced Features to Consider

1. Adaptive GPS Tracking.
2. SQLite-Based Offline-First Location Management.
3. Machine Learning-Based Movement Anomaly Detection.
4. Predictive ETA and Group Arrival Estimation.
5. GPS-Based Ride Simulation.
6. Advanced Regrouping Optimization.

---

## 6. Recommended Implementation Decision

For the initial RydLink implementation, prioritize the existing core features. They are sufficient to demonstrate a complete group ride coordination and safety system.

After the core application works, consider implementing:

1. GPS-Based Ride Simulation – To demonstrate multiple riders and safety scenarios without actual travel.
2. Adaptive GPS Tracking – To improve location update efficiency and reduce battery consumption.
3. Machine Learning-Based Movement Anomaly Detection – To add an advanced intelligent monitoring capability.
4. Predictive ETA – To improve group journey coordination.

Do not add every proposed technology just to increase the technology count. Include a technology only when it supports a clearly defined project requirement.

**Final distinction:** RydLink already includes intelligent separation detection, route deviation detection, unusual stopping detection, rule-based monitoring and smart regrouping. Machine learning, adaptive GPS, SQLite offline-first management and GPS simulation are additional enhancements rather than existing finalized features.

---

*Document: feature-classification.md*
*Project: RydLink – Intelligent Group Ride Coordination and Safety System*
