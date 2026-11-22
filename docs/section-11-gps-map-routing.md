# Section 11: GPS, Map and Routing Design

## 11.1 GPS Coordinate Acquisition

- **Foreground:** `expo-location` `getCurrentPositionAsync` + `watchPositionAsync`, interval 5–10 seconds, highest accuracy
- **Background:** `expo-task-manager` registered task, interval 30–60 seconds, requires foreground service (Android) / background mode (iOS)
- **Permissions:** `ACCESS_FINE_LOCATION`, `ACCESS_BACKGROUND_LOCATION` (Android), `NSLocationAlwaysAndWhenInUseUsageDescription` (iOS)
- **Accuracy check:** Reject updates with accuracy > 30 metres or timestamp older than 2 minutes. Updates with accuracy > 30 m are ignored for separation evaluation but still displayed on map.

## 11.2 Location Update Transmission

- Each valid location → WebSocket `location.update` message
- Include: ride_id, lat, lng, accuracy, speed, heading, recorded_at
- Backend validates GPS accuracy (≤ 30 metres) and timestamp freshness (< 2 minutes) before using for separation evaluation
- Inaccurate or stale GPS readings are stored but marked as `ignored_for_separation: true`
- If WebSocket disconnected: cache locally, resend on reconnect with original timestamps
- Map marker colour indicates separation status: green (normal), yellow (monitoring), orange (separation warning), red (high separation)

## 11.3 Map Marker Updates

- MapLibre receives `location.broadcast` messages
- Marker per participant: avatar, name, speed, last-updated timestamp
- Stale indicator: marker turns gray if no update > 2 minutes; offline if > 5 minutes
- Own marker: distinct color/style

## 11.4 Planned Route Display

- Route polyline from `ride_routes.geometry` (GeoJSON LineString)
- Rendered on MapLibre beneath participant markers
- Distance and ETA displayed in ride header

## 11.5 Distance and ETA Calculation

- OSRM: `GET /route/v1/driving/{start_lng},{start_lat};{dest_lng},{dest_lat}?overview=full&geometries=geojson`
- Returns: polyline, distance (meters), duration (seconds)
- Stored in `ride_routes` table at ride creation

## 11.6 Nearby Points of Interest

- Overpass API query for amenities: `fuel`, `restaurant`, `parking` within radius (default 5km)
- Returns POI name, type, lat/lng, address
- Used by regrouping algorithm (Module 8)

## 11.7 Service Responsibilities and Limitations

| Service | Responsibility | Limitation |
|---------|---------------|------------|
| MapLibre | Map rendering, markers, polylines | Client-side; needs tile source |
| OpenStreetMap | Map data and tiles | Public tiles need attribution; rate limits; not guaranteed production SLA |
| OSRM | Route calculation, distance, ETA | Public demo server rate-limited; production needs self-hosted `.pbf` road network |
| Overpass API | Nearby POI search | Public instances rate-limited; production needs self-hosting |
| Nominatim | Geocoding/reverse (optional) | 1 req/sec max, attribution required, not for production |
