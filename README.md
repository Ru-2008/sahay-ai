# 🚨 Sahay AI

**Intelligent Emergency Response & Resource Coordination Platform**

> *Sahay (સહાય / सहाय) means "help" in Gujarati and Hindi*

**Status:** 🚀 Completed for **Bit N Build'26 Gujarat Round** (PS-9).

---

## 📋 Problem

During floods, fires, industrial accidents, and road incidents, information pours in from many disconnected sources — emergency calls, citizen reports, IoT sensors, field teams, and hospitals. Authorities struggle to:

- **See the full situation** across all sources in real time.
- **Identify duplicates** — ten calls about the same collapsed bridge look like ten separate emergencies.
- **Decide which teams and equipment to send**, especially when resources are limited.
- **Detect delays** — a dispatched ambulance that never moves, an incident with no update for five minutes.
- **Coordinate across agencies** when local resources are exhausted.

The result: slower response times, wasted resources, and preventable harm.

---

## 💡 Solution

Sahay AI is a command-centre platform that:

1. **Collects reports** from citizens, calls, sensors, and field teams through a unified ingestion API.
2. **Uses AI to classify** each report by incident type, estimate severity (1–5), and assign priority.
3. **Detects and merges duplicates** — multiple reports about the same event are consolidated into a single incident with a rising confidence score.
4. **Recommends the right teams and equipment** with explainable reasoning, and suggests mutual aid when local units are short.
5. **Shows everything on a live dashboard** — active emergencies, severity, assigned teams, response status, and a real-time map.
6. **Alerts and escalates** — automatic notifications when a critical incident is unassigned, a team is delayed, or an incident has no update.
7. **Routes vehicles on real roads** — uses the Google Maps Routes API for actual road geometry, not straight lines.
8. **Simulates the full response** — physics-based vehicle animation with live telemetry, dynamic rerouting on road closures, and Safe Forward Rejoin.
9. **Keeps a human in the loop** — dispatch is always approved by a human operator.

---

## ✅ Key Features

- **Multi-source incident collection** — citizen reports, emergency calls, sensor data, field team updates
- **AI classification & severity** — automatic incident type, severity (1–5), and priority assignment
- **Duplicate detection & consolidation** — merge related reports into single incidents with confidence scoring
- **Resource recommendation** — AI suggests teams and equipment with reasoning; mutual-aid flag when local resources are short
- **Real-time command dashboard** — active emergencies, severity indicators, assigned teams, response status, live map
- **Alerts & escalation** — critical-unassigned, delayed-response, and no-update alerts with configurable thresholds
- **AI-generated summaries & recommendations** — natural-language incident summaries and dispatch explanations
- **Analytics** — emergency types, response delays, resource shortages, frequently affected areas (hotspot map)
- **Notifications** — Twilio SMS integration and inbound webhooks for external status updates
- **Scenario simulator** — one-click flood, fire, road accident, heatwave, and festival crowd simulations for demo and testing
- **Deterministic Emergency Score** — strict math-based operational score combining AI severity, corroboration, and complexity
- **Audit Logs & Timeline** — rigorous tracking of "who did what, when, and why" for every dispatch action
- **Interactive map filters** — toggle visibility of Incidents, Units, Hospitals, Flood Zones, and Road Closures on the live map
- **Live incidents table** — real-time filterable/searchable incident table with tab filters (All, Active, Critical, High, Medium, Low, Resolved)
- **Live road routing** — Google Maps Routes REST API for real-road geometry with primary + alternate route rendering
- **Physics-based vehicle simulation** — easing curves, dynamic speedometer, live ETA/distance telemetry
- **Dynamic rerouting** — Road Closure and Wrong Turn simulation events trigger Safe Forward Rejoin from the vehicle's exact mid-drive position
- **3D satellite map toggle** — switch between dark mode and tilted 3D hybrid view
- **Active route highlighting** — thick orange for selected route, thin gray for alternates
- **Unified severity colors** — Red (Critical) / Orange (High) / Yellow (Medium) / Blue (Low) across all UI elements

---

## 🌟 What Makes It Different

