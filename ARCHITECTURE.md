# Architecture

## System Overview (v0.1)

```
┌─────────────────────────────────────────────────────────┐
│                     USER'S BROWSER                       │
│                                                          │
│  ┌──────────────┐    ┌──────────────┐   ┌─────────────┐ │
│  │  Home Page    │───▶│ Results View  │   │  /list      │ │
│  │              │    │              │   │             │ │
│  │ "Find near   │    │ ┌──────────┐ │   │ All places  │ │
│  │  me" button  │    │ │   MAP    │ │   │ (no location│ │
│  │              │    │ │ (Leaflet)│ │   │  needed)    │ │
│  └──────────────┘    │ └──────────┘ │   └─────────────┘ │
│         │            │ ┌──────────┐ │                    │
│         │            │ │   LIST   │ │                    │
│         ▼            │ │ (sorted  │ │                    │
│  ┌──────────────┐    │ │  by dist)│ │                    │
│  │  Browser     │    │ └──────────┘ │                    │
│  │  Geolocation │    └──────────────┘                    │
│  │  API         │           ▲                            │
│  └──────────────┘           │                            │
│                    ┌────────┴────────┐                   │
│                    │  Haversine      │                   │
│                    │  distance calc  │                   │
│                    └────────┬────────┘                   │
│                             │                            │
│                    ┌────────┴────────┐                   │
│                    │  places.json    │                   │
│                    │  (static import)│                   │
│                    └─────────────────┘                   │
│                                                          │
└─────────────────────────────────────────────────────────┘

         ┌───────────────────────┐
         │  OpenStreetMap Tiles  │  (free, no API key)
         │  tile.openstreetmap.org│
         └───────────────────────┘
```

## Data Flow

```
1. User opens app
2. User clicks "Find playgrounds near me"
3. Browser requests geolocation permission
4. On success → lat/lng obtained
5. App loads places.json (bundled in app)
6. Haversine formula calculates distance from user to each place
7. Places sorted by distance
8. Map rendered with user pin + playground pins
9. List rendered below map
```

## Data Architecture (expandable)

```
data/
└── places.json          ◀── single file today

places.json structure:
┌──────────────────────────────────────────────┐
│  [                                           │
│    {                                         │
│      "id": "pg-001",                         │
│      "type": "playground",    ◀── expansion  │
│      "name": "Dulwich Park Playground",      │
│      "lat": 51.4445,                         │
│      "lng": -0.0756,                         │
│      "address": "Dulwich Park, London",      │
│      "googleRating": 4.5,                    │
│      "googleMapsUrl": "https://...",         │
│      "description": "..."                   │
│    },                                        │
│    ...                                       │
│  ]                                           │
└──────────────────────────────────────────────┘
```

## Expansion Architecture

Shows how the architecture grows without rewriting:

```
         v0.1 (NOW)              v0.2                    v1.0
     ┌──────────────┐     ┌──────────────┐      ┌──────────────┐
     │  places.json │     │  places.json │      │   Database   │
     │ (playgrounds │     │ + parks      │      │  (Postgres/  │
     │   only)      │     │ + landmarks  │      │   Supabase)  │
     └──────┬───────┘     └──────┬───────┘      └──────┬───────┘
            │                    │                      │
            ▼                    ▼                      ▼
     ┌──────────────┐     ┌──────────────┐      ┌──────────────┐
     │  Static      │     │  Static      │      │  API Routes  │
     │  import      │     │  import +    │      │  /api/places │
     │              │     │  type filter │      │  + auth      │
     └──────┬───────┘     └──────┬───────┘      └──────┬───────┘
            │                    │                      │
            ▼                    ▼                      ▼
     ┌──────────────┐     ┌──────────────┐      ┌──────────────┐
     │  Map + List  │     │  Map + List  │      │  Map + List  │
     │              │     │  + Filters   │      │  + Filters   │
     │              │     │  (by type)   │      │  + Reviews   │
     │              │     │              │      │  + Images    │
     └──────────────┘     └──────────────┘      └──────────────┘
```

## File Structure

```
adventure-map/
├── public/
│   └── favicon.ico
├── src/
│   ├── app/
│   │   ├── layout.tsx           # root layout
│   │   ├── page.tsx             # home page
│   │   └── list/
│   │       └── page.tsx         # full list view
│   ├── components/
│   │   ├── Map.tsx              # Leaflet map wrapper
│   │   ├── PlaceCard.tsx        # single place in list
│   │   ├── PlaceList.tsx        # sorted list of places
│   │   └── LocationButton.tsx   # geolocation CTA
│   ├── lib/
│   │   ├── distance.ts          # Haversine formula
│   │   └── types.ts             # Place type definition
│   └── data/
│       └── places.json          # curated playground data
├── PRODUCT_SPEC.md
├── ARCHITECTURE.md
├── package.json
├── tsconfig.json
└── tailwind.config.ts
```
