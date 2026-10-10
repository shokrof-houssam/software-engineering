
# Campus-companion Tech Stack

## Overview

A mobile app (iOS & Android) for our university campus featuring a custom interactive 3D campus map, event management, clubs, navigation, and more. The 3D rendering happens client-side on each device — the backend serves data, not graphics.

---

## Architecture

```
React Native App
│
├── Home / Events / Clubs / Profile / Notifications
│
└── Campus Map
       └── WebView
              └── React + Three.js
                     ├── Terrain
                     ├── Buildings
                     ├── Roads / Walkways
                     ├── Trees / Environment
                     ├── Labels
                     └── POIs
```

```
Mobile Users                         CDN / S3
     │                                  │
     ▼                                  ├── campus.glb
Spring Boot API                        ├── textures
     │                                  ├── images
     ▼                                  └── other assets
PostgreSQL
```

---

## Mobile App

| Technology | Purpose |
|---|---|
| React Native | Cross-platform iOS & Android from one codebase |
| TypeScript | Type safety across the app |
| Expo | Faster dev/build/deployment workflow |
| Zustand | Lightweight state management |
| TanStack Query | Server state, caching, data fetching |

---

## 3D Campus Map

| Technology | Purpose |
|---|---|
| Three.js | 3D rendering engine for the campus scene |
| React Three Fiber | React bindings for Three.js |
| Blender | Modeling the campus buildings and environment |
| glTF / GLB | 3D asset format (web/mobile optimized) |
| Draco / Meshopt | Mesh compression for smaller downloads |

The campus map is a **self-contained 3D scene** (not a geographic map with buildings on top). It runs inside a WebView as a React + Three.js web app embedded in React Native.

### 3D Asset Structure

```
campus/
├── terrain.glb
├── buildings/
│   ├── computing.glb
│   ├── library.glb
│   ├── restaurant.glb
│   ├── sports.glb
│   └── ...
├── environment/
│   ├── trees.glb
│   ├── lamps.glb
│   └── benches.glb
└── campus.json          ← semantic metadata (names, positions, rotations)
```

### 3D Asset Pipeline

```
Campus plans / photos → Blender → Low-poly models → GLB/glTF → Draco compression → S3/CDN → Three.js
```

> Stylized low-poly models (think Google Maps 3D / Apple Maps style), not photorealistic renders. Lighter, faster, looks better on mobile.

---

## Backend

| Technology | Purpose |
|---|---|
| Java 21+ | Backend language |
| Spring Boot | Application framework |
| Spring Web | REST API layer |
| Spring Security + JWT/OAuth2 | Authentication and authorization |
| Spring Data JPA | Database access via Hibernate |
| Spring WebSocket | Real-time features (live events, locations, chat) |
| Flyway | Database migration management |

### Project Structure (Modular Monolith)

```
backend/
├── auth/
├── users/
├── campus/
├── buildings/
├── rooms/
├── navigation/
├── events/
├── clubs/
├── notifications/
└── admin/
```

> Start as a modular monolith, not microservices.

---

## Database

| Technology | Purpose |
|---|---|
| PostgreSQL | Primary relational database |

### Core Schema

```
Building: id, name, code, description, coordinates, model_url, thumbnail, number_of_floors
Room: id, building_id, name, floor, type, coordinates
CampusPOI: id, name, category, coordinates
CampusPath: id, start, end, geometry, accessible, indoor, weight
```

### Navigation

Campus paths stored as a graph (nodes + weighted edges). A* or Dijkstra on the backend for routing.

```
GET /api/navigation/route?from=computing&to=restaurant

{
  "distance": 430,
  "duration": 320,
  "path": [...]
}
```

Each edge stores: distance, stairs, wheelchair_accessible, covered, indoor/outdoor.

---

## Infrastructure

| Technology | Purpose |
|---|---|
| Docker | Containerized deployment |
| S3-compatible storage | GLB models, images, assets (served via CDN) |
| Nginx / Caddy | Reverse proxy |
| GitHub Actions | CI/CD pipelines |

---

## Admin Dashboard

| Technology | Purpose |
|---|---|
| Next.js | Admin web app framework |
| React + TypeScript | UI |

Manages buildings, events, clubs, rooms, POIs, announcements, opening hours — all through the same Spring Boot API.

```
                    ┌── React Native iOS
Spring Boot API ────┼── React Native Android
                    └── Next.js Admin Dashboard
```

---

## Add Later (When Needed)

| Technology | Purpose |
|---|---|
| Redis | Caching, sessions, real-time state |
| Spring WebSocket + Redis | Live friend locations, shuttle tracking, chat |
| PostGIS | Geographic queries (nearest building, POIs within radius) |
| GPS → local 3D coords | Show user's real position on the 3D map |
| Kafka | Event streaming (if scale demands it) |
| Elasticsearch | Full-text search across campus data |

---

## Scaling Notes

- 3D rendering is **client-side** — 1,000 users viewing the map = 0 extra server load for rendering
- Static 3D assets served from CDN, cached on device — not through Spring Boot
- Spring Boot is stateless (JWT auth) — horizontally scalable behind a load balancer
- HikariCP connection pooling (Spring Boot default)
- A single Spring Boot instance can comfortably serve a university-scale app
- Scale horizontally only when actually needed