| Differentiator | Details |
|---|---|
| **Regional language support** | Understands reports in Gujarati, Hindi, and Hinglish — not just English. |
| **Confidence scoring** | Confidence rises as independent reports or sensor readings corroborate an incident; likely false reports are flagged. |
| **Explainable dispatch** | Shows *why* each team was chosen, and suggests mutual aid when local units are stretched thin. |
| **Human-in-the-loop** | AI recommends — a human always gives final approval before dispatch. |
| **Graceful degradation** | If the LLM API is unavailable, a keyword-based fallback classifier keeps the system running. |
| **Real-road routing** | Uses the modern Google Maps Routes REST API — vehicles follow actual road geometry, not straight lines. |
| **Physics-based simulation** | Easing curves produce realistic acceleration/deceleration; live speedometer with jitter; dynamic ETA. |
| **Mid-drive rerouting** | Road Closure and Wrong Turn events trigger instant recalculation from the vehicle's exact position. |

---

## 🏗️ Architecture

> **AI proposes. A human decides. The system never goes dark.**

Sahay is a five-stage pipeline. A report enters through one front door, is understood and matched against what is already known, and ends as a recommendation that only an operator can turn into a dispatch.

| Principle | What it means in the design |
|---|---|
| **Many voices, one incident** | Four sources (citizen, call, sensor, field) converge into a single incident. Every corroborating report raises its confidence. |
| **AI proposes, a human decides** | `POST /incidents/{id}/recommend` changes nothing. Only `POST /incidents/{id}/assign`, with `resource_ids` chosen by the operator, dispatches a unit. |
| **Never dark** | Every AI or network dependency has a local twin. A dead API key or a dropped connection degrades the system; it never stops it. |

### Components

Paths are under `backend/app/` unless shown in full.

| Component | Responsibility | Code |
|---|---|---|
| **Ingestion API** | One front door for all four sources. Validates, normalises and detects the language of each report. | `routers/reports.py`, `services/language.py` |
| **Understand** | One structured LLM call per report returns type, severity (1–5), priority, needed resources, a summary and a false-report flag. Place names resolve through the local gazetteer. A guard tries the Gemini tiers in order inside a 5-second budget, skips any tier that just hit its quota, and falls back to keywords if every tier fails. | `services/classifier.py`, `services/keyword_rules.py`, `services/llm/`, `services/geocode.py` |
| **Deduplicate** | Merges same-type reports that are nearby and recent into one incident, raising confidence and `report_count`. | `services/dedupe.py` |
| **Dispatcher** | Ranks available units by distance, ETA and capacity; attaches reasoning; flags mutual aid; creates assignments only after operator approval. | `services/dispatcher.py`, `routers/incidents.py`, `routers/assignments.py` |
| **Alert engine** | Watches configurable thresholds (for example, a critical incident unassigned for 2 minutes, or a unit not en route within 3) and raises critical, delayed, escalation and shortage alerts. | `services/alerts.py`, `routers/alerts.py` |
| **Notifications** | Records alerts sent to personnel as a mock SMS log. Twilio is optional. | `services/notifications.py` |
| **WebSocket hub** | Pushes `incident_created`, `incident_updated`, `assignment_updated`, `alert_created` and `resource_updated` to every open dashboard. | `services/events.py`, `routers/ws.py` |
| **Analytics** | Incidents by type, average response time, resource shortages and hotspots. | `services/analytics.py`, `routers/analytics.py` |
| **Routing engine** | Calls Google Maps Routes REST API to compute real-road geometry. Returns primary + alternate routes with distance, duration, and decoded polyline paths. | `frontend/src/components/map/LiveMap.jsx` |
| **Simulation engine** | Physics-based vehicle animation along route geometry. Easing curves for realistic speed. Mid-drive rerouting via Safe Forward Rejoin. | `frontend/src/components/map/LiveMap.jsx`, `frontend/src/pages/Dashboard.jsx` |
| **Scenario simulator** | Replays flood, factory-fire, road-accident, heatwave and festival-crowd scripts through the same front door as real reports. | `services/simulator.py`, `routers/simulate.py`, `data/scenarios/` |
| **React dashboard** | Live map with interactive layer filters, incident list and detail drawer, dispatch panel with approval button, live incidents table, resource panel, alert feed and analytics. | `frontend/src/pages/`, `frontend/src/components/` |

