# Wanderlust — Personalized Travel Itinerary Planner

**Skill Wallet Capstone · Project 4** · Category: Generative AI · Intelligent SaaS Trip Planning · Platform ID `6ac33976fb8fc7965cd9236d`

An AI-powered travel application that generates optimised, day-by-day vacation itineraries. It recommends destinations from your travel *vibe*, clusters nearby attractions onto the same day to eliminate transit fatigue, tracks every cost in local currency / USD / INR, lets you swap individual activities with natural-language requests, and exports a beautifully formatted offline PDF guide.

> Zero external database dependencies. One-click deployable to Vercel. Works fully offline in **Demo mode**, or plans any destination on Earth with a **Gemini** or **Groq** key.

---

## ✨ Feature overview (mapped to the Kanban epics)

| Module | Epic | What it does |
| --- | --- | --- |
| **Configuration & User Preferences** (sidebar) | Epic 3 | Secure API-key input (Gemini / Groq) with masked field, session-only storage by default, connection test and **Demo mode**. Full preference form: destination *or* open-ended vibe (Beach · Mountains · Cultural · Urban), duration (1–14 days), travel dates, budget level, travel party, pace, interests, home currency (USD/INR) and dietary needs. |
| **Tab 1 — Smart Destination Recommender** | Epic 3 | For travellers without a fixed destination: ranks destinations on vibe match, budget fit (cost index vs. tier), **weather season in the travel month**, interests and party type → top-3 cards with score rings, highlight tags, season & budget fit and estimated daily cost. AI mode asks the LLM for worldwide picks and merges them with local data. |
| **Tab 2 — Day-by-Day Optimized Itinerary** | Epic 4 | Structured days with four sequential slots (Morning Sightseeing → Lunch → Afternoon Leisure → Evening Dining & Entertainment). **Geographic clustering** (k-means++ with capacity balancing) keeps each day inside one neighbourhood zone; **nearest-neighbour routing** orders stops within and across days to prevent backtracking. Full-day excursions (e.g. Milford Sound, Jungfraujoch, Dudhsagar) are auto-detected and placed mid-trip. Itemised per-activity costs + per-day transit estimate, with a **running cumulative budget tracker** in local currency, USD and INR, category split and budget-tier guideline check. An SVG cluster map visualises zones and routes. |
| **Tab 3 — Vibe Customization & Activity Swap** | Epic 4 | Swap any single slot with quick prompts (“Swap museum for outdoor market”, “Find vegetarian lunch spot”, “Cheaper option”…) or free text. Intent parsing maps requests to categories/tags; candidates are scored on match × proximity to the day's cluster centre. **Regenerate one day** without touching the rest. Undo stack + change log. AI mode asks the LLM for a real nearby place, validated and re-priced by the engine. |
| **Tab 4 — Trip Management & Offline PDF Export** | Epic 5 | Save unlimited trips (simulated Supabase `trips` table via localStorage), trip history with open / duplicate / delete / JSON export, storage usage and a viewer for the **relational SQL schema**. **1-click PDF**: cover, trip-at-a-glance, per-day tables with costs in local + home currency, budget tracker, emergency contacts, pre-departure & emergency checklist, local tips, page numbers. |

---

## 🧠 Architecture — "LLM for knowledge, algorithms for optimisation"

```
┌──────────────────────────────┐        ┌───────────────────────────────────────────┐
│  Preferences + API config    │        │  Destination knowledge pack               │
│  (sidebar, persisted)        │───────▶│  • Demo: built-in 13-city dataset          │
└──────────────────────────────┘        │  • AI: Gemini / Groq → strict JSON →       │
                                        │    normalised & validated (coords, costs)  │
                                        └───────────────┬───────────────────────────┘
                                                        ▼
                     ┌──────────────────────────────────────────────────────────────┐
                     │  Deterministic itinerary engine  (src/lib/itinerary.ts)       │
                     │  1. Preference scoring of every attraction                    │
                     │  2. Zone selection: k-means++ clustering (src/lib/geo.ts)     │
                     │  3. Nearest-neighbour routing within & across days            │
                     │  4. Slot scheduling, lunch/evening picks near the route       │
                     │  5. Cost itemisation (tier multipliers, FX to USD/INR)        │
                     └───────────────┬──────────────────────────────────────────────┘
                                     ▼
        ┌────────────────┐   ┌────────────────┐   ┌─────────────────┐   ┌──────────────┐
        │ Itinerary view │   │ Swap/Regenerate│   │ localStorage DB │   │ jsPDF export │
        │ + cluster map  │   │ (single slot / │   │ (Supabase-ready │   │ (offline     │
        │ + budget       │   │  single day)   │   │  schema)        │   │  guide)      │
        └────────────────┘   └────────────────┘   └─────────────────┘   └──────────────┘
```

**Why hybrid?** LLMs are excellent at *knowing* places but unreliable at spatial optimisation and arithmetic. Here the model only supplies structured knowledge (names, coordinates, costs, tips) and the engine performs clustering, routing and budgeting deterministically — results are reproducible, cheap, and any malformed AI output degrades gracefully to the offline dataset.

