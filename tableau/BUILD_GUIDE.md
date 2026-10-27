# Tableau Build Guide - Lichess Chess Analytics

**Time needed:** about 60 to 90 minutes in Tableau Desktop or Tableau Public (Windows or Mac).
**What you start with:** the cleaned data, all calculated fields, a colour palette and the dashboard layouts below. **What you do:** connect, relate, paste calculated fields, drag visuals.

> **Honest note:** a Tableau workbook (`.twb` / `.twbx`) can only be created inside Tableau, so the finished file is not included. Everything that *can* be prepared outside Tableau is ready (data model, calculated fields, field types, palette, dashboard designs, check values). The calculated fields were written carefully but not executed in Tableau here, so use the check values in Step 6 to confirm it all matches.

**Version needs:** Tableau 2020.2 or later (the data model uses *relationships*, introduced in 2020.2). Tableau Public is free and fine for this project.

## Files

| File | What it is |
|---|---|
| `data/fact_games.csv` | 19,113 games (de-duplicated), one row per game, with 0/1 flag columns |
| `data/dim_date.csv`, `dim_opening.csv`, `dim_time_control.csv`, `dim_skill_level.csv` | dimension tables |
| `data/player_games.csv` | one row per player per game (38,226 rows), for exact distinct-player counts |
| `data/player_summary.csv` | one row per player (15,635 players) |
| `data/elo_calibration.csv` | observed vs Elo-expected score by rating gap |
| `calculated_fields.txt` | all calculated fields and the parameter, grouped and commented |
| `DATA_DICTIONARY.md` | every column with its Tableau type, role and whether to hide it |
| `Preferences.tps` | colour palettes (categorical, diverging, sequential) |

All CSVs are produced by the last section of `main.ipynb`. Re-run the notebook and refresh the Tableau data source if you change the analysis.

---

## Step 1 - Connect to the data (10 min)

You will make **three data sources**. Tableau requires every table inside one data source to be related, and `player_summary` and `elo_calibration` are pre-aggregated lookup tables that should stay separate.

| Data source (rename it) | Tables |
|---|---|
| **Chess Games** | `fact_games` + 4 dimensions + `player_games` |
| **Players** | `player_summary` |
| **Elo Calibration** | `elo_calibration` |

Connect with **Connect > To a File > Text file** (Tableau Public: **Connect > Text file**). On the Data Source page set the connection to **Extract** (top right). It is faster and is the usual choice for Tableau Public. Then check the field types against `DATA_DICTIONARY.md`. In particular `fact_games[game_date]` and `dim_date[date]` must show the **calendar** icon (Date).

## Step 2 - Build the model (10 min)

In the **Chess Games** data source, drag `fact_games` onto the canvas first, then drag each other table next to it. Tableau draws a line (a relationship) between them. Click each line and set the fields:

| From (fact side) | To (other table) |
|---|---|
| `fact_games[game_date]` | `dim_date[date]` |
| `fact_games[opening_name]` | `dim_opening[opening_name]` |
| `fact_games[time_control]` | `dim_time_control[time_control]` |
| `fact_games[skill_level]` | `dim_skill_level[skill_level]` |
| `fact_games[game_id]` | `player_games[game_id]` |

Leave **Performance Options** on automatic. Unlike joins, relationships keep each table at its own level of detail, so game counts are not inflated when `player_games` is in the same view.

Then, in a worksheet:
1. **Convert to Dimension** (right-click a field > Convert to Dimension): `dim_date[year]`, `fact_games[hour_utc]`. Tableau treats every numeric column as a measure; the full list is in `DATA_DICTIONARY.md`.
2. **Sort by field** (right-click the field > Default Properties > Sort > Sort by: Field): `dim_skill_level[skill_level]` by `skill_sort`; `dim_time_control[time_category]` by `category_sort`; `dim_date[month_name]` by `month_number`; `dim_date[weekday_name]` by `weekday_number`; `fact_games[rating_gap_band]` by `rating_gap_sort`. Use *Minimum* as the aggregation and *Ascending*.
3. **Hide** helper fields (right-click > Hide): all `is_*` flags, `rating_gap_sort`, `skill_sort`, `category_sort`, `month_number`, `weekday_number`.
4. Set **default number formats**: counts as Number with a thousands separator; the `%` calculated fields (Step 3) as Percentage with 1 decimal; `White Score` and `Expected White Score` as Number with 3 decimals.
5. In **Elo Calibration**, right-click `bucket_midpoint` > Convert to Dimension, then use it as **Continuous** (green pill) on the x axis.

