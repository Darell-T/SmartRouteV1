# SmartRoute V1

This repository is the first prototype of what became [SmartRoute](https://github.com/Darell-T/SmartRoute).

The current version is a full NYC transit assistant for subway, bus, and walking trips. You can see it at [smartroute.fyi](https://smartroute.fyi).

V1 started much smaller: given an origin and destination, it tried to answer a simple question using live subway data — **what train should I take, and am I likely to be delayed?**

## What V1 Did

The prototype combined static MTA data, live GTFS-RT feeds, nearby incident signals, and a Claude recommendation layer.

```text
Origin + destination
        |
        v
Geocode addresses
        |
        v
Find nearby subway stations
        |
        v
Find direct route options
        |
        +------------------+
        |                  |
        v                  v
Live MTA GTFS-RT      Nearby incidents
        |                  |
        +--------+---------+
                 |
                 v
      Build transit context
                 |
                 v
      Claude recommendation
```

The main pieces were:

- Geocoding NYC addresses and finding the five nearest subway stations.
- Loading MTA GTFS static files into memory for stop and route lookups.
- Finding direct subway lines shared by origin and destination stations.
- Fetching multiple MTA GTFS-RT feeds concurrently.
- Filtering live trip updates down to the stations relevant to a rider.
- Flagging vehicle-position updates that had gone stale for more than five minutes.
- Scanning for recent incidents near relevant stations with Grok/X data.
- Passing the structured transit context to Claude for a plain-English recommendation.
- Generating optional spoken output through ElevenLabs.
- A Next.js/TypeScript frontend for trip input and result screens.
- Local Postgres and Redis services through Docker Compose.

## Why This Repository Exists

I kept V1 separate instead of rewriting its history because it shows where SmartRoute started.

This version was a subway-only prototype with a relatively simple pipeline. It could discover nearby stations, identify direct route options, pull live arrivals, attach incident context, and ask Claude to make the final recommendation.

The current [SmartRoute](https://github.com/Darell-T/SmartRoute) grew well beyond that design. Routing moved toward backend-owned trip data and validated agent actions, the product expanded beyond subway-only trips, live-data handling became more defensive, and incident work moved out of the request path.

## Project State

This is a historical prototype, not the production SmartRoute codebase.

### Working or substantially implemented

- GTFS static data loading and lookup
- MTA GTFS-RT feed fetching and parsing
- Address geocoding and nearest-station search
- Direct subway route discovery
- Live schedule filtering
- Basic stalled-train detection
- Claude transit recommendations
- Grok-powered incident lookup
- Redis cache helpers
- FastAPI trip endpoint
- Next.js trip and results UI
- Docker Compose configuration for Postgres and Redis

### Prototype gaps

Some parts of the repository were still mid-build when development moved to the next version:

- The frontend is configured to return mock trip data by default.
- The frontend and backend trip-response contracts were not fully aligned.
- Service-alert and WebSocket routers are placeholders.
- Historical delay analysis and database persistence were not completed.
- Background polling and incident caching were planned but not fully wired into the app lifecycle.
- The Grok and ElevenLabs integrations are present in source but their packages are not included in the historical `requirements.txt`.

That is intentional to preserve the repository as it existed rather than retroactively rebuilding V1 into the current architecture.

## Project Structure

```text
SmartRouteV1/
├── backend/
│   ├── app/
│   │   ├── models/
│   │   ├── routers/
│   │   │   ├── trips.py
│   │   │   ├── alerts.py
│   │   │   └── ws.py
│   │   ├── services/
│   │   │   ├── ai_advisor.py
│   │   │   ├── incident_monitor.py
│   │   │   ├── mta_feed.py
│   │   │   ├── route_calculator.py
│   │   │   └── voice.py
│   │   └── utils/
│   │       ├── cache.py
│   │       ├── geo.py
│   │       └── gtfs_static.py
│   ├── data/gtfs_static/
│   └── requirements.txt
├── frontend/
│   ├── app/
│   ├── components/
│   └── lib/
└── docker-compose.yml
```

## Stack

- **Frontend:** Next.js, React, TypeScript, Tailwind CSS
- **Backend:** Python, FastAPI, Pydantic, httpx
- **Transit data:** MTA GTFS Static and GTFS-RT
- **Recommendation layer:** Anthropic Claude
- **Incident lookup:** xAI Grok
- **Voice:** ElevenLabs
- **Storage / infrastructure:** Redis, PostgreSQL, Docker Compose

## Running the Historical Prototype

### Prerequisites

- Python 3.12+
- Node.js 18+
- Docker, if you want the included Redis/Postgres services

Start the local services:

```bash
docker compose up -d
```

Install the backend dependencies:

```bash
cd backend
python -m venv .venv
```

Windows:

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

macOS/Linux:

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

The historical source also references `xai-sdk` and `elevenlabs`, which need to be installed separately if you want to exercise those integrations.

Download the MTA subway GTFS static feed and place its text files under:

```text
backend/data/gtfs_static/
```

Then start FastAPI:

```bash
python -m uvicorn app.main:app --reload
```

Start the frontend in another terminal:

```bash
cd frontend
npm install
npm run dev
```

Because V1 was left in prototype state, running the repository end to end may require small fixes to the old integration points. For the maintained version, use the current [SmartRoute repository](https://github.com/Darell-T/SmartRoute).

## License

MIT