### Key algorithms
- **Haversine distance** for all geo maths.
- **k-means++ clustering with capacity balancing** (`clusterPoints`): natural neighbourhood zones, overflow re-assigned to the nearest zone with room.
- **Zone selection** (`selectZones`): cluster a generous candidate pool, score each zone by its best `perDay` attractions, keep the top `k`, top-up thin zones only with spares ≤ 6 km away.
- **Nearest-neighbour ordering** inside each day (starting from morning-preferred sights) and across days (anchored at the city centre) — this is what prevents cross-city backtracking.
- **Pace → capacity**: Relaxed 2 · Moderate 3 · Fast-Paced 4 sightseeing stops per day; full-day items (≥ 6 h) occupy a whole day.
- **Budget model**: tier multipliers on dining/experiences, per-day transit estimate from route length, currency-aware rounding, daily guideline caps (Budget $60 · Moderate $150 · Luxury $400 pp).

---

## 🛠 Tech stack

| Layer | Choice |
| --- | --- |
| Front-end | React 19 + TypeScript, Vite 7, Tailwind CSS v4 (custom wanderlust design tokens), lucide-react |
| LLM | Google Gemini (`gemini-2.5-flash` default) or Groq (`llama-3.3-70b-versatile` default) via REST with JSON mode; browser-direct calls, keys never leave the client |
| Persistence | localStorage adapter mirroring the Supabase PostgreSQL schema (`users`, `trips`, `itinerary_days`, `activities`, `destination_packs`) — swap in `supabase-js` without changing call sites |
| PDF | jsPDF + jspdf-autotable with Latin-1 transliteration for international place names |
| Build | Single-file production bundle (`vite-plugin-singlefile`) — deployable anywhere static files are served |

---

## 🚀 Getting started

```bash
npm install
npm run dev        # http://localhost:5173
npm run build      # production bundle in dist/
npm run preview    # serve the production build locally
```

### Using AI mode
1. Open **AI Engine** in the sidebar → choose *Gemini* or *Groq*.
2. Paste your key — [Google AI Studio](https://aistudio.google.com/app/apikey) or [Groq Console](https://console.groq.com/keys) (both have free tiers).
3. Click **Test connection**. Keys are kept in `sessionStorage` unless you tick *Remember on this device*.
4. Type **any** destination (e.g. “Hanoi”, “Reykjavík”, “Jaipur”) and generate.

### Demo mode (no key)
Covers Bali, Kyoto, Barcelona, Lisbon, Bangkok, Queenstown, Interlaken (Swiss Alps), Cape Town, Rome, Goa, Dubai, Istanbul and Manali — ~300 real places with coordinates, costs, tags and emergency info.

---

## ☁️ Deployment (Epic 6–7)

**Vercel (recommended)** — import the repo, framework preset *Vite*, build command `npm run build`, output `dist`. No environment variables are required because API keys are entered by the user at runtime.

**Render (static site)** — build command `npm run build`, publish directory `dist`.

Optional hardening for production: move LLM calls behind a FastAPI proxy (keeps keys server-side, adds rate limiting), and replace `src/lib/storage.ts` with a Supabase client using the included schema.

---

## 📁 Project structure

```
src/
├─ App.tsx                     # layout: sidebar (config + preferences) + tabbed workspace
├─ context/TripContext.tsx     # state, orchestration, fallbacks, toasts, undo
├─ components/
│  ├─ ConfigPanel.tsx          # secure API key / provider / model / demo mode
│  ├─ PreferencesForm.tsx      # comprehensive trip preference capture
│  ├─ DayCard.tsx              # boarding-pass style day timeline
│  ├─ BudgetTracker.tsx        # local/USD/INR totals, guideline bar, cumulative chart
│  ├─ ClusterMap.tsx           # SVG map of zones & routes
│  ├─ Layout.tsx, ui.tsx       # header, hero, tabs, toasts, primitives
│  └─ tabs/                    # RecommenderTab · ItineraryTab · CustomizeTab · TripsTab
├─ lib/
│  ├─ geo.ts                   # haversine, k-means++, nearest-neighbour, seeded RNG
│  ├─ itinerary.ts             # generation engine, swap intent parser, day regeneration
│  ├─ recommender.ts           # destination scoring (vibe · budget · season · interests)
│  ├─ llm.ts                   # Gemini/Groq clients, prompts, JSON validation
│  ├─ currency.ts              # FX helpers, tier caps & multipliers
│  ├─ storage.ts               # localStorage "database" + SQL schema
│  └─ pdf.ts                   # offline PDF guide builder
├─ data/                       # built-in destination knowledge base (SQLite-style fallback)
└─ types.ts                    # shared domain types
```

---

## ✅ Quality & error handling
- Every LLM call has a timeout, friendly HTTP error mapping (401/403/404/429), tolerant JSON extraction and schema normalisation (coordinates, categories, numeric coercion). Failures fall back to the offline engine with a visible notice — the app never dead-ends.
- Storage operations are wrapped in try/catch (quota, private mode).
- Engine smoke-tested across 13 destinations × 3 paces × {1, 3, 7, 14} days: every day has all four slots, no duplicate attractions, finite costs; swaps and single-day regeneration leave other days byte-identical.
- Strict TypeScript, accessible controls (labels, roles, aria states), responsive down to mobile.

## 🔭 Roadmap
Live FX rates · hotel-anchored routing · map tiles (MapLibre) · Supabase auth & multi-device sync · FastAPI proxy with caching · shareable public trip links.