## Step 3 - Add the calculated fields (15 min)

First create the **Min Games** parameter described in section 7 of `calculated_fields.txt` (Create Parameter: Integer, 10 to 500, step 10, default 100). Then create each calculated field from that file: Data pane arrow > **Create Calculated Field**, paste the formula, name it exactly as shown.

Fields in sections 1 to 8 and 10 go in the **Chess Games** data source. Section 9 (player fields) goes in **Players**; create `Unique Players` there too, using the same `COUNTD([player_id])` formula.

## Step 4 - Palette and canvas (5 min)

**Colours.** Copy `Preferences.tps` into your `Documents/My Tableau Repository/` folder and restart Tableau (Tableau Public keeps its own repository folder in Documents; if you cannot find it, use the in-workbook palette option below). If you already have a `Preferences.tps`, paste the three `<color-palette>` blocks inside its `<preferences>` tag instead of replacing the file. Palettes then appear under **Edit Colors > Select Color Palette**: *Chess Analytics*, *Chess Analytics Diverging*, *Chess Analytics Sequential*. (Newer Tableau versions can also create palettes inside the workbook; use the hex codes from the file.)

Suggested colours for the winner field: **White = `#E9B44C`**, **Black = `#1F3A5F`**, **Draw = `#8D99AE`**.

**Canvas.** Create each dashboard at **Fixed size 1600 x 900** (16:9). Dashboard > Format > **Shading**: set the dashboard background to light grey `#F3F4F6`. Set each worksheet's background to white with a thin border (Format > Shading, Format > Borders) so the sheets read as cards. Fonts: Tableau has no theme file for fonts, so set them with Dashboard > Format > Font: titles 12 pt semibold `#1F2937`, labels 10 pt `#4B5563`, KPI numbers 28 pt semibold `#1F3A5F` (Segoe UI if you are on Windows, otherwise a Tableau font).

## Step 5 - Worksheets and dashboards

Build each visual on its own worksheet, then place the sheets on a dashboard. Name worksheets like `P1 KPIs`, `P1 Games by month`, so they stay easy to find.

**Page header.** On every dashboard add a title (Dashboard > Show Title, or a text object) and a subtitle sheet: a worksheet with the Marks card set to **Text**, click **Text** > edit and type
`<Total Games> games | <Unique Players> players | white score <White Score>`
inserting each field with the **Insert** menu. The numbers keep their default formats and update with the filters.

**Filters.** On every dashboard except Players, show filters for **Year** (`dim_date[year]`), **Rated** (`fact_games[rated]`), **Skill level** (`dim_skill_level[skill_level]`) and **Time category** (`dim_time_control[time_category]`). For each filter on a worksheet: click the pill's arrow > **Apply to Worksheets > All Using This Data Source**, then **Show Filter**. Because all sheets share the Chess Games data source, one filter then drives every sheet and every dashboard; on each dashboard, place the quick filters from one sheet (arrow > Filters > the field).

### Dashboard 1 - Overview
| Visual | Fields |
|---|---|
| 6 KPI cards | One sheet with `Measure Names` on Columns and `Measure Values` on Text; put `Total Games`, `White Win %`, `Black Win %`, `Draw %`, `Avg Rating`, `Avg Turns` in Measure Values (format each from its pill on the Measure Values card). Or six single-measure sheets. |
| Line chart | Columns: `MONTH(dim_date[date])` continuous; Rows: `Total Games`. August 2017 is a data-collection spike: right-click the mark > Annotate > Mark. |
| 100% stacked column | Columns: `dim_skill_level[skill_level]`; Rows: `Total Games` with quick table calculation **Percent of Total** computed along `winner`; Color: `winner` (palette above). |
| Pie (Tableau has no native donut) | Angle: `Total Games`; Color: `victory_status`. A sorted bar chart is a good alternative; a donut needs a dual-axis trick and is optional. |

*Story:* white wins 49.9% vs black 45.4% (52.4% of decisive games), a small but real first-move advantage.

