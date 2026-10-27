# Lichess Chess Analytics

Exploratory analysis and a Tableau dashboard on about 20,000 online chess games from Lichess: what influences who wins, how games end, and how much ratings and openings really matter.

## What's in this repo

| Path | What it is |
|---|---|
| `main.ipynb` | Analysis notebook. Part 1: exploration. Part 2: data cleaning, statistical tests, predictive model, key findings, and the export of the Tableau data. |
| `tableau/` | Everything needed to build the Tableau workbook (see below). |
| `tableau/data/` | Star-schema CSVs written by the last section of the notebook. |

The notebook reads the raw Lichess export as `games.csv` in the project root. That file is not included in this repo, so add it there before re-running the notebook.

## Tableau dashboard

A 5-dashboard interactive workbook on a star schema (1 fact table + 4 dimensions) of 19,113 de-duplicated games, joined with Tableau relationships.

- **Dashboards:** Overview, Ratings & Upsets, Openings, Time Controls & Game Length, Players
- **Calculated fields:** 95% confidence intervals, upset rate, Elo expected-vs-actual, table calculations for month-over-month growth, and a parameter for the minimum sample size
- **Exact distinct-player counts** through a long-format player-game table

The `.twbx` workbook file has to be created in Tableau, so this repo ships the kit instead:

| File | Purpose |
|---|---|
| [`tableau/BUILD_GUIDE.md`](tableau/BUILD_GUIDE.md) | Step-by-step build (about 60 to 90 minutes), dashboard layouts, and check values to confirm your numbers |
| [`tableau/calculated_fields.txt`](tableau/calculated_fields.txt) | Every calculated field and the parameter, ready to paste |
| [`tableau/DATA_DICTIONARY.md`](tableau/DATA_DICTIONARY.md) | Each column's type, role and whether to hide it |
| [`tableau/Preferences.tps`](tableau/Preferences.tps) | Colour palettes |
| `tableau/data/*.csv` | `fact_games`, `dim_date`, `dim_opening`, `dim_time_control`, `dim_skill_level`, `player_games`, `player_summary`, `elo_calibration` |

To start: open Tableau Desktop (or the free Tableau Public) and follow the build guide. To refresh the data, re-run section 14 of the notebook and refresh the Tableau data source.

## Key findings

1. **Data quality:** 945 duplicate games (4.7%) were inflating the first-pass counts, and game duration was unusable (43% of games have zero duration), so game length is measured in turns.
2. **First-move advantage is real but small:** white wins 49.9% vs black 45.4%, or 52.4% of decisive games.
3. **Ratings predict results less sharply than Elo theory:** the higher-rated player wins 65% of decisive games, and results are far flatter than the Elo curve at gaps over 400 points.
4. **Upsets fade with the rating gap:** 48% within 50 points, 12% beyond 500.
5. **Skill level changes how games end more than time control does:** beginners checkmate 40% of the time, advanced players 20% (they resign 65% of the time).
6. **Opening rankings need context:** results are tangled up with the rating of the players who choose each opening.
7. **Model ceiling:** rating difference alone gives about 65% accuracy (ROC-AUC 0.72); extra pre-game features add nothing.

**Limitations:** a sample of selected users rather than all of Lichess; heavy concentration in 2017; no engine evaluations; repeat players mean games are not fully independent.
