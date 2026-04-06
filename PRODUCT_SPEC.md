# Adventure Map v0.1 — Product Specification

## Overview

A single-page web app that helps parents find the nearest playground from a curated Google Maps list. Users share their location and instantly see the closest playgrounds on a map and in a sorted list.

## User Story

> As a parent, I want to open the app, share my location, and immediately see the closest playgrounds from a curated list so I can decide where to go.

## Scope

### In Scope (v0.1)
- Import playground data from the Google Maps list
- "Find playgrounds near me" — browser geolocation
- Map view showing playground pins
- List view of playgrounds sorted by distance from user
- Each playground shows: name, distance, Google Maps rating, address
- Clicking a playground opens directions in Google Maps

### Out of Scope (future)
- User accounts / authentication
- Personal reviews or ratings
- Images
- Printable adventure lists
- Categories beyond playgrounds (parks, landmarks, etc.)
- Admin UI for managing places

## Functional Requirements

### F1: Playground Data
- Playground data is stored as a static JSON file (`data/playgrounds.json`)
- Each entry has: `name`, `lat`, `lng`, `address`, `googleRating`, `googleMapsUrl`
- Data is seeded once from the Google Maps list, and can be manually updated
- **Why JSON file**: no backend needed for v0.1; when we expand to more place types, we migrate to a database

### F2: Home Page
- Shows app title and a prominent "Find playgrounds near me" button
- On click: requests browser geolocation permission
- On success: transitions to the results view
- On denial: shows a message explaining location is needed, with option to enter a postcode manually

### F3: Results View — Map
- Full-width map (Leaflet + OpenStreetMap — free, no API key)
- User location shown as a distinct marker
- Each playground shown as a pin
- Pins are numbered to match the list below
- Clicking a pin shows a popup with playground name, rating, and "Get directions" link

### F4: Results View — List
- Below the map, a scrollable list of all playgrounds sorted by distance (nearest first)
- Each card shows:
  - Playground name
  - Distance (e.g., "0.8 km")
  - Google Maps rating (stars)
  - Address
  - "Directions" button → opens Google Maps directions in new tab
- Distance is calculated client-side using the Haversine formula

### F5: Responsive Design
- Mobile-first layout (primary use case is a parent on their phone)
- Map takes ~50% of viewport on mobile, list below
- On desktop, map and list can sit side-by-side

## Non-Functional Requirements

- **No backend required** — entirely static, can be hosted on GitHub Pages or Netlify
- **No API keys required** — uses Leaflet/OpenStreetMap (free) instead of Google Maps JS API
- **Fast** — playground data loads from a local JSON file, no external API calls
- **Accessible** — semantic HTML, keyboard navigable, readable contrast
- **Expandable** — data model and architecture support adding new place types easily

## Data Model

```
Place {
  id:            string     // unique identifier
  type:          string     // "playground" (expandable: "park", "landmark", etc.)
  name:          string
  lat:           number
  lng:           number
  address:       string
  googleRating:  number     // 1-5
  googleMapsUrl: string     // link to Google Maps listing
  description:   string     // optional, short description
}
```

The `type` field exists from day one so that when we add parks, landmarks, etc., the data model doesn't change — we just add entries with different types and add UI filters.

## Tech Stack

| Layer      | Choice                        | Rationale                                    |
|------------|-------------------------------|----------------------------------------------|
| Framework  | Next.js (App Router)          | React-based, easy to add API routes later    |
| Styling    | Tailwind CSS                  | Rapid prototyping, mobile-first utilities    |
| Map        | Leaflet + OpenStreetMap tiles | Free, no API key, well-supported             |
| Data       | Static JSON file              | Simplest; migrate to DB when needed          |
| Hosting    | Vercel / GitHub Pages         | Free tier, zero-config for Next.js           |
| Language   | TypeScript                    | Type safety for the data model               |

## Pages / Routes

| Route   | Description                                    |
|---------|------------------------------------------------|
| `/`     | Home page with "Find playgrounds near me" CTA  |
| `/list` | Full list of all playgrounds (no location needed) |

## Expansion Path

This architecture is designed to grow:

1. **More place types** → Add entries to `playgrounds.json` (rename to `places.json`) with `type: "park"` etc. Add filter UI.
2. **Database** → Replace JSON import with a fetch to an API route (`/api/places`). Next.js makes this trivial.
3. **Admin UI** → Add authenticated routes to manage places via the API.
4. **User reviews** → Add a reviews collection linked to place IDs.
5. **Images** → Add an `images` array to the Place model, store in S3/Cloudinary.

## Success Metrics

- User can find their nearest playground in under 5 seconds
- Works on mobile Chrome/Safari without issues
- Page loads in under 2 seconds on 3G