### Dashboard 2 - Ratings and Upsets
| Visual | Fields |
|---|---|
| Calibration lines (data source **Elo Calibration**) | Columns: `bucket_midpoint` (continuous); Rows: `Measure Values` containing `AVG(actual_white_score)` and `AVG(expected_white_score)`; Marks: Line, Color: `Measure Names`. The gap between the two lines is the point. |
| Column chart | Columns: `fact_games[rating_gap_band]`; Rows: `Upset Rate` |
| Cards | `Upset Rate`, `Higher-Rated Win %` |
| Highlight table (what drives white's score) | Rows: `rating_gap_band`; Columns: `skill_level` and `rated`; Color: `White Score` (palette *Chess Analytics Diverging*, centre 0.5); Text: `White Score` |
| Scatter | Columns: `Avg Rating Gap`; Rows: `Avg Turns`; Detail and Label: `skill_level` |

The highlight table shows how white's score varies with rating gap, skill level and rating status. For a "why is this mark different" view, right-click a mark and use **Explain Data** if your edition offers it. The model-based driver analysis (rating difference alone gives about 65% accuracy) lives in the notebook.

*Story:* upsets fall from about 48% (gap under 50) to about 12% (gap over 500), but observed results are flatter than Elo theory at large gaps (a lower-rated white player scores 14% where Elo predicts under 2%).

### Dashboard 3 - Openings
| Visual | Fields |
|---|---|
| `Min Games` control | Right-click the parameter > **Show Parameter Control** (slider) |
| Bar chart | Rows: `dim_opening[opening_family]`; Columns: `White Score (Min Sample)`; Color: `White Score Bias`; sort descending; add a constant line at 0.5 (Analytics pane > Constant Line). Add filter `Games (Min Sample)` > **Special > Non-null values** so hidden small samples drop off. |
| Table | Rows: `opening_family`; Measure Values: `Total Games`, `White Score`, `White Score CI Low`, `White Score CI High`, `Significance vs 50%`; filter `Total Games` at least 100 |
| Treemap | Marks: Square; Size: `Total Games`; Color and Label: `dim_opening[eco_group]` |
| Matrix | Filter `first_move = e4`; Rows: `black_response`; Columns: `dim_skill_level[skill_level]`; Text and Color: `Black Score (Min Sample)` (diverging, centre 0.5) |

*Story:* Sicilian Defense scores 0.48 for white (good for black); Bishop's Opening 0.61 and English 0.57 favour white. Caveat to mention: weak openings are often played by lower-rated players (Van't Kruijs players average about 1390 vs about 1675 for Sicilian players).

### Dashboard 4 - Time Controls and Game Length
| Visual | Fields |
|---|---|
| Clustered column | Columns: `dim_time_control[time_category]`; Rows: `Measure Values` with `Timeout %` and `Draw %`; Color: `Measure Names` |
| Column chart | Columns: `fact_games[victory_status]`; Rows: `Median Turns` |
| Histogram | Right-click `fact_games[turns]` > **Create > Bins**, size 10; Columns: the `turns (bin)` field; Rows: `Total Games` |
| Bar chart | Rows: `fact_games[time_control]` with filter > **Top** tab > By field: Top 10 by `Total Games`; Columns: `Total Games` |
| 100% stacked bar | Rows: `dim_skill_level[skill_level]`; Columns: `Total Games` with **Percent of Total** along `victory_status`; Color: `victory_status` |

*Story:* losses on time fall from 16% in Bullet (only 123 games) to 3% in Classical; draws are longest games (median 88 turns). Skill level changes how games end more than time control: beginners checkmate 40% of the time, advanced players 20% (they resign 65% of the time).

### Dashboard 5 - Players (data source **Players**, no shared filters)
| Visual | Fields |
|---|---|
| Cards | `Unique Players`, `Regular Players`, `Median Games per Player` |
| Table (leaderboard) | Rows: `player_id`; Measure Values: `games`, `avg_rating`, `win_rate`, `overperformance`; filter `is_regular = Regular (20+ games)`; sort by `overperformance` descending |
| Scatter | Columns: `AVG(avg_rating)`; Rows: `AVG(avg_score)`; Size: `SUM(games)`; Detail: `player_id`; same regular filter |

*Story:* the median player appears in only 1 game, and the top 1% of players make up about 18% of player-games, so player-level results need a minimum-games filter.

---

## Step 6 - Check your numbers

With no filters selected, these cards should match exactly (computed independently in pandas):

