# Exoplanet Database & Analysis Engine

Full-stack application analyzing 5,700+ NASA-confirmed exoplanets using PostgreSQL, Node.js/Express, and Chart.js.

## Features
- PostgreSQL with window functions, CTEs, and aggregations
- Verifies Kepler's Third Law computationally across 5,700+ planets
- Computes stellar habitable zones using luminosity-based bounds
- REST API with multiple analytics endpoints
- Frontend visualizations with Chart.js

## Tech Stack
- Backend: Node.js, Express
- Database: PostgreSQL
- Frontend: Chart.js, HTML/CSS

## Run
npm install
node server.js

## What this demonstrates
Real astronomical physics encoded in SQL — not just CRUD. Window functions rank planets by orbital period within star systems, CTEs compute habitable zone boundaries, and aggregations verify the P² ∝ a³ relationship across the entire dataset.
