# RydLink - System Architecture and UML Diagrams

This document contains all required diagrams for the RydLink technical documentation, created using Mermaid syntax. These diagrams complement the backend, frontend, integration, and testing documentation files.

## Table of Contents
1. [Backend Architecture Diagram](#1-backend-architecture-diagram)
2. [Frontend Architecture Diagram](#2-frontend-architecture-diagram)
3. [Component Diagram](#3-component-diagram)
4. [Backend Module Interaction Diagram](#4-backend-module-interaction-diagram)
5. [Frontend–Backend Integration Diagram](#5-frontend-backend-integration-diagram)
6. [REST API Communication Flow](#6-rest-api-communication-flow)
7. [WebSocket Communication Flow](#7-websocket-communication-flow)
8. [GPS Location Tracking Sequence Diagram](#8-gps-location-tracking-sequence-diagram)
9. [Rider Separation Detection Activity Diagram](#9-rider-separation-detection-activity-diagram)
10. [Intelligent Regrouping Activity Diagram](#10-intelligent-regrouping-activity-diagram)
11. [Emergency Reporting Sequence Diagram](#11-emergency-reporting-sequence-diagram)
12. [Push Notification Sequence Diagram](#12-push-notification-sequence-diagram)
13. [Complete Ride Lifecycle Activity Diagram](#13-complete-ride-lifecycle-activity-diagram)
14. [Testing Workflow Diagram](#14-testing-workflow-diagram)

---

## 1. Backend Architecture Diagram

```mermaid
graph TD
    %% Client Layer
    subgraph Client [Client Layer]
        RN[React Native Mobile App]
        Expo[Expo SDK]
        Location[Expo Location]
        WS[WebSocket Client]
    end

    %% API Gateway
    subgraph API [API Gateway Layer]
        Nginx[Nginx Reverse Proxy]
        Daphne[Daphne ASGI Server]
    end

    %% Django Backend
    subgraph Django [Django Backend Layer]
        DRF[Django REST Framework]
        Auth[Authentication App]
        Rides[Rides App]
        Participants[Participants App]
        Invitations[Invitations App]
        Monitoring[Monitoring App]
        Alerts[Alerts App]
        Routes[Routes App]
        Events[Events App]
        Emergencies[Emergencies App]
        Notifications[Notifications App]
        History[History App]
    end

    %% Channels Layer
    subgraph Channels [Real-time Layer]
        Channels[Django Channels]
        Consumers[WebSocket Consumers]
        Groups[Ride Groups: ride_{ride_id}]
    end

    %% Database Layer
    subgraph Database [Database Layer]
        PostgreSQL[(PostgreSQL 14/15)]
        PostGIS[PostGIS 3.x Extension]
        pgcrypto[pgcrypto Extension]
    end

    %% Cache Layer
    subgraph Cache [Cache Layer]
        Redis[(Redis 6/7)]
    end

    %% External Services
    subgraph External [External Services]
        OSRM[OSRM Routing Server]
        Overpass[Overpass API]
        Nominatim[Nominatim Geocoding]
        OSM[OpenStreetMap Tiles]
        FCM[Firebase Cloud Messaging]
        ExpoPush[Expo Notifications]
    end

    %% Connections
    RN -->|REST API| Nginx
    RN -->|WebSocket| Daphne
    Nginx -->|HTTP| DRF
    Daphne -->|ASGI| Channels
    DRF --> Auth
    DRF --> Rides
    DRF --> Participants
    DRF --> Invitations
    DRF --> Monitoring
    DRF --> Alerts
    DRF --> Routes
    DRF --> Events
    DRF --> Emergencies
    DRF --> Notifications
    DRF --> History
    Channels --> Consumers
    Consumers --> Groups
    Auth --> PostgreSQL
    Rides --> PostgreSQL
    Participants --> PostgreSQL
    Invitations --> PostgreSQL
    Monitoring --> PostgreSQL
    Alerts --> PostgreSQL
    Routes --> PostgreSQL
    Events --> PostgreSQL
    Emergencies --> PostgreSQL
    Notifications --> PostgreSQL
    History --> PostgreSQL
    Channels --> Redis
    Monitoring --> OSRM
    Monitoring --> Overpass
    Notifications --> FCM
    Notifications --> ExpoPush
    PostgreSQL -.-> PostGIS
    PostgreSQL -.-> pgcrypto

    %% Styling
    classDef client fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef gateway fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef backend fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef realtime fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef database fill:#ffebee,stroke:#b71c1c,stroke-width:2px
    classDef cache fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef external fill:#eceff1,stroke:#37474f,stroke-width:2px

    class RN,Expo,Location,WS client
    class Nginx,Daphne gateway
    class DRF,Auth,Rides,Participants,Invitations,Monitoring,Alerts,Routes,Events,Emergencies,Notifications,History backend
    class Channels,Consumers,Groups realtime
    class PostgreSQL,PostGIS,pgcrypto database
    class Redis cache
    class OSRM,Overpass,Nominatim,OSM,FCM,ExpoPush external
```

---

## 2. Frontend Architecture Diagram

```mermaid
graph TD
    %% Entry Point
    Entry[App.js / index.js]

    %% Navigation Layer
    subgraph Navigation [Navigation Layer]
        Root[RootNavigator]
        AuthNav[AuthNavigator]
        MainNav[MainNavigator]
    end

    %% State Management
    subgraph State [State Management]
        AuthCtx[AuthContext]
        RideCtx[RideContext]
        NotifCtx[NotificationContext]
        Store[Redux Store]
    end

    %% Services Layer
    subgraph Services [Services Layer]
        API[API Service - Axios]
        WS[WebSocket Service]
        NotifSvc[Notifications Service]
        LocationSvc[Location Service]
    end

    %% Screens Layer
    subgraph Screens [Screens Layer]
        %% Auth Screens
        Reg[Register Screen]
        Log[Login Screen]

        %% Main Screens
        RC[Ride Creation Screen]
        RL[Ride Lobby Screen]
        LM[Live Map Screen]
        US[User Search Screen]
        PF[Profile Screen]
        RH[Ride History Screen]
        EM[Emergency Screen]
        NF[Notification Screen]
    end

    %% Components Layer
    subgraph Components [Components Layer]
        %% Common
        BTN[Button]
        INP[Input]
        LOAD[LoadingSpinner]
        TOAST[Error/Success Toast]

        %% Map Components
        MapC[Map Component]
        PartMarker[Participant Marker]
        RoutePoly[Route Polyline]
        MarkerCluster[Marker Cluster]

        %% Tracking Components
        SepInd[Separation Indicator]
        StatusBadge[Status Badge]
        RegroupBanner[Regrouping Banner]

        %% Event Components
        EventCard[Event Card]
        EmergBanner[Emergency Banner]
    end

    %% Native Modules
    subgraph Native [Native Modules - Expo]
        ExpoLoc[Expo Location]
        ExpoTask[Expo Task Manager]
        ExpoNotif[Expo Notifications]
        AsyncStorage[AsyncStorage]
    end

    %% Connections
    Entry --> Root
    Root --> AuthNav
    Root --> MainNav
    AuthNav --> Reg
    AuthNav --> Log
    MainNav --> RC
    MainNav --> RL
    MainNav --> LM
    MainNav --> US
    MainNav --> PF
    MainNav --> RH
    MainNav --> EM
    MainNav --> NF

    Root --> AuthCtx
    Root --> RideCtx
    Root --> NotifCtx
    Root --> Store

    AuthCtx --> API
    RideCtx --> API
    RideCtx --> WS
    NotifCtx --> NotifSvc

    Reg --> API
    Log --> API
    RC --> API
    RL --> API
    RL --> WS
    LM --> API
    LM --> WS
    US --> API
    PF --> API
    RH --> API
    EM --> API
    EM --> WS
    NF --> API

    BTN --> MapC
    MapC --> PartMarker
    MapC --> RoutePoly
    MapC --> MarkerCluster

    RL --> SepInd
    RL --> StatusBadge
    LM --> RegroupBanner
    LM --> EventCard
    LM --> EmergBanner

    ExpoLoc --> LocationSvc
    ExpoTask --> LocationSvc
    ExpoNotif --> NotifSvc
    AsyncStorage --> AuthCtx

    %% Styling
    classDef entry fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef nav fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef state fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef svc fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef screen fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    classDef comp fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef native fill:#f1f8e9,stroke:#33691e,stroke-width:2px

    class Entry entry
    class Root,AuthNav,MainNav nav
    class AuthCtx,RideCtx,NotifCtx,Store state
    class API,WS,NotifSvc,LocationSvc svc
    class Reg,Log,RC,RL,LM,US,PF,RH,EM,NF screen
    class BTN,INP,LOAD,TOAST,MapC,PartMarker,RoutePoly,MarkerCluster,SepInd,StatusBadge,RegroupBanner,EventCard,EmergBanner comp
    class ExpoLoc,ExpoTask,ExpoNotif,AsyncStorage native
```

---

## 3. Component Diagram

```mermaid
graph TD
    %% Frontend Components
    subgraph Frontend [React Native Frontend]
        direction TB
        
        subgraph Screens [Screens]
            RegScreen[RegisterScreen]
            LoginScreen[LoginScreen]
            RideCreateScreen[RideCreationScreen]
            RideLobbyScreen[RideLobbyScreen]
            LiveMapScreen[LiveMapScreen]
            UserSearchScreen[UserSearchScreen]
            ProfileScreen[ProfileScreen]
            RideHistoryScreen[RideHistoryScreen]
            EmergencyScreen[EmergencyScreen]
            NotificationScreen[NotificationScreen]
        end

        subgraph Components [Components]
            MapComponent[MapComponent]
            ParticipantMarker[ParticipantMarker]
            RoutePolyline[RoutePolyline]
            SeparationIndicator[SeparationIndicator]
            StatusBadge[StatusBadge]
            RegroupingBanner[RegroupingBanner]
            EventCard[EventCard]
            EmergencyBanner[EmergencyBanner]
        end

        subgraph Services [Services]
            APIService[APIService Axios]
            WebSocketService[WebSocketService]
            NotificationService[NotificationService]
            LocationService[LocationService]
        end

        subgraph Contexts [Contexts]
            AuthContext[AuthContext]
            RideContext[RideContext]
            NotificationContext[NotificationContext]
        end
    end

    %% Backend Components
    subgraph Backend [Django Backend]
        direction TB
        
        subgraph API [REST API]
            AuthViews[Auth Views]
            RideViews[Ride Views]
            ParticipantViews[Participant Views]
            InvitationViews[Invitation Views]
            AlertViews[Alert Views]
            MeetingPointViews[Meeting Point Views]
            EventViews[Event Views]
            EmergencyViews[Emergency Views]
            DeviceTokenViews[Device Token Views]
        end

        subgraph Consumers [WebSocket Consumers]
            LocationConsumer[LocationConsumer]
            AlertConsumer[AlertConsumer]
            InvitationConsumer[InvitationConsumer]
        end

        subgraph Models [Database Models]
            User[User]
            Ride[Ride]
            RideParticipant[RideParticipant]
            RideInvitation[RideInvitation]
            LocationUpdate[LocationUpdate]
            RideRoute[RideRoute]
            RideAlert[RideAlert]
            MeetingPointRec[MeetingPointRecommendation]
            RideEvent[RideEvent]
            EmergencyReport[EmergencyReport]
            DeviceToken[DeviceToken]
            Notification[Notification]
        end

        subgraph Services [Business Services]
            SeparationEngine[Separation Detection Engine]
            RegroupingEngine[Regrouping Engine]
            OSRMService[OSRM Service]
            OverpassService[Overpass Service]
            PushNotificationService[Push Notification Service]
        end
    end

    %% External
    subgraph External [External Services]
        OSRM[(OSRM Server)]
        Overpass[(Overpass API)]
        PostgreSQL[(PostgreSQL + PostGIS)]
        Redis[(Redis)]
        FCM[(Firebase Cloud Messaging)]
        ExpoPush[(Expo Notifications)]
        OSM[(OpenStreetMap Tiles)]
    end

    %% Frontend to Backend connections
    RegScreen --> APIService
    LoginScreen --> APIService
    RideCreateScreen --> APIService
    RideLobbyScreen --> APIService
    RideLobbyScreen --> WebSocketService
    LiveMapScreen --> APIService
    LiveMapScreen --> WebSocketService
    UserSearchScreen --> APIService
    ProfileScreen --> APIService
    RideHistoryScreen --> APIService
    EmergencyScreen --> APIService
    EmergencyScreen --> WebSocketService
    NotificationScreen --> APIService

    APIService --> AuthViews
    APIService --> RideViews
    APIService --> ParticipantViews
    APIService --> InvitationViews
    APIService --> AlertViews
    APIService --> MeetingPointViews
    APIService --> EventViews
    APIService --> EmergencyViews
    APIService --> DeviceTokenViews

    WebSocketService --> LocationConsumer
    WebSocketService --> AlertConsumer
    WebSocketService --> InvitationConsumer

    %% Backend to Database
    AuthViews --> User
    RideViews --> Ride
    ParticipantViews --> RideParticipant
    InvitationViews --> RideInvitation
    LocationConsumer --> LocationUpdate
    RideViews --> RideRoute
    AlertViews --> RideAlert
    MeetingPointViews --> MeetingPointRec
    EventViews --> RideEvent
    EmergencyViews --> EmergencyReport
    DeviceTokenViews --> DeviceToken
    APIService --> Notification

    %% Backend to External
    AuthViews --> PostgreSQL
    RideViews --> PostgreSQL
    ParticipantViews --> PostgreSQL
    InvitationViews --> PostgreSQL
    LocationConsumer --> PostgreSQL
    RideViews --> RideRoute
    AlertViews --> RideAlert
    MeetingPointViews --> MeetingPointRec
    EventViews --> RideEvent
    EmergencyViews --> EmergencyReport
    DeviceTokenViews --> DeviceToken

    SeparationEngine --> PostgreSQL
    RegroupingEngine --> PostgreSQL
    OSRMService --> OSRM
    OverpassService --> Overpass
    PushNotificationService --> FCM
    PushNotificationService --> ExpoPush

    Channels[Django Channels] --> Redis
    LocationConsumer --> Channels
    AlertConsumer --> Channels
    InvitationConsumer --> Channels

    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef backend fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef external fill:#eceff1,stroke:#37474f,stroke-width:2px

    class RegScreen,LoginScreen,RideCreateScreen,RideLobbyScreen,LiveMapScreen,UserSearchScreen,ProfileScreen,RideHistoryScreen,EmergencyScreen,NotificationScreen frontend
    class MapComponent,ParticipantMarker,RoutePolyline,SeparationIndicator,StatusBadge,RegroupingBanner,EventCard,EmergencyBanner frontend
    class APIService,WebSocketService,NotificationService,LocationService frontend
    class AuthContext,RideContext,NotificationContext frontend
    class AuthViews,RideViews,ParticipantViews,InvitationViews,AlertViews,MeetingPointViews,EventViews,EmergencyViews,DeviceTokenViews backend
    class LocationConsumer,AlertConsumer,InvitationConsumer backend
    class User,Ride,RideParticipant,RideInvitation,LocationUpdate,RideRoute,RideAlert,MeetingPointRec,RideEvent,EmergencyReport,DeviceToken,Notification backend
    class SeparationEngine,RegroupingEngine,OSRMService,OverpassService,PushNotificationService backend
    class OSRM,Overpass,PostgreSQL,Redis,FCM,ExpoPush,OSM external
```

---

## 4. Backend Module Interaction Diagram

```mermaid
sequenceDiagram
    participant C as Client (React Native)
    participant API as Django REST Framework
    participant Auth as Auth App
    participant Ride as Rides App
    participant Part as Participants App
    participant Inv as Invitations App
    participant Mon as Monitoring App
    participant Alert as Alerts App
    participant Route as Routes App
    participant Event as Events App
    participant Emerg as Emergencies App
    participant Notif as Notifications App
    participant WS as Django Channels
    participant Redis as Redis Channel Layer
    participant DB as PostgreSQL + PostGIS
    participant OSRM as OSRM Server
    participant Overpass as Overpass API

    %% Registration flow
    C->>API: POST /api/auth/register/
    API->>Auth: Validate & create user
    Auth->>DB: INSERT user
    DB-->>Auth: User created
    Auth-->>API: User data
    API-->>C: 201 Created

    %% Login flow
    C->>API: POST /api/auth/login/
    API->>Auth: Validate credentials
    Auth->>DB: SELECT user
    DB-->>Auth: User record
    Auth-->>API: JWT tokens
    API-->>C: 200 OK + tokens

    %% Ride creation flow
    C->>API: POST /api/rides/
    API->>Ride: Create ride
    Ride->>DB: INSERT ride
    Ride->>Route: Calculate route via OSRM
    Route->>OSRM: Request route
    OSRM-->>Route: Route geometry + distance + duration
    Route->>DB: INSERT ride_route
    DB-->>Route: Route stored
    Route-->>Ride: Route data
    Ride-->>API: Ride created
    API-->>C: 201 Created + ride data

    %% Invitation flow
    C->>API: POST /api/rides/{id}/participants/
    API->>Inv: Create invitation
    Inv->>DB: INSERT ride_invitation
    DB-->>Inv: Invitation created
    Inv-->>API: Invitation data
    API-->>C: 201 Created

    %% WebSocket location flow
    C->>WS: Connect ?ride_id=&user_id=&token=
    WS->>Redis: GROUP_ADD ride_{ride_id}
    C->>WS: location_update {lat, lng, accuracy, timestamp}
    WS->>Mon: Process location update
    Mon->>DB: INSERT location_update
    DB-->>Mon: Location stored
    Mon->>Mon: Calculate group reference
    Mon->>Mon: Run separation detection
    alt Separation detected
        Mon->>Alert: Create alert
        Alert->>DB: INSERT ride_alert
        DB-->>Alert: Alert stored
        Alert->>WS: Broadcast separation_warning
    end
    WS->>Redis: GROUP_SEND ride_{ride_id} location_broadcast
    Redis-->>C: location_broadcast (all participants)

    %% Emergency flow
    C->>API: POST /api/emergencies/
    API->>Emerg: Create emergency report
    Emerg->>DB: INSERT emergency_report
    DB-->>Emerg: Emergency stored
    Emerg->>WS: Broadcast emergency
    WS->>Redis: GROUP_SEND ride_{ride_id} emergency
    Emerg->>Notif: Send push notification
    Notif->>FCM: Send push
    API-->>C: 201 Created

    %% Ride end flow
    C->>API: POST /api/rides/{id}/end/
    API->>Ride: End ride
    Ride->>DB: UPDATE ride status=completed
    DB-->>Ride: Ride updated
    Ride->>WS: Broadcast ride_ended
    API-->>C: 200 OK
```

---

## 5. Frontend–Backend Integration Diagram

```mermaid
graph LR
    %% Frontend
    subgraph FE [React Native Frontend]
        direction TB
        UI[UI Screens & Components]
        State[State Management - Redux/Context]
        API[API Service - Axios]
        WS[WebSocket Service]
        Notif[Expo Notifications]
        Loc[Expo Location]
    end

    %% Backend
    subgraph BE [Django Backend]
        direction TB
        DRF[Django REST Framework]
        Channels[Django Channels + Daphne]
        ORM[Django ORM]
        GIS[PostGIS Spatial Queries]
        MonEngine[Monitoring Engine]
        RegEngine[Regrouping Engine]
    end

    %% Database
    subgraph DB [(PostgreSQL + PostGIS)]
        direction TB
        Tables[12 Tables]
        Indexes[Spatial Indexes - GiST]
    end

    %% Cache
    subgraph Cache [(Redis)]
        ChannelLayer[Channel Layer]
    end

    %% External
    subgraph Ext [External Services]
        OSRM[OSRM Routing]
        Overpass[Overpass API]
        FCM[FCM / Expo Push]
    end

    %% Data flows
    UI <--> State
    State <--> API
    State <--> WS
    State <--> Notif
    State <--> Loc

    API <--> DRF
    WS <--> Channels
    Notif --> FCM
    Loc --> Loc

    DRF <--> ORM
    Channels <--> ChannelLayer
    ORM <--> Tables
    Tables <--> Indexes

    MonEngine <--> GIS
    RegEngine <--> GIS
    MonEngine --> OSRM
    RegEngine --> Overpass

    DB -.-> Tables
    Cache -.-> ChannelLayer

    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef backend fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef database fill:#ffebee,stroke:#b71c1c,stroke-width:2px
    classDef cache fill:#f1f8e9,stroke:#33691e,stroke-width:2px
    classDef external fill:#eceff1,stroke:#37474f,stroke-width:2px

    class UI,State,API,WS,Notif,Loc frontend
    class DRF,Channels,ORM,GIS,MonEngine,RegEngine backend
    class Tables,Indexes database
    class ChannelLayer cache
    class OSRM,Overpass,FCM external
```

---

## 6. REST API Communication Flow

```mermaid
sequenceDiagram
    participant App as React Native App
    participant Axios as Axios HTTP Client
    participant API as Django REST API
    participant Auth as JWT Authentication
    participant ORM as Django ORM
    participant DB as PostgreSQL

    %% Successful API call
    App->>Axios: Request (method, endpoint, body, headers)
    Axios->>API: HTTP Request
    
    alt Valid JWT Token
        API->>Auth: Validate token
        Auth-->>API: Token valid, user ID
        
        API->>ORM: Query database
        ORM->>DB: SQL query
        DB-->>ORM: Query result
        ORM-->>API: Model instances
        
        API-->>Axios: 200 OK + JSON data
        Axios-->>App: Response data
        
    else Invalid/Expired Token
        Auth-->>API: Token invalid
        API-->>Axios: 401 Unauthorized
        Axios-->>App: Error response
        
        App->>Axios: Refresh token request
        Axios->>API: POST /api/auth/refresh/
        API->>Auth: Validate refresh token
        Auth-->>API: New access token
        API-->>Axios: 200 OK + new token
        Axios-->>App: New token stored
        
        App->>Axios: Retry original request with new token
        Axios->>API: HTTP Request (retry)
        API-->>Axios: 200 OK + JSON data
        Axios-->>App: Response data
    end

    %% Error case
    App->>Axios: Request with invalid data
    Axios->>API: HTTP Request
    API->>Auth: Validate token
    Auth-->>API: Token valid
    API->>ORM: Validate & save
    ORM->>DB: Constraint violation
    DB-->>ORM: Error
    ORM-->>API: Validation error
    API-->>Axios: 400 Bad Request + ERR_001
    Axios-->>App: Error with field details
```

---

## 7. WebSocket Communication Flow

```mermaid
sequenceDiagram
    participant Client as React Native Client
    participant WS as WebSocket Connection
    participant Consumer as Django Channels Consumer
    participant Redis as Redis Channel Layer
    participant Group as Ride Group ride_{ride_id}
    participant DB as PostgreSQL

    %% Connection establishment
    Client->>WS: new WebSocket(url?ride_id=&user_id=&token=)
    WS->>Consumer: Connect with query params
    Consumer->>DB: Verify JWT token & ride participation
    DB-->>Consumer: Valid participant
    Consumer->>Redis: GROUP_ADD ride_{ride_id}
    Redis-->>Consumer: Group joined
    Consumer-->>WS: Connection accepted
    WS-->>Client: onopen event

    %% Location update flow
    Client->>WS: send(location_update)
    WS->>Consumer: location_update message
    Consumer->>DB: Validate & store location
    DB-->>Consumer: Location stored
    Consumer->>Redis: GROUP_SEND ride_{ride_id} location_broadcast
    Redis->>Group: Broadcast to all members
    Group-->>Consumer: Message delivered
    Consumer-->>WS: Broadcast to all clients
    WS-->>Client: onmessage (location_broadcast)

    %% Separation detection flow
    Consumer->>Consumer: Run separation detection (every 30s)
    Consumer->>DB: Get latest locations
    DB-->>Consumer: Location data
    Consumer->>Consumer: Calculate distances & thresholds
    alt Alert threshold exceeded
        Consumer->>DB: INSERT ride_alert
        DB-->>Consumer: Alert stored
        Consumer->>Redis: GROUP_SEND ride_{ride_id} separation_warning/high_separation
        Redis->>Group: Broadcast alert
        Group-->>Consumer: Alert delivered
        Consumer-->>WS: Alert to leader + affected rider
        WS-->>Client: onmessage (separation_warning/high_separation)
    end

    %% Disconnection
    Client->>WS: close()
    WS->>Consumer: Disconnect
    Consumer->>Redis: GROUP_DISCARD ride_{ride_id}
    Redis-->>Consumer: Group left
    Consumer->>DB: UPDATE participant status=left
    DB-->>Consumer: Participant updated
    Consumer-->>WS: Connection closed
    WS-->>Client: onclose event
```

---

## 8. GPS Location Tracking Sequence Diagram

```mermaid
sequenceDiagram
    participant Device as Mobile Device
    participant ExpoLoc as Expo Location
    participant LocationSvc as Location Service
    participant WS as WebSocket Service
    participant Consumer as Django Channels Consumer
    participant DB as PostgreSQL + PostGIS
    participant Mon as Monitoring Engine

    %% Foreground location tracking
    Device->>ExpoLoc: requestForegroundPermissionsAsync()
    ExpoLoc-->>Device: Permission granted
    
    loop Every 5-10 seconds
        Device->>ExpoLoc: getCurrentPositionAsync()
        ExpoLoc-->>Device: GPS coordinates {lat, lng, accuracy, speed, heading, timestamp}
        
        Device->>LocationSvc: Validate GPS (accuracy ≤ 30m, freshness < 2min)
        
        alt Valid GPS
            LocationSvc->>WS: send(location_update)
            WS->>Consumer: WebSocket message
            Consumer->>DB: INSERT location_update (PostGIS geography)
            DB-->>Consumer: Stored
            Consumer->>Mon: Trigger separation detection
            Mon->>DB: Query group reference
            DB-->>Mon: Group centroid
            Mon-->>Consumer: Alert if needed
            Consumer-->>WS: Broadcast location_broadcast
            WS-->>Device: onmessage (location_broadcast)
        else Invalid GPS
            LocationSvc-->>Device: Use last valid location
        end
    end

    %% Background location tracking
    Device->>ExpoLoc: requestBackgroundPermissionsAsync()
    ExpoLoc-->>Device: Background permission granted
    
    Device->>ExpoLoc: startBackgroundLocationUpdateAsync()
    
    loop Every 30-60 seconds (background)
        ExpoLoc-->>Device: Background location callback
        Device->>LocationSvc: Validate & queue
        LocationSvc->>WS: send(location_update) (if online)
        LocationSvc-->>Device: Store locally (if offline)
    end

    %% App minimized
    Device->>Device: App minimized
    Device->>ExpoLoc: Continue background updates
    ExpoLoc-->>Device: Location updates continue (reduced frequency)
```

---

## 9. Rider Separation Detection Activity Diagram

```mermaid
activityDiagram
    start
    
    note right: Separation Detection Engine<br/>Runs every 30 seconds
    
    :Get all active participants in ride;
    :Get latest location for each participant;
    
    for each participant
        :Validate GPS accuracy ≤ 30m;
        
        if accuracy > 30m then
            :Skip participant, use last valid location;
        else
            :Validate location freshness < 2 minutes;
            
            if location stale then
                :Skip participant in evaluation;
            else
                :Calculate distance from group reference position;
                
                if distance ≤ 300m then
                    :Status = normal;
                    :Reset timer;
                    if previous status != normal then
                        :Generate regrouped alert;
                    end if
                elseif distance ≤ 500m then
                    :Status = monitoring;
                    :Start/pause timer;
                elseif distance ≤ 1000m then
                    :Status = separation_warning;
                    :Start 60-second timer;
                    
                    if timer ≥ 60s then
                        :Generate separation_warning alert;
                        :Reset timer to prevent duplicates;
                    end if
                else
                    :Status = high_separation;
                    :Start 30-second timer;
                    
                    if timer ≥ 30s then
                        :Generate high_separation alert;
                        :Reset timer to prevent duplicates;
                    end if
                end if
                
                :Update participant separation status in DB;
            end if
        end if
    end for
    
    :Broadcast alerts via WebSocket;
    :Broadcast location_broadcast to all participants;
    stop
```

---

## 10. Intelligent Regrouping Activity Diagram

```mermaid
activityDiagram
    start
    
    note left: Intelligent Regrouping Algorithm<br/>Rule-based, no ML
    
    :Separation warning or high separation triggered;
    :Rider returns within 300m OR Leader initiates;
    
    :Calculate group reference position (centroid);
    
    :Discover candidate POIs via Overpass API;
    :Filter by type: fuel, restaurant, parking;
    :Search radius: 5 km from centroid;
    
    for each candidate POI
        for each participant
            :Calculate OSRM route travel time;
            
            if no valid route then
                :Skip candidate for this participant;
            end if
        end for
        
        :Calculate worst-case ETA (max across participants);
        :Calculate average ETA;
        :Apply suitability filter (ETA ≤ 30 min);
        
        if suitable then
            :Add to ranked candidates;
        end if
    end for
    
    :Sort candidates by worst-case ETA (ascending);
    :Sort by average ETA (secondary);
    
    :Present top 3 recommendations to Ride Leader;
    
    :Leader approves or rejects;
    
    if approved then
        :Update recommendation status = approved;
        :Broadcast meeting_point_approved via WebSocket;
        :Show meeting point on map to all participants;
        :Update ETA for each participant;
        
        :Participant navigates to meeting point;
        
        :Upon arrival within 300m for 30s;
        :Auto-resolve separation alert;
        :Broadcast regrouped notification;
    else
        :Recommendation rejected;
        :Leader may request more options;
    end if
    
    stop
```

---

## 11. Emergency Reporting Sequence Diagram

```mermaid
sequenceDiagram
    participant Rider as Rider (Mobile App)
    participant API as Django REST API
    participant Emerg as Emergencies App
    participant DB as PostgreSQL
    participant WS as Django Channels
    participant Redis as Redis Channel Layer
    participant Group as Ride Group
    participant Notif as Push Notification Service
    participant FCM as FCM / Expo Push
    participant Leader as Ride Leader (Mobile App)
    participant Other as Other Participants

    %% Emergency report
    Rider->>API: POST /api/emergencies/
    API->>Emerg: Create emergency report
    Emerg->>DB: INSERT emergency_report (status=active)
    DB-->>Emerg: Emergency stored
    Emerg-->>API: Emergency created
    API-->>Rider: 201 Created + emergency data

    %% WebSocket broadcast
    Emerg->>WS: Broadcast emergency (high priority)
    WS->>Redis: GROUP_SEND ride_{ride_id} emergency
    Redis->>Group: Broadcast to all members
    
    Group-->>Leader: onmessage (emergency)
    Group-->>Other: onmessage (emergency)
    
    Leader-->>Leader: Show full-screen emergency banner
    Other-->>Other: Show emergency banner

    %% Push notification
    Emerg->>Notif: Send high-priority push notification
    Notif->>FCM: Send to all device tokens
    FCM-->>Leader: Push notification delivered
    FCM-->>Other: Push notification delivered

    %% Acknowledgment
    Leader->>API: PUT /api/emergencies/{id}/acknowledge/
    API->>Emerg: Update acknowledged_at
    Emerg->>DB: UPDATE emergency_report
    DB-->>Emerg: Updated
    Emerg-->>API: Acknowledged
    API-->>Leader: 200 OK

    Emerg->>WS: Broadcast emergency_acknowledged
    WS->>Redis: GROUP_SEND ride_{ride_id} emergency_acknowledged
    Redis->>Group: Broadcast acknowledgment

    %% Resolution
    Rider->>API: PUT /api/emergencies/{id}/resolve/
    API->>Emerg: Update resolved_at, status=resolved
    Emerg->>DB: UPDATE emergency_report
    DB-->>Emerg: Resolved
    Emerg-->>API: Resolved
    API-->>Rider: 200 OK

    Emerg->>WS: Broadcast emergency_resolved
    WS->>Redis: GROUP_SEND ride_{ride_id} emergency_resolved
    Redis->>Group: Broadcast resolution

    Leader-->>Leader: Emergency banner removed
    Other-->>Other: Emergency banner removed
```

---

## 12. Push Notification Sequence Diagram

```mermaid
sequenceDiagram
    participant App as React Native App
    participant ExpoNotif as Expo Notifications
    participant API as Django REST API
    participant DB as PostgreSQL
    participant NotifSvc as Push Notification Service
    participant Expo as Expo Push Service
    participant FCM as Firebase Cloud Messaging
    participant Device as Mobile Device (OS)

    %% Token registration
    App->>ExpoNotif: requestPermissionsAsync()
    ExpoNotif-->>App: Permission granted
    
    App->>ExpoNotif: getExpoPushTokenAsync()
    ExpoNotif-->>App: ExpoPushToken[xxx]
    
    App->>API: POST /api/device-token/ {token, platform}
    API->>DB: INSERT device_token
    DB-->>API: Token stored
    API-->>App: 200/201 Created

    %% Notification trigger (e.g., separation alert)
    NotifSvc->>DB: SELECT device_tokens WHERE user_id = ?
    DB-->>NotifSvc: Device tokens
    
    NotifSvc->>Expo: send_push_notifications([{to, title, body, data, category}])
    
    alt iOS Device
        Expo->>Device: APNs push notification
    else Android Device
        Expo->>FCM: Forward to FCM
        FCM->>Device: FCM push notification
    end
    
    Device-->>App: Notification received
    
    %% Foreground handling
    alt App in foreground
        App->>App: Notifications.presentNotificationAsync()
        App-->>User: Show in-app notification banner
    else App in background/closed
        Device-->>User: System notification banner
        User->>Device: Tap notification
        Device->>App: Open app to related screen
    end

    %% Delivery status tracking
    NotifSvc->>DB: UPDATE notification delivery status
    DB-->>NotifSvc: Status updated
```

---

## 13. Complete Ride Lifecycle Activity Diagram

```mermaid
activityDiagram
    start
    
    note top: Complete Ride Lifecycle<br/>From creation to history
    
    :User registers and logs in;
    
    :Ride Leader creates ride (start, dest, departure time);
    :Ride status = planned;
    :Route calculated via OSRM;
    
    :Leader invites participants;
    :Invitation status = pending (24h expiry);
    
    :Participants accept/reject invitations;
    :Accepted: participant added to ride;
    :Rejected: invitation closed;
    
    :Ride lobby - participants join;
    :Participant status = joined;
    
    :Leader starts ride;
    :Ride status = active;
    :WebSocket connection established;
    :Location tracking starts (foreground/background);
    
    loop During active ride
        :Participants share GPS locations;
        :Location broadcast to all participants;
        :Separation detection every 30s;
        
        if separation detected then
            :Generate separation warning/high separation alert;
            :Notify leader + affected rider;
        end if
        
        if regrouping needed then
            :Generate meeting point recommendations;
            :Leader approves meeting point;
            :Share with all participants;
        end if
        
        if ride event created then
            :Broadcast ride event to all participants;
        end if
        
        if emergency reported then
            :Broadcast emergency (high priority);
            :Send push notification;
            :Acknowledge and resolve;
        end if
    end loop
    
    :Leader ends ride;
    :Ride status = completed;
    :WebSocket connections closed;
    :Location tracking stopped;
    
    :Archive ride to history;
    :Ride data retained per policy (90 days for locations);
    
    :Participants can view ride history;
    stop
```

---

## 14. Testing Workflow Diagram

```mermaid
flowchart TD
    %% Start
    Start([Start Testing])
    
    %% Test Levels
    subgraph Unit [Unit Testing]
        U1[Backend Unit Tests<br/>pytest]
        U2[Frontend Unit Tests<br/>Jest]
        U3[Coverage Report<br/>≥80% backend, ≥70% frontend]
    end
    
    subgraph Integration [Integration Testing]
        I1[Frontend-Backend API]
        I2[Frontend-Backend WebSocket]
        I3[Django-PostgreSQL]
        I4[Django Channels-Redis]
        I5[Backend-OSRM]
        I6[Backend-Overpass]
        I7[Backend-Push Notifications]
    end
    
    subgraph System [System Testing]
        S1[End-to-End Ride Lifecycle]
        S2[Registration to History]
        S3[Separation Detection E2E]
        S4[Emergency Reporting E2E]
    end
    
    subgraph Performance [Performance Testing]
        P1[Concurrent Participants<br/>50 max]
        P2[GPS Update Load]
        P3[WebSocket Message Load]
        P4[API Response Time<br/><800ms]
        P5[Battery Consumption<br/><15%/hour]
    end
    
    subgraph Security [Security Testing]
        Sec1[Authentication & Authorization]
        Sec2[JWT Validation]
        Sec3[API Access Control]
        Sec4[WebSocket Security]
        Sec5[Input Validation]
    end
    
    subgraph UAT [User Acceptance Testing]
        UAT1[Ride Leader Scenarios]
        UAT2[Ride Participant Scenarios]
        UAT3[End-to-End User Flows]
    end
    
    %% Gates
    Gate1{All Unit Tests Pass?}
    Gate2{All Integration Tests Pass?}
    Gate3{All System Tests Pass?}
    Gate4{Performance Benchmarks Met?}
    Gate5{No Critical Security Issues?}
    Gate6[UAT Sign-off]
    
    %% Flow
    Start --> U1
    Start --> U2
    U1 --> U3
    U2 --> U3
    U3 --> Gate1
    
    Gate1 -->|Yes| I1
    Gate1 -->|No| Fix1[Fix Bugs]
    Fix1 --> U1
    
    I1 --> I2
    I2 --> I3
    I3 --> I4
    I4 --> I5
    I5 --> I6
    I6 --> I7
    I7 --> Gate2
    
    Gate2 -->|Yes| S1
    Gate2 -->|No| Fix2[Fix Integration Issues]
    Fix2 --> I1
    
    S1 --> S2
    S2 --> S3
    S3 --> S4
    S4 --> Gate3
    
    Gate3 -->|Yes| P1
    Gate3 -->|No| Fix3[Fix System Issues]
    Fix3 --> S1
    
    P1 --> P2
    P2 --> P3
    P3 --> P4
    P4 --> P5
    P5 --> Gate4
    
    Gate4 -->|Yes| Sec1
    Gate4 -->|No| Fix4[Fix Performance Issues]
    Fix4 --> P1
    
    Sec1 --> Sec2
    Sec2 --> Sec3
    Sec3 --> Sec4
    Sec4 --> Sec5
    Sec5 --> Gate5
    
    Gate5 -->|Yes| UAT1
    Gate5 -->|No| Fix5[Fix Security Issues]
    Fix5 --> Sec1
    
    UAT1 --> UAT2
    UAT2 --> UAT3
    UAT3 --> Gate6
    
    Gate6 -->|Yes| Release[Release to Production]
    Gate6 -->|No| Fix6[Fix UAT Issues]
    Fix6 --> UAT1
    
    Release --> End([End - Release Ready])
    
    %% Styling
    classDef startEnd fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef test fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef gate fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef fix fill:#ffebee,stroke:#b71c1c,stroke-width:2px
    classDef release fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class Start,End startEnd
    class U1,U2,U3,I1,I2,I3,I4,I5,I6,I7,S1,S2,S3,S4,P1,P2,P3,P4,P5,Sec1,Sec2,Sec3,Sec4,Sec5,UAT1,UAT2,UAT3 test
    class Gate1,Gate2,Gate3,Gate4,Gate5 gate
    class Fix1,Fix2,Fix3,Fix4,Fix5,Fix6 fix
    class Release,Gate6 release
```

---