| Measure | All games | Rated only | Blitz only | Advanced only | Year 2017 |
|---|---|---|---|---|---|
| Total Games | 19,113 | 15,467 | 3,777 | 4,258 | 13,088 |
| White Win % | 49.9% | 49.9% | 50.2% | 48.2% | 49.3% |
| Black Win % | 45.4% | 45.7% | 45.5% | 45.7% | 45.8% |
| Draw % | 4.6% | 4.4% | 4.3% | 6.1% | 4.9% |
| Avg Rating | 1,594 | 1,596 | 1,599 | 1,964 | 1,606 |
| Avg Turns | 60.5 | 62.0 | 60.6 | 67.4 | 62.2 |
| Median Turns | 55 | 57 | 56 | 62 | 57 |
| Upset Rate | 34.7% | 35.0% | 33.8% | 32.7% | 35.6% |
| Timeout % | 8.4% | 8.4% | 10.3% | 9.2% | 8.1% |
| Checkmate % | 31.3% | 31.6% | 30.8% | 19.6% | 30.5% |
| Resignation % | 56.0% | 55.8% | 54.8% | 65.3% | 56.8% |
| White Score | 0.523 | 0.521 | 0.523 | 0.512 | 0.517 |
| Expected White Score | 0.507 | 0.505 | 0.519 | 0.505 | 0.504 |

Other checks: `Unique Players` = 15,635 (in both the Chess Games and Players data sources); `Regular Players` = 392; `Median Games per Player` = 1. `player_games` has 38,226 rows (exactly 2 per game).

If a number is off: (1) check the relationships point at the right fields and that you chose **All Using This Data Source** on each filter, (2) check `Total Games` is 19,113 and not 38,226 (it must use `COUNTD([game_id])` from `fact_games`, not `player_games`), (3) check `game_date` loaded as a Date, (4) check the `is_*` flags are numbers, (5) for `Median Turns` use an extract.

## Step 7 - Polish and publish (5 to 10 min)

- Align objects on the dashboard grid with consistent card sizes. Titles in sentence case that state the *finding* ("Upsets fade as the rating gap grows") rather than just the field names.
- Add tooltips (Worksheet > Tooltip) and alt text for the key visuals; test the dashboards on filter combinations.
- Optional: filter-action buttons (Dashboard > Actions) for "Rated only" and "Advanced players". Saved custom views need Tableau Cloud or Server.
- Save as `chess_analytics.twbx` (packaged workbook, includes the data). Export a PDF (File > Print to PDF) and take one screenshot per dashboard (Dashboard > Export Image) for the GitHub README.
- Publishing to Tableau Public needs only a free Tableau Public account, which suits a portfolio. Remember that everything on Tableau Public is public, including the data behind the dashboards if downloads are allowed. This data is public Lichess data, but check the download setting before you publish.

## Suggested README section for GitHub

```
## Tableau dashboard
5-dashboard interactive workbook built on a star schema (1 fact + 4 dimensions) from 19,113 de-duplicated Lichess games, using Tableau relationships.
- Dashboards: Overview, Ratings & Upsets, Openings, Time Controls & Game Length, Players
- 35+ calculated fields incl. 95% confidence intervals, upset rate, Elo expected-vs-actual, table calculations for MoM growth, and a parameter for minimum sample size
- Exact distinct-player counts via a long-format player-game table
Key findings: white's first-move advantage is small but significant (52.4% of decisive games); ratings predict results less sharply than Elo theory (higher-rated player wins 65% of decisive games); skill level changes how games end more than time control does.
```

## Interview talking points (Data and AI internship)

- **Data quality first:** I found 945 duplicate games (4.7%) that Part 1 had counted twice. One player went from 154 games to 49 after de-duplication. I also found game duration was unusable (43% of games had zero duration) and measured length in turns instead.
- **Statistics, not just charts:** confidence intervals and significance tests for white's advantage and for each opening, with effect sizes (Cramer's V) to separate "significant" from "meaningful".
- **A surprising finding I checked rather than assumed:** Elo's formula overstates how often the stronger player wins at large gaps in this dataset.
- **No leakage in the model:** only pre-game features were used; rating difference alone gives about 65% accuracy (AUC 0.72) and extra features add nothing.
- **Honest limitations:** selected users, 2017-heavy sample, no engine evaluations, repeat players.
- **Tableau design choices:** a star schema joined with relationships (so measures stay at the right level of detail), flag columns that keep calculated fields simple, a parameter to control minimum sample size, sort-by-field, table calculations for time comparisons, a long-format player table to get exact distinct-player counts, and a documented data model.
