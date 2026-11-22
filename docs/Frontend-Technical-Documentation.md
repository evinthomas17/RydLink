# RydLink - Frontend Technical Documentation

## Table of Contents
1. [React Native Application Structure](#1-react-native-application-structure)
2. [Navigation Setup](#2-navigation-setup)
3. [Authentication Screens](#3-authentication-screens)
4. [User Profile Screens](#4-user-profile-screens)
5. [Ride Creation Screens](#5-ride-creation-screens)
6. [User Search and Invitation Screens](#6-user-search-and-invitation-screens)
7. [Ride Lobby Screens](#7-ride-lobby-screens)
8. [Live Map Integration](#8-live-map-integration)
9. [GPS Location Integration](#9-gps-location-integration)
10. [Real-Time Participant Tracking](#10-real-time-participant-tracking)
11. [Route Visualization](#11-route-visualization)
12. [Separation Alert Interface](#12-separation-alert-interface)
13. [Regrouping Recommendation Interface](#13-regrouping-recommendation-interface)
14. [Ride Event and Stop Interface](#14-ride-event-and-stop-interface)
15. [Emergency Management Interface](#15-emergency-management-interface)
16. [Notification Interface](#16-notification-interface)
17. [Ride History Interface](#17-ride-history-interface)

---

## 1. React Native Application Structure

### 1.1 Module Name
React Native Application Structure

### 1.2 Module Objective
Define the complete React Native project initialization, folder architecture, and configuration for the RydLink mobile application, ensuring consistent development setup and code organization.

### 1.3 Module Purpose
Provide the foundational structure for all frontend modules, including project setup, directory organization, and configuration patterns that all subsequent screens and components will follow.

### 1.4 Functional Requirements
- Initialize React Native project with Expo SDK
- Establish folder architecture following best practices
- Configure React Navigation navigation solution
- Set up API service layer with Axios
- Configure WebSocket service for real-time communication
- Set up state management with React Context/Hooks
- Define utility functions and constants
- Configure environment-specific settings
- Implement error handling across the application
- Establish styling conventions using StyleSheet

### 1.5 Technologies and Packages Required
- React Native 0.73+
- Expo SDK 50+
- React Navigation 6.x
- Axios 1.x
- MapLibre React Native Latest
- Expo Location Latest
- Expo Task Manager Latest
- Expo Notifications Latest
- @react-native-async-storage/async-storage
- @maplibre/react-native-maplibre-gl
- @react-navigation/native
- @react-navigation/native-stack
- React Native Vector Icons
- Date-fns (date handling)
- AsyncStorage (local persistence)

### 1.6 Dependencies on Other Modules
- Authentication module requires navigation and auth state management
- Ride lobby module depends on auth, ride state, and WebSocket connection
- Live tracking module depends on GPS, WebSocket, and MapLibre integration
- Regrouping module depends on map, Overpass API, and OSRM routing
- Emergency module depends on push notifications and WebSocket

### 1.7 Folder and File Structure
```
src/
├── assets/
│   ├── icons/
│   └── images/
├── constants/
│   ├── api.js          # API base URL, WebSocket config
│   ├── routes.js       # Route names, screen constants
│   └── themes.js       # Color palette, typography
├── contexts/
│   ├── AuthContext.js  # JWT token, user session, login/logout
│   ├── RideContext.js  # Current ride, participants, status
│   └── NotificationContext.js # Push token, notification handling
├── hooks/
│   ├── useAuth.js      # Authentication state hooks
│   ├── useLocation.js  # GPS location hooks
│   ├── useWebSocket.js # WebSocket connection hooks
│   └── useRide.js      # Ride state hooks
├── navigation/
│   ├── RootNavigator.js # Root navigation (auth vs main)
│   ├── AuthNavigator.js # Login/Register navigation
│   ├── MainNavigator.js # Main app navigation
│   └── screens.js       # Screen imports, navigation config
├── services/
│   ├── api.js          # Axios instance, interceptors
│   ├── websocket.js    # WebSocket connection management
│   └── notifications.js # Expo Notifications setup
├── screens/
│   ├── auth/
│   │   ├── RegisterScreen.js
│   │   └── LoginScreen.js
│   ├── main/
│   │   ├── RideCreationScreen.js
│   │   ├── RideLobbyScreen.js
│   │   ├── LiveMapScreen.js
│   │   ├── UserSearchScreen.js
│   │   ├── ProfileScreen.js
│   │   ├── RideHistoryScreen.js
│   │   ├── EmergencyScreen.js
│   │   └── NotificationScreen.js
│   └── shared/
│       ├── LoadingScreen.js
│       ├── ErrorBoundary.js
│       └── EmptyState.js
├── components/
│   ├── common/
│   │   ├── Button.js
│   │   ├── Input.js
│   │   ├── LoadingSpinner.js
│   │   ├── ErrorToast.js
│   │   └── SuccessToast.js
│   ├── map/
│   │   ├── MapComponent.js
│   │   ├── ParticipantMarker.js
│   │   ├── RoutePolyline.js
│   │   └── MarkerCluster.js
│   ├── tracking/
│   │   ├── SeparationIndicator.js
│   │   ├── StatusBadge.js
│   │   └── RegroupingBanner.js
│   └── events/
│       ├── EventCard.js
│       └── EmergencyBanner.js
├── utils/
│   ├── format.js       # Number/date formatting
│   ├── validation.js   # Form validation helpers
│   └── constants.js    # Zone colors, thresholds
├── store/
│   ├── rootReducer.js
│   ├── rootSaga.js     # Redux Saga for async operations
│   └── middleware.js
├── App.js              # Root component with navigation
├── App.json            # Expo configuration
├── App.tsx             # TypeScript config (if used)
└── index.js            # Entry point
```

### 1.8 Database Models and Fields (Frontend Perspective)
*(Frontend interacts with backend API - models described from perspective of API response data)*

**User Response:**
```json
{
  "id": "uuid",
  "email": "alice@example.com",
  "first_name": "Alice",
  "last_name": "Smith",
  "phone": "+1234567890",
  "profile_picture_url": "https://...",
  "is_active": true,
  "email_verified": true
}
```

**Ride Response:**
```json
{
  "id": "uuid",
  "name": "Weekend Trip",
  "leader": "550e8400-e29b-41d4-a716-446655440000",
  "start_location": {"lat": 40.7128, "lng": -74.0060},
  "dest_location": {"lat": 40.7589, "lng": -73.9851},
  "departure_time": "2026-11-21T14:00:00Z",
  "status": "planned|active|completed|cancelled",
  "participant_count": 3,
  "max_participants": 50
}
```

**Participant Response:**
```json
{
  "user_id": "uuid",
  "display_name": "Alice",
  "role": "leader|participant",
  "status": "joined|left|active",
  "separation_status": "normal|monitoring|separation_warning|high_separation|regrouped",
  "separation_distance": 245.5
}
```

**Alert Response:**
```json
{
  "id": "uuid",
  "type": "separation_warning|high_separation|regrouped|deviation|stationary",
  "severity": "warning|high|info",
  "distance_m": 520,
  "message": "Separation Warning: 520m from group",
  "timestamp": "2026-11-21T10:30:00Z"
}
```

**Event Response:**
```json
{
  "id": "uuid",
  "type": "fuel|food|rest|breakdown|other",
  "display_name": "Alice",
  "message": "Fuel stop at gas station",
  "location": {"lat": 40.7200, "lng": -74.0100},
  "status": "planned|completed|cancelled",
  "created_at": "2026-11-21T10:30:00Z"
}
```

**Emergency Response:**
```json
{
  "id": "uuid",
  "type": "breakdown|emergency",
  "severity": "low|medium|high|critical",
  "description": "Rider experiencing mechanical failure",
  "location": {"lat": 40.7150, "lng": -74.0080},
  "status": "active|acknowledged|resolved|false_alarm",
  "created_at": "2026-11-21T10:30:00Z"
}
```

**Device Token Response:**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "token": "ExpoPushToken[xxxx-xxxx-xxxx]",
  "platform": "ios|android",
  "created_at": "2026-11-21T10:30:00Z"
}
```

**Notification Response:**
```json
{
  "id": "uuid",
  "user_id": "uuid",
  "ride_id": "uuid",
  "type": "invitation|separation|event|emergency",
  "title": "Separation Warning",
  "body": "Alice has separated from the group by 500m",
  "data": {"ride_id": "...", "user_id": "..."},
  "read": false,
  "created_at": "2026-11-21T10:30:00Z"
}
```

### 1.9 API Specifications

#### Base URL
- Development: `http://localhost:8000/api/`
- Production: `https://api.rydlink.com/api/`

#### Authentication API

**Register:**
- Endpoint: `POST /api/auth/register/`
- Auth: Public
- Body: `{email, password, phone, first_name, last_name}`
- 201: `{success, data: {user}, meta}`
- 400: `{success, error: {code, message}}`

**Login:**
- Endpoint: `POST /api/auth/login/`
- Auth: Public
- Body: `{email, password}`
- 200: `{success, data: {user, tokens: {access, refresh}}, meta}`
- 400: Invalid credentials

**Refresh:**
- Endpoint: `POST /api/auth/refresh/`
- Auth: Public
- Body: `{refresh_token}`
- 200: `{success, data: {access_token}, meta}`
- 401: Invalid/expired refresh token

**Logout:**
- Endpoint: `POST /api/auth/logout/`
- Auth: Bearer (refresh token)
- Body: `{refresh_token}`
- 200: `{success, meta}`

**Profile:**
- Endpoint: `GET/PUT /api/auth/profile/`
- Auth: Bearer (access token)
- Body (PUT): `{first_name, last_name, profile_picture_url?}`
- 200: `{success, data: {user}, meta}`
- 401: Missing/invalid token

#### Ride API

**Create Ride:**
- Endpoint: `POST /api/rides/`
- Auth: Bearer (access token)
- Body: 
  ```json
  {
    "name": "Weekend Trip",
    "start_lat": 40.7128,
    "start_lng": -74.0060,
    "dest_lat": 40.7589,
    "dest_lng": -73.9851,
    "departure_time": "2026-11-21T14:00:00Z"
  }
  ```
- 201: `{success, data: {ride}, meta}`
- 400: Validation errors (missing fields, invalid coords)

**List Rides:**
- Endpoint: `GET /api/rides/`
- Auth: Bearer
- Query: `?status=active&type=created|invited`
- 200: `{success, data: {rides}, meta}`

**Ride Details:**
- Endpoint: `GET /api/rides/{id}/`
- Auth: Bearer
- 200: `{success, data: {ride}, meta}`
- 404: Ride not found

**Invite Participant:**
- Endpoint: `POST /api/rides/{id}/participants/`
- Auth: Bearer (leader only)
- Body: `{invitee_id, message?}`
- 201: `{success, data: {invitation}, meta}`
- 403: User is not ride leader
- 400: User already participant, invitation already sent

**Update Participant:**
- Endpoint: `PUT /api/rides/{id}/participants/{user_id}/`
- Auth: Bearer (leader only)
- Body: `{role, status}`
- 200: `{success, data: {participant}, meta}`

**Start Ride:**
- Endpoint: `POST /api/rides/{id}/start/`
- Auth: Bearer (leader only)
- 200: `{success, data: {ride}, meta}`

**End Ride:**
- Endpoint: `POST /api/rides/{id}/end/`
- Auth: Bearer (leader only)
- 200: `{success, data: {ride}, meta}`

#### Participant API

**Participant Status:**
- Endpoint: `GET/PUT /api/participants/{ride_id}/status/`
- Auth: Bearer
- Body (PUT): `{status: "joined|left|active"}`
- 200: `{success, data: {participant}, meta}`

**Separation Status:**
- Endpoint: `GET /api/participants/{ride_id}/separation-status/`
- Auth: Bearer
- 200: `{success, data: {separation_status, distance_m, severity}, meta}`

#### Alert API

**List Alerts:**
- Endpoint: `GET /api/alerts/`
- Auth: Bearer
- Query: `?ride_id={id}`
- 200: `{success, data: {alerts}, meta}`

**Alert Details:**
- Endpoint: `GET /api/alerts/{id}/`
- Auth: Bearer
- 200: `{success, data: {alert}, meta}`

#### Meeting Point API

**List Recommendations:**
- Endpoint: `GET /api/meeting-points/`
- Auth: Bearer
- Query: `?ride_id={id}`
- 200: `{success, data: {recommendations}, meta}`

**Approve Meeting Point:**
- Endpoint: `PUT /api/meeting-points/{id}/`
- Auth: Bearer (leader only)
- Body: `{approved: true|false}`
- 200: `{success, data: {recommendation}, meta}`

#### Event API

**Create Event:**
- Endpoint: `POST /api/events/`
- Auth: Bearer
- Body:
  ```json
  {
    "ride_id": "uuid",
    "type": "fuel|food|rest|breakdown|other",
    "lat": 40.7200,
    "lng": -74.0100,
    "note": "Optional note"
  }
  ```
- 201: `{success, data: {event}, meta}`
- 400: Validation errors

**List Events:**
- Endpoint: `GET /api/events/{ride_id}/`
- Auth: Bearer
- 200: `{success, data: {events}, meta}`

#### Emergency API

**Create Emergency:**
- Endpoint: `POST /api/emergencies/`
- Auth: Bearer
- Body:
  ```json
  {
    "ride_id": "uuid",
    "type": "breakdown|emergency",
    "severity": "low|medium|high|critical",
    "description": "Rider experiencing mechanical failure",
    "lat": 40.7150,
    "lng": -74.0080
  }
  ```
- 201: `{success, data: {emergency}, meta}`
- 400: Validation errors

**List Emergencies:**
- Endpoint: `GET /api/emergencies/{ride_id}/`
- Auth: Bearer
- 200: `{success, data: {emergencies}, meta}`

#### Device Token API

**Register Device Token:**
- Endpoint: `POST/PUT /api/device-token/`
- Auth: Bearer
- Body: `{token: "ExpoPushToken[...]", platform: "ios|android"}`
- 200/201: `{success, data: {token}, meta}`

#### Push Notification Categories (Expo)

**Notification Categories:**
- `separation_warning`: "I'm okay" action, "Contact Leader" action
- `high_separation`: "I'm okay" action, "Contact Leader" action, "Report Emergency" action
- `emergency`: "Acknowledge" action, "Resolve" action, "Share Location" action
- `ride_event`: "View Event" action, "Mark Completed" action
- `regrouping`: "Approve" action, "Reject" action, "View Map" action

### 1.10 WebSocket Integration

#### WebSocket Connection URL
- Development: `ws://localhost:8000/ws/`
- Production: `wss://api.rydlink.com/ws/`

#### Connection Parameters (query string)
```
?ride_id={ride_uuid}&user_id={user_uuid}&token={jwt_access_token}
```

#### WebSocket Service Implementation
```javascript
// src/services/websocket.js
import { Alert, Platform } from 'react-native';

let wsConnection = null;
let rideId = null;
let userId = null;
let token = null;
let messageCallback = null;

export const initWebSocket = (ridingRideId, user, jwtToken) => {
  rideId = ridingRideId;
  userId = user.id;
  token = jwtToken;
  
  const wsUrl = Platform.OS === 'ios' 
    ? `wss://api.rydlink.com/ws/?ride_id=${rideId}&user_id=${userId}&token=${token}`
    : `wss://api.rydlink.com/ws/?ride_id=${rideId}&user_id=${userId}&token=${token}`;
  
  wsConnection = new WebSocket(wsUrl);
  
  wsConnection.onopen = () => {
    console.log('WebSocket connected');
  };
  
  wsConnection.onmessage = (event) => {
    const message = JSON.parse(event.data);
    if (messageCallback) {
      messageCallback(message);
    }
    handleIncomingMessage(message);
  };
  
  wsConnection.onclose = () => {
    console.log('WebSocket disconnected');
    // Attempt reconnection
    setTimeout(() => initWebSocket(rideId, user, token), 3000);
  };
  
  wsConnection.onerror = (error) => {
    console.error('WebSocket error:', error);
  };
  
  return wsConnection;
};

export const sendLocationUpdate = (locationData) => {
  if (wsConnection && wsConnection.readyState === WebSocket.OPEN) {
    wsConnection.send(JSON.stringify({
      type: 'location_update',
      ...locationData
    }));
  }
};

export const subscribeMessages = (callback) => {
  messageCallback = callback;
  return () => { messageCallback = null; };
};

export const disconnectWebSocket = () => {
  if (wsConnection) {
    wsConnection.close();
    wsConnection = null;
  }
};

export const sendInvitationResponse = (accepted) => {
  if (wsConnection && wsConnection.readyState === WebSocket.OPEN) {
    wsConnection.send(JSON.stringify({
      type: 'invitation_response',
      ride_id: rideId,
      token: /* invitation token */,
      accepted
    }));
  }
};
```

#### Incoming Message Handling
```javascript
const handleIncomingMessage = (message) => {
  const { type } = message;
  
  switch (type) {
    case 'location_broadcast':
      // Update participant markers on map
      updateParticipantLocations(message.participants);
      break;
      
    case 'separation_warning':
      // Show separation warning UI
      showSeparationWarning(message);
      break;
      
    case 'high_separation':
      // Show high separation alert UI
      showHighSeparationAlert(message);
      break;
      
    case 'regrouped':
      // Show regrouping notification
      showRegroupedNotification(message);
      break;
      
    case 'ride_event':
      // Add event to ride events list
      addRideEvent(message);
      break;
      
    case 'emergency':
      // Show emergency banner
      showEmergencyAlert(message);
      break;
      
    case 'invitation_response':
      // Handle invitation acceptance/rejection
      handleInvitationResponse(message);
      break;
      
    case 'error':
      // Show error message
      showError(message.message);
      break;
      
    default:
      console.log('Unknown WebSocket message type:', type);
  }
};
```

#### Connection Lifecycle
1. Initialize WebSocket when ride lobby screen mounts with ride_id, user_id, and JWT token
2. On connection open, subscribe to location broadcasts and other events
3. On message receive, process based on message type and update state/UI
4. On connection close, attempt automatic reconnection after 3-second delay
5. On explicit disconnect (ride end or leave ride), close WebSocket connection
6. Preserve last known locations during reconnection, resynchronize with backend

#### Reconnection Strategy
- Automatic reconnection with exponential backoff (3s, 5s, 10s, 15s max)
- Upon reconnection, resend last known location update
- Sync participant status with backend (rejoin ride group)
- Restore last known separation status from API
- Show stale location indicator if current location older than 2 minutes

#### Duplicate Message Prevention
- Track last message timestamp and type
- Skip processing if identical message received within 1 second
- Use message `id` field if present (for alert/confirmation messages)
- Debounce location updates (ignore if received faster than 2-second interval)

#### Error Handling
- Authentication failure: Close connection, show auth error, redirect to login
- Invalid ride_id: Show "Invalid ride" message, disconnect
- Participant not in ride: Show "Not a participant" message, disconnect
- Rate limiting: Throttle location_update messages (>10/s)
- Network disconnect: Show stale-location indicator, resume on reconnection
- Server error: Display error banner, offer retry option

### 1.11 Route Visualization

#### MapLibre Integration
- Map style: OpenStreetMap compatible tile source
- Attribution: Required OSM attribution
- Map type: `maplibre-style-streets-vectors` or custom OSM style

#### Route Retrieval and Rendering
1. On ride details load, retrieve route via OSRM API
2. OSRM request: `GET /route/v1/driving/{start_lng},{start_lat};{dest_lng},{dest_lat}?overview=full`
3. Response contains: `geometry` (encoded polyline), `distance`, `duration`
4. Decode polyline using `maplibre-gl-utility` or custom decoder
5. Render polyline on MapLibre `<MapboxGL.Map>` component
6. Style polyline: `#FF6B6B` color, `4px` width, `opacity: 0.8`

#### Markers
- **Current user marker**: Custom icon with user avatar, pulsing animation
- **Ride Leader marker**: Crown icon, distinct color (`#4A90E2`)
- **Destination marker**: Target icon at destination coordinates
- **Starting location marker**: Home pin at start coordinates
- **Participant markers**: Avatar-based icons, color-coded by role
- **Stale marker**: Semi-transparent with "stale" badge (>2 min old)
- **Meeting point marker**: Star icon when recommendations available

#### Distance and ETA Display
- Calculate remaining distance from current position to destination
- Display estimated travel time based on current speed
- Update dynamically as participant moves
- Show total route distance at ride start

#### Route Deviation Visualization
- Monitor participant distance from planned route
- Display deviation distance when off-route
- Color-code: Green (on-route, ≤50m), Yellow (slight deviation, 50-200m), Red (significant deviation, >200m)
- Recalculate ETA based on current position and remaining route

#### Map Controls
- Current location button (center map on GPS)
- Zoom controls (plus/minus)
- Map type selector (standard, satellite, hybrid)
- Compass rotation (matches device orientation)
- Pitch control (3D terrain view optional)

#### Map Permissions
- Foreground location permission (always required)
- Background location permission (development build required for iOS)
- Camera permission (MapLibre tile rendering)
- Notification permission (push notifications)

#### Map Loading and Error States
- **Loading state**: Full-screen spinner with "Loading map..." text
- **Error state**: "Failed to load map" with retry button
- **No internet**: "Map tiles unavailable. Using cached data."
- **Attribution error**: Missing OSM attribution displayed in banner
- **Style load failure**: Fallback to basic OpenStreetMap configuration

### 1.12 GPS Location Integration

#### Expo Location Configuration
- SDK: `expo-location` Latest
- Requires: Expo development build (bare React Native) for full functionality
- Expo Go limited to foreground location only

#### Permission Requirements

**Foreground Permission:**
- Request on app launch when user first enables location features
- Message: "RydLink would like to access your location to coordinate group rides"
- iOS: Requires NSLocationWhenInUseUsageDescription in Info.plist
- Android: Requires ACCESS_FINE_LOCATION in AndroidManifest.xml

**Background Permission:**
- Request after user grants foreground permission
- Message: "RydLink would like to access your location even when the app is in the background"
- iOS: Requires NSLocationAlwaysUsageDescription, UIBackgroundModes.location in Info.plist
- Android: Requires ACCESS_BACKGROUND_LOCATION permission (runtime Android 10+)
- Background tracking only works with Expo development build, not Expo Go

#### GPS Configuration
| Parameter | Foreground | Background |
|-----------|------------|------------|
| Update interval | Every 5–10 seconds | Every 30–60 seconds |
| Accuracy requirement | ≤ 30 metres for evaluation | No explicit constraint, but stale locations filtered |
| Timestamp freshness | < 2 minutes | < 2 minutes (enforced by backend) |
| Required fields | lat, lng, accuracy, speed, heading, timestamp | lat, lng, timestamp |
| Distance filter | N/A | Minimum 10m movement to reduce battery drain |

#### Current Location Retrieval
```javascript
// src/hooks/useLocation.js
import { useState, useEffect, useCallback } from 'react';
import * as Location from 'expo-location';

export const useLocation = () => {
  const [location, setLocation] = useState(null);
  const [accuracy, setAccuracy] = useState(null);
  const [isTracking, setIsTracking] = useState(false);
  const [hasPermission, setHasPermission] = useState(false);
  const [error, setError] = useState(null);

  // Request permissions
  const requestPermissions = useCallback(async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    setHasPermission(status === 'granted');
    
    if (status !== 'granted') {
      // On iOS, also request always permission
      const { status: alwaysStatus } = await Location.requestAlwaysPermissionsAsync();
      setHasPermission(alwaysStatus === 'granted');
    }
  }, []);

  // Get current location
  const getCurrentLocation = useCallback(async () => {
    if (!hasPermission) {
      await requestPermissions();
      if (!hasPermission) return;
    }
    
    try {
      const current = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High,
        timeout: 20000,
        maximumAge: 5000,
        distanceFilter: 10,
      });
      setLocation(current.coords);
      setAccuracy(current.coords.accuracy);
      return current.coords;
    } catch (e) {
      setError(e);
      console.error('Location error:', e);
    }
  }, [hasPermission]);

  // Start background tracking
  const startTracking = useCallback(async () => {
    if (isTracking) return;
    
    try {
      const status = await Location.hasBackgroundPermissionAsync();
      if (!status) {
        // Request background permission (iOS: after foreground grant)
        await Location.requestBackgroundPermissionsAsync();
      }
      
      const task = await Location.startBackgroundLocationUpdateAsync();
      setIsTracking(true);
      
      // Listen for location updates
      Location.addLocationListener((loc) => {
        setLocation(loc.coords);
        // Send via WebSocket
        sendLocationUpdate({
          latitude: loc.coords.latitude,
          longitude: loc.coords.longitude,
          accuracy: loc.coords.accuracy,
          speed: loc.coords.speed,
          heading: loc.coords.heading,
          timestamp: loc.timestamp,
        });
      });
    } catch (e) {
      console.error('Background tracking error:', e);
    }
  }, [isTracking, hasPermission]);

  return {
    location,
    accuracy,
    isTracking,
    hasPermission,
    error,
    requestPermissions,
    getCurrentLocation,
    startTracking,
  };
};
```

#### GPS Accuracy Validation (Frontend)
```javascript
// In location update sender
const validateGPS = (coords) => {
  if (!coords) return false;
  
  // Check accuracy threshold
  const accuracyThreshold = 30; // metres
  if (coords.accuracy > accuracyThreshold) {
    // Send last valid location instead
    return false;
  }
  
  // Check for invalid values
  if (isNaN(coords.latitude) || isNaN(coords.longitude)) {
    return false;
  }
  
  // Check latitude/longitude ranges
  if (coords.latitude < -90 || coords.latitude > 90) return false;
  if (coords.longitude < -180 || coords.longitude > 180) return false;
  
  // Check speed reasonability
  if (coords.speed !== null && (coords.speed < 0 || coords.speed > 100)) {
    // Could indicate GPS error, but don't reject - use last valid
  }
  
  return true;
};
```

#### Background Tracking Battery Considerations
- iOS: Background location significantly impacts battery; limit to essential rides
- Android: Use `distanceFilter: 50` to reduce updates when stationary
- Both platforms: Pause tracking when ride is paused/ended
- Show battery usage warning after 2 hours of continuous tracking
- Allow user to manually disable background tracking per ride

#### Android-Specific Requirements
- Android 8.0 (API 26)+: Separate foreground/background permission requests
- Android 12 (API 31)+: Approximate location location settings
- Foreground service notification required for background tracking
- Manifest entries required:
  ```xml
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
  <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
  ```
- Debugging: Use `adb shell dumpsys battery` to monitor battery drain
- Testing: Use Android Studio Location Mockup plugin for GPS simulation

#### Application Lifecycle Handling
- Foreground → Background: Continue tracking with reduced frequency
- Background → Foreground: Sync last location, resume normal frequency
- App terminated: Location updates stop; user must restart tracking
- Incoming call: iOS pauses location, Android may continue depending on settings
- Multitasking: iOS limits background time (~18 minutes), Android more flexible

### 1.13 Real-Time Participant Tracking

#### WebSocket Connection
- Connect when ride lobby screen mounts
- Authenticate with ride_id, user_id, JWT token
- Join ride-specific WebSocket group `ride_{ride_id}`
- Maintain connection throughout active ride

#### Location Transmission
1. Frontend acquires GPS coordinates via Expo Location
2. Validate accuracy (≤ 30m) and freshness (< 2 min)
3. Send `location_update` WebSocket message every 5-10s (foreground) or 30-60s (background)
4. Message includes: latitude, longitude, accuracy, speed, heading, timestamp
5. Backend validates and stores location update
6. Backend broadcasts `location_broadcast` to all ride participants

#### Participant Location Reception
1. WebSocket receives `location_broadcast` message
2. Update Redux/Rx state with new participant positions
3. Update MapLibre markers for each participant
4. Update separation status indicators
5. Handle stale location indicators (>2 min old)

#### Marker Updates
```javascript
// Reducer or state update
const updateParticipants = (state, newParticipants) => {
  return {
    ...state,
    participants: newParticipants.map(participant => ({
      ...participant,
      lastSeen: new Date().toISOString(),
      separationZone: calculateZone(
        participant.latitude,
        participant.longitude,
        state.groupReference
      ),
    })),
  };
};
```

#### Location Synchronization
- On WebSocket connect, request current participant locations from API
- Sync with last known locations from WebSocket broadcast
- If connection lost, rely on last stored location from API
- On reconnection, resynchronize state with backend

#### Connection Status
- Display connection indicator: Connected/Disconnected
- Show "Live location sharing active" when WebSocket connected
- Show "Location sharing paused" when disconnected
- Handle reconnection gracefully without data loss

#### Multiple Participant Handling
- Limit displayed participants to ride maximum (50)
- Cluster markers when many participants near each other
- Show participant list with status indicators
- Prioritize leader and nearest participants on map
- Update all participants simultaneously via WebSocket broadcast

#### Stale Location Indicators
- Marker shows "⚠️ Stale" badge when location >2 minutes old
- Semi-transparent marker icon
- Tooltip: "Location update paused - tap to resume"
- Separation detection uses last valid location if stale
- UI shows "Live location paused" banner when no updates for 3 minutes

### 1.14 Separation Alert Interface

#### Zone Status Display
| Zone | Distance Range | Color | Visual Indicator |
|------|----------------|-------|------------------|
| Normal | 0–300 m | `#4CAF50` (Green) | No alert badge |
| Monitoring | 300–500 m | `#FFC107` (Yellow) | "Monitoring" text |
| Separation Warning | >500 m | `#FF9800` (Orange) | "Warning" badge after 60s |
| High Separation | >1,000 m | `#F44336` (Red) | "High Alert" badge after 30s |

#### Separation Alert Components

**Separation Banner** (top of screen when alert active):
- Alert type icon (⚠️ for warning, ! for high)
- Distance from group display: "520 m from group"
- Severity color bar (orange/red gradient)
- "I'm okay" button (resets timer if within zone)
- "Contact Leader" button (opens chat/alert details)
- "Dismiss" button

**Distance Display**
- Real-time distance: "342 m"
- Moving distance vs. stationary distance
- Last updated timestamp: "Updated 23s ago"
- GPS accuracy: "Accuracy: 5 m"

#### Status Indicators

**Normal State** (0-300 m):
- Green status badge: "Within range"
- No alert sounds or banners
- Participant marker: Green icon

**Monitoring State** (300-500 m):
- Yellow status badge: "Monitoring"
- Timer display: "3:21" (remaining in zone)
- Participant marker: Yellow/orange icon
- No alert generated yet

**Separation Warning State** (>500 m, 60s timer):
- Orange status badge: "Separation Warning"
- Timer countdown: "58s in warning zone"
- Red participant marker pulsing
- Alert banner shown at top of screen
- Push notification received (if enabled)

**High Separation State** (>1,000 m, 30s timer):
- Red status badge: "High Separation"
- Timer countdown: "29s until alert"
- Critical red participant marker pulsing fast
- High-priority alert banner
- High-priority push notification
- Leader notified via app and notification

#### Alert Resolution
- Rider returns within 300 m of group
- Timer starts for regroup confirmation (30 seconds)
- If rider remains within 300 m for 30 seconds: alert auto-resolves
- Status changes to "Regrouped"
- `regrouped` WebSocket broadcast to all participants
- Marker changes to green "back with group"
- Alert banner dismissed with success message

#### "I'm Okay" Button
- Manually reset separation timer when rider is within warning/high zone
- Only available when distance allows timer reset
- Sends API request: `PUT /api/participants/{ride_id}/status/`
- Timer resets, status returns to monitoring or normal
- Prevents false alerts when rider intentionally stops/rests
- Logs action in ride events (optional)

#### Contact Leader Button
- Opens rider-to-leader chat interface
- Pre-filled message: "I'm experiencing separation, need assistance"
- One-tap options: "Share location for 5 min", "Request regrouping point", "Call leader"
- Leader receives notification of contact request
- Logs communication in ride events

### 1.15 Regrouping Recommendation Interface

#### Recommended Meeting Point
- Display: "Recommended meeting point: Coffee Shop, 2.3 km away"
- Estimated travel time for each participant: "You: 8 min, Bob: 12 min, Carol: 10 min"
- Mode of travel: Driving / Walking / Cycling (default: driving)
- Map marker with star icon at recommendation location

#### Candidate Locations Discovery
1. Ride Leader initiates regrouping recommendation
2. Backend discovers candidate POIs via Overpass API
3. Filter by: Fuel stations, Restaurants, Parking areas
4. Search radius: 5 km from group centroid (configurable)
5. Exclude: Locations outside ride route (optional)
6. Validate each candidate location

#### Candidate Location Validation

**Fuel Stations:**
- Overpass query: `amenity=fuel`
- Must have: `refuel:diesel` or `refuel:petrol` tags
- Open status check (if possible via website)
- Parking availability consideration

**Restaurants/Cafes:**
- Overpass query: `amenity=cafe` or `amenity=restaurant`
- Minimum rating filter (e.g., ≥ 4.0 stars)
- Open now consideration (current time vs. opening hours)
- Seating availability (secondary)

**Parking Areas:**
- Overpass query: `parking=yes` or `parking_lot`
- Free vs. paid parking consideration
- Capacity information (if available)
- Proximity to group centroid

#### OSRM Travel Time Calculation
1. For each candidate location and each participant:
   - Calculate OSRM route: `/route/v1/driving/{participant_lng},{participant_lat};{candidate_lng},{candidate_lat}?overview=false`
   - Extract `duration` from route summary
   - Add walking time from participant location to route start (if needed)
   - Add walking time from route end to meeting point (if needed)

2. Example calculation:
   - Participant A to Candidate X: 5 min
   - Participant B to Candidate X: 8 min
   - Participant C to Candidate X: 12 min
   - Worst-case ETA: 12 min

#### Meeting Point Ranking Algorithm
1. Calculate ETA for each participant to each candidate
2. Determine worst-case ETA per candidate (max across all participants)
3. Rank candidates by worst-case ETA (ascending = best)
4. Apply suitability filter:
   - Remove candidates with ETA > 30 minutes (configurable)
   - Remove candidates with no available routes
   - Remove candidates too far from group route (route deviation > 2 km)

5. Final ranking by:
   - Primary: Worst-case ETA (ascending)
   - Secondary: Average ETA (ascending)
   - Tertiary: Proximity to group route
   - Quaternary: Candidate type preference (parking > restaurant > fuel)

#### Example Calculation
```
Ride with 3 participants: Alice, Bob, Carol
Group centroid: (40.7200, -74.0080)

Candidate 1: Gas Station at (40.7250, -74.0100)
- Alice: 4 min, Bob: 6 min, Carol: 9 min
- Worst-case: 9 min
- Average: 6.3 min
- Route deviation: 1.2 km

Candidate 2: Cafe at (40.7300, -73.9950)
- Alice: 7 min, Bob: 5 min, Carol: 15 min (no direct route)
- Worst-case: ∞ (unavailable)
- Filtered out

Candidate 3: Parking at (40.7150, -74.0000)
- Alice: 3 min, Bob: 4 min, Carol: 5 min
- Worst-case: 5 min
- Average: 4 min
- Route deviation: 0.5 km

Ranking: Parking (5 min worst) > Gas Station (9 min worst)
Selected: Parking at (40.7150, -74.0000)
```

#### Ride Leader Approval
- Recommendation presented to Ride Leader on lobby/sidebar
- Leader sees: candidate name, type, worst-case ETA, average ETA, locations on map
- Leader actions: "Approve", "Reject", "Request More Options"
- On approve: 
  - `PUT /api/meeting-points/{id}/` with `{approved: true}`
  - WebSocket `meeting_point_approved` broadcast
  - All participants receive recommendation with approved flag
  - Map marker shown at approved location
  - ETA updates for each participant

#### Participant Notification
- Notification received when recommendation generated
- Banner: "Regrouping recommendation available - 5 min worst-case travel"
- Tap opens map with candidate location marked
- Participant can view details and ETA
- Participant can approve/reject (if leader enables)
- Notification dismissed when meeting point reached

#### Meeting Point Updates
- Recommendations refresh every 5 minutes (configurable)
- New candidates added as participants move
- Candidates removed if unreachable or offline
- Leader can request new recommendations at any time
- Approved meeting point pinned to map until ride ends
- Updates broadcast via WebSocket to all participants

#### Regrouping Workflow
1. Separation warning/high separation triggered
2. Rider returns within 300 m (auto or manual)
3. System detects regrouping condition
4. Leader initiates "Find Meeting Point" or system auto-generates
5. Candidates displayed with ETAs
6. Leader approves meeting point
7. All participants navigated to meeting point
8. Upon arrival, separation alert auto-resolves
9. `regrouped` WebSocket broadcast to all participants
10. Ride continues with normal monitoring

### 1.16 Ride Event and Stop Interface

#### Event Creation Interface
- **Button**: "+ Add Event" on ride lobby or live map
- **Event Type Selector**: Fuel, Food, Rest, Breakdown, Other
- **Location Picker**: Current location, map selection, or GPS coordinates
- **Optional Fields**: 
  - Planned time (future event)
  - Note/description (e.g., "Gas station on right")
  - Photo attachment (optional)

#### Event List Screen
- **List format**: Card-style or list view
- **Event entries**: Type icon, time/location, status badge
- **Filter options**: All, Planned, Completed, Today
- **Swipe actions**: Mark completed, Edit, Cancel

#### Event Details
- Event type icon (fuel pump, fork, bed, warning triangle)
- Location on mini map or address
- Planned time (if set)
- Status: "Planned", "In progress", "Completed", "Cancelled"
- Creator display name
- Created at timestamp

#### Event Location
- Latitude/longitude stored with event
- Map representation: Mini map or full-screen on tap
- Address reverse-geocoded from coordinates
- Distance from current user location (if applicable)

#### Event Updates
- Leader can edit planned time or location
- Participant can mark event as "Completed" (if enabled)
- Leader can cancel planned events
- Updates broadcast via WebSocket to all participants
- Local notification reminder (if planned time set)

#### Event Cancellation
- Leader cancels event: All participants notified
- Participant cancels own event: Local only, unless leader enabled
- Cancelled events shown with "Cancelled" strikethrough
- Reason optional (dropdown: "No longer needed", "Location changed", "Other")

#### Map Representation
- Event markers on map with type-specific icons
- Pulsing animation for upcoming events (within 1 hour)
- Clustering when many events on screen
- Tap event marker: Show event details card
- Route integration: Event marker on planned route polyline

### 1.17 Emergency Management Interface

#### Emergency Reporting
- **Emergency Button**: Prominent red button on ride lobby and live map
- **Emergency Categories**:
  - `breakdown`: Vehicle mechanical failure, flat tire, etc.
  - `emergency`: Medical emergency, accident, safety threat
- **Emergency Location**: Auto-GPS, map selection, or manual input
- **Emergency Description**: Free text (optional but recommended)
- **Severity Selection**: Low, Medium, High, Critical (pre-defined)

#### Breakdown Reporting
- Category: `breakdown`
- Severity: Low (minor issue), Medium (requires assistance), High (stranded), Critical (safety risk)
- Description fields: "Flat tire", "Engine overheating", "Out of fuel", etc.
- Location: Auto-GPS at time of report
- Additional: Can passenger continue, need relay ride, etc.

#### Emergency Categories (Database enum)
```sql
-- ride_events type check
CHECK (type IN ('fuel','food','rest','breakdown','other'))

-- emergency_reports type check
CHECK (type IN ('breakdown','emergency'))

-- emergency_reports severity check  
CHECK (severity IN ('low','medium','high','critical'))
```

#### Emergency Location
- Auto-GPS coordinates at time of report
- User can adjust location on map
- Address reverse-geocoded and displayed
- Share location option (via WebSocket to all participants)
- Last known location if GPS unavailable

#### Emergency Details Form
```
Report Emergency
---------
Category: [breakdown] or [emergency]
Severity: [low] [medium] [high] [critical] (required)
Description: [free text, 200 chars max]
Location: [auto-GPS, adjustable on map]
---------
[Send Emergency Report] [Cancel]
```

#### Emergency Notification
- High-priority push notification to all ride participants
- WebSocket `emergency` broadcast to all ride groups
- Frontend: Full-screen emergency banner with:
  - Category icon
  - Severity indicator (red/orange/yellow/green)
  - Description text
  - Rider name and profile picture
  - Location map snapshot
  - "Acknowledge" button
  - "Share Location" button (5 min)
  - "Contact Services" button (optional)

#### Emergency Acknowledgement
- Participants tap "Acknowledge" to confirm receipt
- Acknowledgment stored in `emergency_reports.acknowledged_at`
- WebSocket `emergency_acknowledged` broadcast
- Leader dashboard shows acknowledgment status
- Follow-up: Repeat alert if no acknowledgment within 2 minutes

#### Emergency Resolution
- Rider or leader marks emergency as resolved
- Resolution stored in `emergency_reports.resolved_at`
- WebSocket `emergency_resolved` broadcast
- Frontend emergency banner removed
- Ride events log: "Emergency resolved at [time]"
- Historical record kept for 90 days post-ride

#### Emergency History
- Viewable in ride history screen
- List of all emergencies on ride
- Filter by: severity, status, date
- Details: report time, acknowledgment time, resolution time
- Participant involvement summary
- Data retention: 90 days completed ride, then archived

### 1.18 Notification Interface

#### Notification List Screen
- **Format**: Scrollable list, newest first
- **Categories tabs**: All, Invitations, Separation Alerts, Ride Events, Emergencies
- **Search/filter**: By ride, by date, by type
- **Empty state**: "No notifications yet"

#### Notification Item Design
| Field | Display |
|-------|---------|
| Type icon | Category-specific icon |
| Title | Short summary (e.g., "Separation Warning") |
| Body | Detailed message (e.g., "Alice separated by 500m") |
| Time ago | "2 min ago", "Yesterday" |
| Ride indicator | Ride name or "General" |
| Read indicator | Bold (unread), subtle (read) |

#### Read/Unread Status
- On tap: Mark as read, navigate to related ride/event
- Visual indicator: Dot/badge on notification card
- Mark-all-read button on list header
- Backend: `PUT /api/notifications/{id}/read/` or `PUT /api/notifications/read-all/`

#### Notification Categories

**Invitation Notifications**
- Type: `invitation_received` or `invitation_accepted`/`rejected`
- Title: "New ride invitation from [Name]"
- Body: "Alice invited you to 'Weekend Trip'"
- Action: View ride, Accept/Reject

**Separation Alerts**
- Type: `separation_warning` or `high_separation`
- Title: "Separation Warning" / "High Separation Alert"
- Body: "Alice is 520m from group"
- Action: View map, Contact leader

**Ride Event Notifications**
- Type: `ride_event`
- Title: "Fuel stop added" / "Rest stop planned"
- Body: "Alice added fuel stop at [location]"
- Action: View event, Mark completed

**Emergency Notifications**
- Type: `emergency`
- Title: "Emergency reported" / "Breakdown reported"
- Body: "[Name] reported [type] - [severity]"
- Action: Acknowledge, Share location, Contact services

**Regrouping Notifications**
- Type: `regrouped`
- Title: "Rider has regrouped"
- Body: "Alice has returned to the group"
- Action: None needed, resume normal tracking

#### Push Notification Handling

**Foreground Notification Handling**
```javascript
// App is foreground, show in-app notification
Notifications.getExpoNotificationResponseAsync().then(notificationResponse => {
  if (notificationResponse) {
    const { requestId } = notificationResponse;
    // Navigate to related screen
    switch (notificationResponse.notification.title) {
      case 'Emergency':
        navigation.navigate('EmergencyScreen');
        break;
      case 'Separation Warning':
        navigation.navigate('LiveMapScreen');
        break;
    }
  }
});
```

**Background Notification Handling**
- Expo Notifications delivers to device even when app closed
- User taps notification: Opens app to related screen
- Notification permission required for foreground delivery
- FCM integration for production Android push
- EAS Build required for iOS push notification handling

#### Notification Delivery Status
- Backend tracks: sent, delivered, read
- WebSocket confirms message receipt
- retry on delivery failure (max 3 attempts)
- Dead letter queue for permanently failed tokens
- Monthly delivery report for ride leaders

#### Device Token Registration
1. User grants push notification permission
2. App calls `Notifications.getExpoPushTokenAsync()`
3. Token received: `ExpoPushToken[xxxx-xxxx-xxxx]`
4. Token sent to backend: `POST /api/device-token/`
5. Backend stores in `device_tokens` table
6. Token associated with user account
7. On logout: Token revoked or marked inactive
8. On app uninstall: Remove token from server (optional)

#### Foreground vs Background Notification
- **Foreground**: App open, notification shown in app's notification list
  - Controlled via `Notifications.presentNotificationAsync()`
  - Custom layout and action buttons
  - No system notification sound required
- **Background**: App closed/minimized
  - System delivers via Expo/FCM
  - Default sound and vibration
  - User taps to open app
  - Required for critical alerts (emergencies)

#### Notification Preferences
Per-user settings (stored in user profile or separate prefs):
- `[ ]` Separation warning notifications
- `[ ]` High separation alerts
- `[ ]` Regrouping recommendations
- `[ ]` Ride event notifications
- `[ ]` Emergency alerts (always on)
- `[ ]` Invitation notifications
- `[ ]` Sound enabled/disabled
- `[ ]` Vibration enabled/disabled
- `[ ]` Visual only (no sound/vibration)

#### In-App vs WebSocket vs Push Notification Distinction
| Type | When Delivered | Persistence | Use Case |
|------|---------------|-------------|----------|
| **In-App** | App open/foreground | Until read | Real-time status updates, lists, temporary alerts |
| **WebSocket** | App open, WebSocket connected | Until app close/clear | Live location, real-time broadcasting, immediate feedback |
| **Push (OS)** | App closed/background | Until read/cleared | Critical alerts, invitations, emergencies, always-on notifications |

### 1.19 Ride History Interface

#### Completed Rides List
- **Format**: Card or list view, grouped by date
- **Each ride card**: 
  - Ride name and date
  - Duration and distance
  - Participant count
  - Summary of events and alerts
  - Status badge (completed/cancelled)

#### Ride Details Screen
- **Header**: Ride name, dates, total distance, duration
- **Route summary**: Polyline overview with start/end markers
- **Participants list**: All who participated with roles
- **Events summary**: Fuel stops, food stops, rest stops, events
- **Alerts summary**: Separation warnings, high separation, emergencies
- **Route map**: Full-screen MapLibre with complete route

#### Participant Information
- Rider list with avatars/names
- Role: Leader or Participant
- Join/leave timestamps
- Separation status history (if applicable)
- Total distance traveled per participant

#### Route Summary
- Total distance: "4.2 km"
- Total duration: "2 hours 15 min"
- Average speed: "3.2 km/h"
- Elevation gain (if tracked)
- Route visualization on map

#### Historical Information
- Ride completion time
- Average separation distance during ride
- Number of alerts generated
- Number of events recorded
- Emergency reports (if any)
- Distance traveled vs. planned route

#### Ride History Filtering
- By date range: Today, This week, This month, Custom range
- By events: With alerts, Without alerts, With emergencies only
- By participants: My rides, Rides I led, Specific participant
- By status: Completed, Cancelled, All