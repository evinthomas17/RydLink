# Section 15: Error Handling and Special Scenarios

## 15.1 Scenario Analysis

| Scenario | Expected Behavior | Recovery Mechanism |
|----------|-------------------|---------------------|
| GPS unavailable | Show last known location + "GPS unavailable" warning | Retry periodically; use cached location |
| GPS permission denied | Prompt for permission; disable tracking; show manual entry option | Settings redirect; re-prompt on app resume |
| Internet disconnected | Offline indicator; cache locations locally; WebSocket reconnect | Auto-reconnect with exponential backoff; resync on restore |
| WebSocket disconnected | Show "reconnecting"; stop broadcasting; retain last locations | Reconnect, re-authenticate, resubscribe, send buffered updates |
| Stale location (>2 min) | Marker turns gray; show "last updated X min ago" | Refresh on new update; offline after 5 min |
| Duplicate invitation | 409 error; existing pending invitation returned | User can respond to existing invitation |
| Invalid invitation | 404/410 error; invitation not found or expired | User can request new invitation from leader |
| Expired invitation (24h) | Status set to 'expired'; cannot respond | Leader can re-invite |
| User leaves ride | `status='left'`, `is_active=false`; broadcast `participant.left` | User can re-join if re-invited |
| Ride Leader disconnects | Ride continues; other participants still see leader's last location | Leader reconnects; if prolonged, participants notified |
| Ride Leader leaves permanently | Leader can transfer role or end ride; if leader leaves without transfer, ride remains active until ended | Next participant can be promoted by remaining leader action or ride auto-ends |
| OSRM unavailable | Show direct line between start/dest; error toast | Retry; fallback to haversine straight-line distance |
| Overpass API unavailable | Meeting point recommendation fails; leader can manually enter coordinates | Error message; manual entry fallback |
| Incorrect GPS coordinates | Reject if out of range or accuracy > 50m; log and ignore | Use last valid location |
| Battery restrictions (Android) | OS may kill background task; tracking pauses | Foreground service notification; battery optimization exclusion request |
| Ride cancellation | `status='cancelled'`; broadcast `ride.ended`; cleanup WebSocket groups | Participants notified; ride moved to history |
| Emergency during network disconnect | Event cached locally; sent on reconnect with original timestamp | Sync queue; high-priority delivery on restore |

## 15.2 Ride Leader Transfer Mechanism

- **Temporary disconnect:** Ride remains active; leader's last location shown as stale; no transfer needed
- **Leader closes app:** Same as temporary disconnect; if reconnects within 5 min, resumes
- **Leader leaves ride:** Must transfer leadership to another participant before leaving, or end the ride
- **Leader device unavailable > 5 min:** System alerts participants that leader is unreachable
- **Leader wants to transfer control:** Leader selects new leader from participants; backend updates `rides.leader_id` and `ride_participants.role`; broadcast `leader.transferred`

**Authorization:** Only current leader can initiate transfer. New leader must be active participant.
