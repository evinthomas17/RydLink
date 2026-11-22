# Section 12: Intelligent Monitoring and Recommendation Algorithms

Rule-based algorithms implemented in Python. No machine learning.

## 12.1 Group Separation Detection

**Logic:**
- Reference: group centroid (average of all participant locations) OR distance from leader
- GPS validation: accept readings with accuracy ≤ 30 m; ignore inaccurate readings
- Location freshness: participant location must be < 2 minutes old
- Do not classify disconnected participant as separated solely because updates stopped
- Distance calculation: Haversine formula or PostGIS `ST_Distance` in metres
- Group reference position: geographic centroid of active participants; handles distributed groups

**Prototype thresholds (configurable):**
| Condition | Distance | Confirmation Duration | System Behaviour |
|-----------|----------|----------------------|------------------|
| Normal | 0–300 m | — | No separation alert |
| Monitoring | 300–500 m | — | Monitor rider's movement |
| Separation Warning | > 500 m | 60 seconds | Generate a separation warning |
| High Separation | > 1,000 m | 30 seconds | Generate a high-separation alert |
| Regrouped | Within 300 m | 30 seconds | Automatically resolve the separation alert |

**Pseudocode:**
```python
def detect_separation(participants, group_ref, config):
    """
    Detect rider separation from group.
    
    Args:
        participants: dict of user_id -> {lat, lng, timestamp, accuracy, speed}
        group_ref: dict with lat/lng of group centroid
        config: dict with threshold configuration
    
    Returns:
        list of alert dicts
    """
    alerts = []
    now = datetime.utcnow()
    
    for user_id, rider in participants.items():
        # Skip if location too stale
        if now - rider['timestamp'] > timedelta(minutes=2):
            continue
        
        # Skip if GPS accuracy too poor
        if rider['accuracy'] > config['gps_accuracy_threshold_m']:
            continue
        
        # Calculate distance from group reference
        distance_m = haversine(rider['lat'], rider['lng'], 
                               group_ref['lat'], group_ref['lng'])
        
        # Get current separation status
        status = rider.get('separation_status', 'normal')
        timer = rider.get('separation_timer_started_at')
        
        if distance_m <= config['normal_distance_m']:
            # Rider is back within normal range
            if status != 'normal':
                # Rider has regrouped
                alerts.append({
                    'type': 'regrouped',
                    'user_id': user_id,
                    'distance': distance_m,
                    'severity': 'info',
                    'message': f'Rider has regrouped ({distance_m:.0f}m from group)'
                })
            # Reset to normal
            status = 'normal'
            timer = None
            
        elif config['normal_distance_m'] < distance_m <= config['monitoring_distance_m']:
            # Monitoring zone
            if status != 'monitoring':
                status = 'monitoring'
                timer = now
            elif timer and (now - timer).seconds >= config['monitoring_confirmation_s']:
                # Stay in monitoring, no new alert
                pass
            else:
                # Still entering monitoring zone
                status = 'monitoring'
                timer = now
                
        elif config['monitoring_distance_m'] < distance_m <= config['separation_warning_distance_m']:
            # Separation Warning zone
            if status != 'separation_warning':
                status = 'separation_warning'
                timer = now
            elif timer and (now - timer).seconds >= config['separation_warning_confirmation_s']:
                # Generate Separation Warning
                alerts.append({
                    'type': 'separation_warning',
                    'user_id': user_id,
                    'distance': distance_m,
                    'severity': 'warning',
                    'message': f'Separation Warning: {distance_m:.0f}m from group'
                })
                # Reset timer to prevent duplicate alerts
                timer = now
            else:
                # Still in warning zone, extend timer
                timer = now
                
        elif distance_m > config['high_separation_distance_m']:
            # High Separation zone
            if status != 'high_separation':
                status = 'high_separation'
                timer = now
            elif timer and (now - timer).seconds >= config['high_separation_confirmation_s']:
                # Generate High Separation Alert
                alerts.append({
                    'type': 'high_separation',
                    'user_id': user_id,
                    'distance': distance_m,
                    'severity': 'high',
                    'message': f'High Separation Alert: {distance_m:.0f}m from group'
                })
                # Reset timer to prevent duplicate alerts
                timer = now
            else:
                # Still in high separation, extend timer
                timer = now
        
        # Update rider state
        rider['separation_status'] = status
        rider['separation_distance'] = distance_m
        rider['separation_timer_started_at'] = timer
    
    return alerts
```

