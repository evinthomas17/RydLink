# Section 17: Development and Deployment Plan

## 17.1 Development Environment Setup

| Component | Install | Command/Notes |
|-----------|---------|---------------|
| Node.js | 20/22 LTS | nvm or installer |
| npm | Bundled | — |
| Python | 3.11 or 3.12 | python.org |
| pip | Bundled | — |
| PostgreSQL | 14/15 | + PostGIS extension |
| Redis | 6/7 | — |
| Expo CLI | `npm i -g expo-cli` | — |
| Android Studio | Latest | For Android emulator |
| Git | Latest | — |

## 17.2 Backend Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install django djangorestframework djangorestframework-simplejwt \
    channels channels-redis daphne psycopg requests

# Create project
django-admin startproject config .
python manage.py startapp rides

# Enable PostGIS
# In PostgreSQL: CREATE EXTENSION postgis;
```

## 17.3 Frontend Setup

```bash
npx create-expo-app rydlink-mobile
cd rydlink-mobile
npx expo install expo-location expo-task-manager expo-notifications \
    @react-navigation/native @maplibre/react-native-maplibre-gl axios
```

## 17.4 Database Setup

- Install PostgreSQL
- Enable PostGIS: `CREATE EXTENSION postgis;`
- Create database: `createdb rydlink`
- Configure `DATABASE_URL` in `.env`
- Run migrations: `python manage.py migrate`

## 17.5 Redis Setup

- Install Redis: `redis-server`
- Configure `REDIS_URL` in `.env`
- Django Channels channel layer: `channels_redis.core.RedisChannelLayer`

## 17.6 OSRM Setup

**Development:** Use public demo server (rate-limited) or self-host  
**Production:** Self-host

```bash
# Download road network .pbf (e.g., from Geofabrik)
wget https://download.geofabrik.de/north-america/us/new-york-latest.osm.pbf

# Build OSRM
osrm-extract -p car.lua new-york-latest.osm.pbf
osrm-partition new-york-latest.osrm
osrm-customize new-york-latest.osrm
osrm-routed --algorithm mld new-york-latest.osrm
```

## 17.7 Project Folder Structure

```
rydlink/
├── backend/
│   ├── config/           # Django settings, urls, ASGI, WSGI
│   ├── rides/            # Main app (models, views, consumers, services)
│   ├── users/            # Custom user model, auth
│   ├── requirements.txt
│   ├── manage.py
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── navigation/
│   │   ├── services/     # API, WebSocket, location, notifications
│   │   └── context/
│   │   App.js
│   ├── package.json
│   └── app.json          # Expo config
├── docs/                 # This documentation
├── docker-compose.yml    # Optional (if Docker installed later)
└── README.md
```

## 17.8 Git Workflow

- `main` branch: production-ready
- `develop` branch: integration
- Feature branches: `feature/module-X`
- Pull requests with review before merge
- Conventional commits: `feat:`, `fix:`, `docs:`

## 17.9 Environment Variables

```
# Backend (.env)
SECRET_KEY=...
DEBUG=False
ALLOWED_HOSTS=...
DATABASE_URL=postgres://user:pass@localhost:5432/rydlink
REDIS_URL=redis://localhost:6379/0
JWT_ACCESS_TOKEN_LIFETIME=900
JWT_REFRESH_TOKEN_LIFETIME=604800
OSRM_BASE_URL=http://localhost:5000
OVERPASS_BASE_URL=https://overpass-api.de/api/interpreter

# Frontend (.env)
API_BASE_URL=https://api.rydlink.com/api/v1
WS_BASE_URL=wss://api.rydlink.com/ws
EXPO_PUBLIC_MAP_TOKEN=...
```

## 17.10 Development Phases

| Phase | Modules | Dependencies | Duration |
|-------|---------|--------------|----------|
| 1 | Environment setup | — | 1 week |
| 2 | Authentication & profile | Phase 1 | 1 week |
| 3 | Ride creation & group | Phase 2 | 1 week |
| 4 | Invitation management | Phase 3 | 1 week |
| 5 | Ride lobby & participants | Phase 3, 4 | 1 week |
| 6 | Live GPS tracking | Phase 5 | 2 weeks |
| 7 | Map visualization | Phase 6 | 1 week |
| 8 | Route management | Phase 2 | 1 week |
| 9 | Intelligent monitoring | Phase 6, 7 | 2 weeks |
| 10 | Meeting point recommendations | Phase 9 | 1 week |
| 11 | Ride event management | Phase 6 | 1 week |
| 12 | Emergency management | Phase 6, 11 | 1 week |
| 13 | Push notifications | Phase 1 | 1 week |
| 14 | Network connectivity | Phase 6 | 1 week |
| 15 | Ride completion & history | Phase 5–12 | 1 week |
| 16 | Testing & deployment | All | 2 weeks |

## 17.11 Integration Process

- Each phase integrates with previous via REST API contracts
- WebSocket events integrated once Module 5 (tracking) is stable
- Backend frontend integration tested with Postman + manual frontend testing

## 17.12 Testing Process

- Unit tests after each module
- API testing with Postman collections
- Integration testing after phase completion
- UAT with beta group before deployment

## 17.13 Deployment Architecture

```
Production:
[Mobile Clients] → [HTTPS LB] → [Daphne Workers] → [Redis] → [PostgreSQL + PostGIS]
                                                ↳ [OSRM Container]
                                                ↳ [Overpass Container]
                                                ↳ [FCM/Expo Push]
```
