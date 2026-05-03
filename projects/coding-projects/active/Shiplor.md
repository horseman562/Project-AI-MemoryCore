# Shiplor (NaviAI) — Maritime Voyage Planning Platform
*Coding Project - Created 2026-04-16*

## Description
A web-based AI-assisted maritime voyage planning tool for shipping professionals. Originally a hackathon project by a friend, now being repurposed as Naqiuddin's assignment project. Helps plan sea routes, manage port communications, and monitor maritime hazards.

## Project Details
- **Type**: Coding Project (Assignment)
- **Status**: Active — Fixing routing + integrating real AI
- **Created**: 2026-04-16
- **Last Accessed**: 2026-04-16 (session 1)
- **Position**: #1
- **Codebase**: `D:\2025_project_portfolio\shiplor`
- **Active Branch**: `main`

## Technical Stack
- **Frontend**: React 18 + TypeScript + Vite
- **UI**: shadcn/ui + Tailwind CSS
- **Map**: Mapbox GL 3.17
- **Routing**: Custom great-circle algorithm (being replaced)
- **State**: React Query + localStorage (no backend)
- **Icons**: Lucide React
- **Notifications**: Sonner

## App Pages / Routes
| Route | Page | Description |
|-------|------|-------------|
| `/` | Index (MapView) | Demo map — hardcoded Tampa Bay → Cozumel route |
| `/dashboard` | Dashboard | Past voyages list + alerts widget + comms widget |
| `/route-planner` | RoutePlanner | Main page — pick ports, draw route, save/next |
| `/loading` | Loading | Fake AI loading screen between planner and comparison |
| `/route-comparison` | RouteComparison | Shows 3 route variants (A/B/C) to choose from |
| `/communications-plan` | CommunicationsPlan | Fake AI generates port authority emails |

## Where Mock/Dummy Data Lives
| File | What's hardcoded |
|------|-----------------|
| `src/pages/Dashboard.tsx:23` | 4 fake past voyages (MV Ocean Star etc.) |
| `src/components/dashboard/AlertFeedWidget.tsx:8` | 4 fake maritime alerts |
| `src/components/route-planner/RouteSummary.tsx:40` | UKHO charts, piracy zones, weather zones |
| `src/components/route-planner/PortSearch.tsx:15` | ~110 real-world ports with hardcoded coordinates |
| `src/pages/RoutePlanner.tsx:27` | Demo waypoints: Le Havre → Covenas → Port Neches |
| `src/components/map/MapView.tsx:12` | Demo route: Tampa Bay → Cozumel (homepage only) |

## Known Bugs
1. **India crossing bug** — Singapore → Abu Dhabi route cuts over India
   - Root cause: `adjustWaypointToAvoidLand()` in `maritimeRouting.ts` only nudges ±1.0 degree — too small to route around India
   - Fix planned: replace with `searoute-js` npm package (real maritime routing)

## Planned Improvements (Phase by Phase)

### Phase 1 — Fix Real Routing ✅ DONE
- Replaced `generateMaritimeRoute()` in `RoutePlanner.tsx` with `searoute-js`
- Installed: `npm install searoute-js`
- Patched `node_modules/geojson-path-finder/dijkstra.js` — fixed ESM/CJS conflict with tinyqueue (`var Queue = _Queue.default || _Queue`)
- Had to clear Vite cache (`rm -rf node_modules/.vite`) for patch to take effect
- Singapore → Abu Dhabi now correctly routes south of India through Indian Ocean ✅

### Phase 2 — Real AI Integration (later)
- Integrate AI (Claude or GPT) for weather prediction along the route
- Details TBD

## Progress Log

### 2026-04-16 (Session 1) — continued
- Full codebase study — understood all pages, mock data, routing logic
- Identified India crossing bug in `src/utils/maritimeRouting.ts`
- Discussed routing fix options: strategic waypoints vs searoute-js vs other APIs
- Decided on `searoute-js` (free, client-side npm package, real maritime routing)
- Also pushed code to GitHub — created `.gitignore` to exclude node_modules
- Installed `searoute-js` and integrated into `RoutePlanner.tsx` ✅
- Fixed ESM/CJS tinyqueue conflict in `node_modules/geojson-path-finder/dijkstra.js` ✅
- Real sea routing working — Singapore → Abu Dhabi correctly avoids India ✅
- **Next step**: Phase 2 — integrate real AI for weather prediction along route

## Key Files
- `src/utils/maritimeRouting.ts` — current routing logic (to be replaced)
- `src/pages/RoutePlanner.tsx` — main planner page, calls `generateMaritimeRoute()`
- `src/components/route-planner/PortSearch.tsx` — port database (~110 ports)
- `src/components/map/MapView.tsx` — demo homepage map (separate from planner)

---
*Coding Project Template v1.0*