## 12.2 Timer Management

- **Confirmation timer** starts when rider crosses into a new distance zone
- Timer **pauses/resets** when rider returns below the relevant threshold
- **Duplicate alert prevention**: After generating an alert, reset timer to prevent repeated alerts for same event
- **Timer state** persisted in `ride_participants.separation_timer_started_at`

## 12.3 Automatic Regrouping Detection

- When separated rider returns within 300 m of group, start regrouping confirmation timer
- If rider remains within 300 m continuously for 30 seconds, auto-resolve active separation alert
- Update rider's `separation_status` to `normal`
- Send `separation.regrouped` notification via WebSocket to all participants
- Reset detection timers after resolution

## 12.4 Alert Management

### Alert Types

| Alert Type | Severity | Trigger Condition |
|------------|----------|-------------------|
| `separation_warning` | warning | Rider > 500 m from group for 60 continuous seconds |
| `high_separation` | high | Rider > 1,000 m from group for 30 continuous seconds |
| `regrouped` | info | Rider within 300 m of group for 30 continuous seconds |

### Alert Fields

| Field | Description |
|-------|-------------|
| `alert_id` | Unique identifier (UUID) |
| `ride_id` | Associated ride |
| `user_id` | Affected participant |
| `type` | Alert type (`separation_warning`, `high_separation`, `regrouped`) |
| `severity` | Alert severity (`warning`, `high`, `info`) |
| `detected_distance` | Distance from group at detection (metres) |
| `detection_timestamp` | When alert was generated |
| `resolution_timestamp` | When alert was resolved (for `regrouped`) |
| `message` | Human-readable alert message |

### Alert Distribution

- **Separation Warning**: Ride Leader + affected rider receive WebSocket + push notification
- **High Separation Alert**: Ride Leader + affected rider receive high-priority WebSocket + push notification
- **Regrouped**: All ride participants receive `separation.regrouped` WebSocket notification

## 12.5 Integration with Architecture

```
React Native (Expo Location)
    ↓ GPS coordinates (5–10s foreground, 30–60s background)
WebSocket → Django Channels → Monitoring Engine
    ↓ GPS validation (accuracy ≤ 30 m, timestamp freshness < 2 min)
    ↓ Group reference calculation (centroid of active participants)
    ↓ Distance calculation (Haversine/PostGIS ST_Distance)
    ↓ Threshold evaluation (configurable: 300/500/1000 m)
    ↓ Confirmation timer management (60s/30s)
    ↓ Alert creation → WebSocket broadcast
    ↓ Frontend: separation status indicator on map
```

**Responsibility boundaries:**
- GPS collection → Frontend (Expo Location)
- Location transmission → Frontend/Backend WebSocket
- Separation calculation → Backend Monitoring Engine
- Alert delivery → Backend + Frontend
- Regrouping detection → Backend Monitoring Engine

## 12.6 Configuration

All thresholds are configurable via application settings (not hardcoded):

| Parameter | Default Value | Configurable |
|-----------|--------------|--------------|
| `normal_distance_m` | 300 | Yes |
| `monitoring_distance_m` | 500 | Yes |
| `separation_warning_distance_m` | 500 | Yes |
| `separation_warning_confirmation_s` | 60 | Yes |
| `high_separation_distance_m` | 1000 | Yes |
| `high_separation_confirmation_s` | 30 | Yes |
| `gps_accuracy_threshold_m` | 30 | Yes |
| `location_freshness_minutes` | 2 | Yes |
| `regrouping_distance_m` | 300 | Yes |
| `regrouping_confirmation_s` | 30 | Yes |

**Note:** These are initial configurable project values. The system shall allow future adjustments through configuration rather than hardcoding thresholds throughout the application.