### Never dark

Each dependency that can fail has a local twin, so the demo survives a bad network.

| Dependency | Preferred path | Local twin |
|---|---|---|
| Language model | Gemini 3.8 Flash at low thinking; 3.6 Flash and 3.5 Flash-Lite take over on quota errors | A shared 5-second budget, a cache and a per-model cooldown. If every tier fails, the keyword engine returns the same output shape |
| Place lookup | The report carries `lat` and `lng` | The Vadodara gazetteer turns a place name into coordinates, with no online geocoder |
| Notifications | Twilio SMS (optional) | Mock SMS log, so no alert is lost |
| Database | PostgreSQL with PostGIS (optional) | SQLite file, zero setup |
| The whole AI layer | A live LLM provider | `LLM_PROVIDER=mock` runs the entire pipeline with no API key |

### PS-9 coverage

| PS-9 requirement | Sahay component | Contract |
|---|---|---|
| Incident collection from multiple sources | Ingestion API and scenario simulator | `POST /reports`; `source`: citizen, call, sensor, field |
| Classification, severity and priority | Understand | `type`, `severity` 1–5, `priority` |
| Duplicate detection | Deduplicate | `report_count`, `confidence` |
| Resource recommendation | Dispatcher | `POST /incidents/{id}/recommend`; units and facilities |
| Real-time monitoring | WebSocket hub and dashboard | `/ws/live`; incident and assignment status |
| Alerts and escalation | Alert engine | `GET /alerts`; `kind`: critical, delayed, escalation, shortage |
| AI assistance | Summaries and reasoning | `summary` on incidents; `reasoning` on recommendations |
| Analytics | Analytics | `GET /analytics/summary`: incidents\_by\_type, avg\_response\_seconds, resource\_shortages, hotspots |
| Notifications | Notifications | Mock SMS log; `alert_created` event |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11+, FastAPI, SQLAlchemy, WebSockets |
| Database | SQLite (dev), PostgreSQL with PostGIS (optional production) |
| AI | LLM API (Gemini) with keyword-based fallback classifier |
| Frontend | React, Vite, Tailwind CSS |
| Maps | Google Maps Routes REST API, `@vis.gl/react-google-maps`, 3D satellite hybrid view |
| Notifications | Mock SMS log (Twilio integration optional) |

---

## 📁 Project Structure

```text
sahay-ai/
├── backend/                        # Python + FastAPI
│   ├── app/
│   │   ├── main.py                 # FastAPI app entry point
│   │   ├── core/                   # Config, database, error handlers
│   │   ├── models/                 # SQLAlchemy ORM models
│   │   ├── routers/                # API & WebSocket endpoints
│   │   ├── schemas/                # Pydantic request/response schemas
│   │   ├── services/               # AI pipeline, dispatcher, alerts
│   │   │   └── llm/                # Multi-tier Gemini router + mock
│   │   ├── data/                   # Seed data & scenario scripts
│   │   └── utils/                  # Geo & time helpers
│   └── tests/                      # 50+ Pytest automated tests
├── frontend/                       # React + Vite
│   └── src/
│       ├── api/                    # Decoupled fetch/REST clients
│       ├── components/
│       │   ├── map/                # LiveMap, Polyline, markers
│       │   ├── incidents/          # IncidentCard, Drawer, List
│       │   ├── dispatch/           # DispatchModal, recommendations
│       │   ├── alerts/             # AlertFeed, AlertItem
│       │   ├── analytics/          # Charts, shortage tables
│       │   ├── resources/          # Resource panel
│       │   ├── simulate/           # Scenario launcher
│       │   ├── layout/             # AppShell, Sidebar, TopBar
│       │   └── common/             # ErrorBoundary, Spinner, Toast
│       ├── pages/                  # Dashboard, Incidents, Analytics, etc.
│       ├── hooks/                  # useIncidents, useWebSocket, etc.
│       ├── context/                # LiveDataProvider (WebSocket state)
│       ├── constants/              # Shared enums
│       └── utils/                  # Format, geo, time helpers
├── docs/                           # Documentation assets
├── .env.example                    # Environment variable template
├── API_SPEC.md                     # API contract (source of truth)
├── ARCHITECTURE.md                 # Full system architecture (22 sections)
├── DESIGN.md                       # Design decisions
├── PROGRESS.md                     # Development log
├── README.md                       # ← You are here
└── LICENSE                         # MIT
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.11+** and `pip`
- **Node.js 18+** and `npm`

### 1. Clone the repository

```bash
git clone https://github.com/Akshay82480sharma/sahay-ai-v2.git
cd sahay-ai-v2
cp .env.example .env
# Edit .env with your API keys if needed (mock mode works without keys)
```

### 2. Backend

```bash
cd backend
python -m venv venv

# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
uvicorn app.main:app --reload
```

- API docs: [http://localhost:8000/docs](http://localhost:8000/docs)
- Health check: [http://localhost:8000/health](http://localhost:8000/health)

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

- Dashboard: [http://localhost:5173](http://localhost:5173)

### 4. Run the Demo Simulator

Once both backend and frontend are running:

```bash
# Trigger a flood simulation (generates ~15 reports)
curl -X POST http://localhost:8000/simulate/flood
```

Or use the Swagger UI at `/docs` to fire the simulation endpoint.

---

## 🎬 Demo Walkthrough (Flood Scenario)

1. **Trigger simulation** → `POST /simulate/flood` generates ~16 incoming reports (citizen, sensor, field team) across Vadodara. Also available: `fire`, `road_accident`, `heatwave`, `festival`.
2. **AI processes reports** → classifies each as `flood`, estimates severity, geo-locates, and detects duplicates.
3. **Reports merge** → ~16 reports consolidate into ~3 distinct incidents with rising confidence scores.
4. **Dashboard updates live** → incidents appear on the map, severity indicators light up, resource panel shows availability.
5. **Map filtering** → operator toggles Incidents / Units / Hospitals / Flood Zones / Road Closures to focus on relevant layers.
6. **Incidents table** → navigate to the Incidents page to see all incidents in a filterable/searchable table with real-time counts by severity.
7. **Dispatch recommendation** → operator clicks an incident, sees AI-recommended teams with reasoning, approves dispatch.
8. **Vehicle tracks on real roads** → dispatched unit follows actual road geometry with live ETA, speed, and distance telemetry.
9. **Road closure** → operator triggers `[ROAD CLOSURE]` → vehicle instantly reroutes from its current position via Safe Forward Rejoin.
10. **Delayed response alert** → one team doesn't go en-route within the threshold → escalation alert fires.
11. **Analytics view** → charts show incident types, response times, resource usage, and hotspot areas.

---

## 👥 Team

Built for **Bit N Build'26 — Gujarat Round** (Problem Statement PS-9).

| Name | Role | GitHub |
|---|---|---|
| *Rudrarajsinh Rana* | Backend (AI & Dashboard) | [github.com/Ru-2008](https://github.com/Ru-2008) |
| *Akshay Sharma* | Frontend (AI & Dashboard) | [github.com/Akshay82480sharma](https://github.com/Akshay82480sharma) |

---

## 🗺️ Roadmap & Phase Breakdown

The project was built concurrently by two developers across parallel phases:

- **Phase 0** — Repository setup, API contract, shared base models, and Geocoding setup.
- **Phase 1** — Thin slice backend (Report ingestion → AI Classification) and WebSocket real-time wiring.
- **Phase 2** — React Frontend base, Incident Map/List, and Dispatch routing with human approval.
- **Phase 3** — Spatial deduplication engine, Scenario Simulator, automated Alerts, and Analytics.
- **Phase 4** — Real Gemini LLM integration, Twilio SMS, final polish, joint integration tests.
- **Phase 5** — Google Maps Routes API integration, physics-based vehicle simulation, live telemetry, dynamic mid-drive rerouting, and UI unification.

---

## 📄 License

This project is licensed under the [MIT License](./LICENSE).

---

<div align="center">
<sub>Built with ❤️ for smarter emergency response</sub>
</div>
