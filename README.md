# Exoplanet Database & Analysis Engine

Full-stack application analyzing 5,700+ NASA-confirmed exoplanets. Real astronomical physics encoded in SQL — not just CRUD.

## Tech Stack

- **Backend**: Node.js, Express
- **Database**: PostgreSQL
- **Frontend**: Chart.js, HTML/CSS

## Setup

1. Clone the repo
2. Install dependencies:
npm install
3. Copy `.env.example` to `.env` and fill in your PostgreSQL credentials:
cp .env.example .env
4. Create the database and import the dataset:
psql -U postgres -c "CREATE DATABASE exoplanet;"

psql -U postgres -d exoplanet -c "CREATE TABLE planets AS SELECT * FROM ..."
5. Run the server:
node server.js
## API Endpoints

| Endpoint | Description |
|---|---|
| `GET /planets` | First 50 planets |
| `GET /planets/search` | Filter by method, mass, distance |
| `GET /stats/methods` | Discovery method breakdown |
| `GET /stats/kepler` | Kepler's Third Law verification |
| `GET /stats/habitable` | Planets in stellar habitable zones |
| `GET /stats/timeline` | Cumulative discovery timeline |
| `GET /stats/ranking` | Top 3 planets by mass per method |
| `GET /stats/trends` | Year-over-year method trends |
| `GET /stats/growth` | Annual discovery growth via LAG() |

## The Physics

**Kepler's Third Law** — P² ∝ a³

The `/stats/kepler` endpoint computes `(period² / semiMajorAxis³)` for every planet orbiting a solar-mass star. A constant ratio across 5,700+ planets confirms the law holds dataset-wide.

**Habitable Zone calculation**

The `/stats/habitable` endpoint computes inner and outer habitable zone bounds using stellar luminosity:
hz_inner = 0.75 * sqrt(R² * (T/5778)⁴)

hz_outer = 1.50 * sqrt(R² * (T/5778)⁴)

Where R is stellar radius and T is effective temperature relative to the Sun (5778K). Planets whose orbital distance falls within these bounds are flagged as potentially habitable.

## SQL highlights

**Window function — cumulative discovery timeline:**
```sql
SELECT disc_year,
       COUNT(*) as discovered,
       SUM(COUNT(*)) OVER (ORDER BY disc_year) as cumulative
FROM planets
GROUP BY disc_year
ORDER BY disc_year
```

**Window function — year-over-year growth with LAG():**
```sql
SELECT disc_year,
       COUNT(*) as discovered,
       LAG(COUNT(*)) OVER (ORDER BY disc_year) as prev_year,
       COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY disc_year) as growth
FROM planets
GROUP BY disc_year
ORDER BY disc_year
```

**RANK() — top 3 planets by mass per discovery method:**
```sql
SELECT pl_name, discoverymethod, pl_bmassj, rank
FROM (
    SELECT pl_name, discoverymethod, pl_bmassj,
           RANK() OVER (PARTITION BY discoverymethod ORDER BY pl_bmassj DESC) as rank
    FROM planets WHERE pl_bmassj IS NOT NULL
) ranked
WHERE rank <= 3
```

## Common interview questions this project covers

**What is a window function and how does it differ from GROUP BY?**
GROUP BY collapses rows into one row per group. Window functions compute across a set of rows related to the current row without collapsing them — you keep all the original rows plus the computed value. SUM() OVER gives a running total; LAG() accesses the previous row's value.

**What is LAG() and what problem does it solve here?**
LAG() accesses the value from a previous row in the result set without a self-join. For year-over-year growth, the alternative would be joining the table to itself on `disc_year = disc_year - 1` — more verbose and slower. LAG() does it in one pass.

**Why parameterized queries in the search endpoint?**
The search endpoint builds dynamic SQL from user input. Without parameterization, a user could inject `'; DROP TABLE planets; --` as a method name and destroy the database. Parameterized queries (`$1`, `$2`) pass values separately from the query structure — the database never interprets them as SQL.

**What is a CTE and why use one?**
A CTE (Common Table Expression) defined with WITH is a named subquery you can reference multiple times in the same query. In the trends endpoint, `yearly_stats` and `method_totals` are computed once and joined — without CTEs you'd repeat the same subquery twice, making it harder to read and maintain.

**Why ROUND()::numeric in PostgreSQL?**
PostgreSQL's ROUND() only accepts the numeric type for decimal rounding, not float. The `::numeric` cast converts the float result of the habitable zone calculation before rounding — otherwise it throws a type error.
