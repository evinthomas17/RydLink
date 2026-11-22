# Section 18: Future Enhancements and Limitations

## 18.1 Future Enhancements

| Enhancement | Description | Justification |
|-------------|-------------|---------------|
| Traffic-aware routing | OSRM with live traffic data for accurate ETA | Improves regrouping ETA accuracy |
| WhatsApp/SMS integration | Share ride status via messaging apps | User convenience (currently out of scope) |
| Vehicle telemetry (OBD-II) | Connect to car diagnostics for breakdown data | More accurate emergency detection |
| Machine learning | Predict separation before it occurs | Requires sufficient training data (not for MVP) |
| iOS support | Full iOS background tracking | Currently Android-first |
| Wearable integration | Smartwatch alerts for separation | Safety convenience |
| Multi-day rides | Support overnight stops and multi-day itineraries | Extended trip use cases |
| Ride templates | Save and reuse common ride configurations | User convenience |

## 18.2 Machine Learning Considerations

ML is **not** introduced in the initial version because:
- No historical training data exists for a new application
- Rule-based algorithms are sufficient and explainable
- ML adds complexity without clear MVP benefit

Future ML justification would require:
- 6+ months of ride data
- Demonstrated limitations of rule-based thresholds
- Clear safety improvement metric

## 18.3 Current Limitations

- Not a replacement for dedicated navigation apps (no turn-by-turn)
- Background tracking requires Expo development build (not Expo Go)
- Public OSRM/Overpass/Nominatim have rate limits; production requires self-hosting
- GPS accuracy depends on device hardware and environment
- Push notifications require EAS build + FCM for production
- Python 3.11/3.12 targeted (Django compatibility); Python 3.15 not guaranteed
- Android-first; iOS background tracking more restrictive
