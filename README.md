# UEFA Euros 1960–2024: Tournament & Player Performance Analysis

## Overview
An end-to-end Excel data analytics project covering 17 UEFA European Championship tournaments (1960–2024). The project combines tournament-level results, coaching staff, and full player lineups into a single relational data model, then surfaces the story through an interactive dashboard.

**Business framing:** this kind of analysis mirrors what broadcasters, sports analytics firms, and sponsorship teams use to understand tournament growth, national dominance, and historical performance trends — turning raw historical sports data into decisions worth money (broadcast valuation, sponsorship pricing, scouting insight).

## Tools Used
- Microsoft Excel (Power Query, Data Model, Relationships, PivotTables, PivotCharts)

## Data Sources
Three raw CSV files, sourced together as one dataset:

| File | Grain (one row per...) | Key Columns |
|---|---|---|
| `euro_summary.csv` | Tournament (year) | year, winner, final, result, matches, goals, red_cards, attendance, attendance_average, goals_average, red_cards_average |
| `coaches.csv` | Coach per match | country_code, country, name, role, ID_match, year |
| `lineups.csv` | Player per match | country_code, player name, shirt name, jersey number, position (national/field/detailed), country of birth, birth date, ID_match, ID_player, starting position, start_position_x/y, height, weight, club ID, ID_national_team |

## Data Cleaning Process (Power Query)

**1. Encoding fixes**
Player and coach names with accented characters (e.g. Löw, Beşiktaş-style spellings, Cyrillic-adjacent names) were displaying as `?` or garbled symbols after import. Root cause: Power Query defaulting to the wrong File Origin on import.
- Fix: Power Query Editor → gear icon on the "Source" step → File Origin → set to **Unicode (UTF-8)**.
- Confirmed against the raw CSV in Notepad first, to rule out the file itself being corrupted before troubleshooting Excel's import settings.

**2. Historical country codes**
Several coaches and players were missing `country_code` because the dataset used modern nationality labels for historical, now-dissolved countries (e.g. a coach recorded as "Russian" who actually managed the **Soviet Union** in the 1960s, code `URS`). Resolved by researching the correct era-appropriate country/code rather than using the modern equivalent — applied consistently to any USSR/Yugoslavia/Czechoslovakia-era entries.

**3. Data types**
- `year` → Whole Number (was defaulting to Date in places)
- `matches`, `goals`, `red_cards`, `attendance` → Whole Number
- `attendance_average`, `goals_average`, `red_cards_average` → Decimal Number
- `winner`, `final`, `result` → Text

**4. Structural issue — many-to-many relationship**
`lineups` and `coaches` both repeat `ID_match` (multiple players/coaches per match), which Excel's Data Model can't link directly (needs at least one "one" side). Resolved by building a **bridge table**: a distinct list of `ID_match` values, loaded to the Data Model, sitting between `coaches` and `lineups` as the connecting point.

**5. Blank values — judgment calls, not blanket deletes**
- `start_position_x/y` blank for bench players → correct as-is (substitutes have no starting pitch coordinate)
- `height` / `weight` blank for some 1960s–70s players → left blank (genuinely unrecorded historically), excluded automatically from any AVERAGE() calculations
- `id_club` blank → left as-is (amateur/unlisted club era)

## Data Model
Four tables connected via Data → Relationships:
- `coaches[ID_match]` → `bridge[ID_match]`
- `lineups[ID_match]` → `bridge[ID_match]`
- `coaches[year]` → `euro_summary[year]`

This lets PivotTables pull fields from all three source tables simultaneously without duplicating rows.

## Dashboard

**KPI Cards**
- Total tournaments: 17 (1960–2024)
- Total goals scored: 996
- Total matches played: 388
- Average attendance: ~8,999,926 *(verify — flagged as worth double-checking against source totals)*
- Highest attendance ever: 2,681,288 (single tournament)
- Most successful country: Spain (most tournament wins)

**Charts**
1. **Goals per tournament** — line chart, year (x-axis) vs total goals (y-axis). Shows how attacking output has shifted tournament to tournament.
2. **Attendance growth** — chart tracking total/average attendance by year. Shows the scale of growth in the tournament's popularity from 1960 to 2024.
3. **Tournament wins by country** — bar chart, ranks countries by number of Euros titles. Spain leads, ahead of West Germany/Germany.

**PivotTables built**
- Year + Winner (rows) → Sum of Goals (values)
- Country (rows) → Count of ID_match (values), Year as filter — matches played per country per tournament
- Winner (rows) → Count of Winner (values), sorted descending — win count by country
- Country/Year (rows) → Sum of Attendance (values)

## Known Limitations / Notes
- `attendance_average` figure should be re-verified — the number currently shown is unusually large and may reflect a units or aggregation error worth double-checking against the raw CSV.
- Player/coach names were corrected for known encoding issues; a small number of edge cases may remain unchecked.

## Phase 2 (In Progress / Optional Extension)
Two additional datasets — `qualifying.csv` and `friendlies.csv` — sit in the same source folder, covering pre-tournament matches from 1960–2024 (qualifying) and 2021 onward (friendlies).

**Investigation question:** does strong qualifying-stage form (goals scored in qualifying) correlate with how far a country goes in the actual tournament?

- `qualifying`/`friendlies` use a separate `ID_match` numbering system from the main Euros tables — confirmed by comparing ID ranges — so they are linked via `country_code` (through a second bridge table) rather than `ID_match`.
- The `goals` column in `qualifying` stores each goal as a text record (scorer, minute, phase) rather than a clean number — a Power Query custom column counts occurrences of `"phase"` per cell as a proxy for goals scored, avoiding a full text-parsing rebuild.
- "How far a country went" isn't directly recorded, so **matches played per tournament** (count of ID_match per country/year from `coaches`) is used as a stand-in — more matches played generally means a deeper tournament run under knockout format.

This phase is not required for the core dashboard to be considered complete — it's a value-add extension.

---
*Project by Bokamosho — self-taught data analyst, Bloemfontein, South Africa.*
