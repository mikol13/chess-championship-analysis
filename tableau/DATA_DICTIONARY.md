# Data dictionary - Tableau data sources

Tableau reads the CSVs with the Text file connector and guesses types. This page lists what each column should be, so you can check the guess after connecting.

**Two things to know about Tableau data sources:**

1. Every numeric column arrives as a *Measure*. Fields you want to filter or put on an axis as categories need **right-click > Convert to Dimension** (listed below). Fields marked *Hide* are helper columns: right-click > **Hide**.
2. A Tableau data source needs all its tables related to each other, so the three groups below are three separate data sources: **Chess Games** (6 tables), **Players** (1 table) and **Elo Calibration** (1 table).

| Data source | Table | Rows | Role in the model |
|---|---|---|---|
| Chess Games | `fact_games` | 19,113 | fact - one row per game |
| Chess Games | `player_games` | 38,226 | related to fact_games on game_id - one row per player per game |
| Chess Games | `dim_date` | 1,480 | dimension |
| Chess Games | `dim_opening` | 1,477 | dimension |
| Chess Games | `dim_time_control` | 400 | dimension |
| Chess Games | `dim_skill_level` | 3 | dimension |
| Players | `player_summary` | 15,635 | standalone, pre-aggregated - separate data source |
| Elo Calibration | `elo_calibration` | 14 | standalone, pre-aggregated - separate data source |

## `fact_games` (fact_games.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `game_id` | String | Dimension | Lichess game ID (unique key). Also present in player_games. |
| `game_date` | Date | Dimension | Date the game was created. Relates to dim_date[date]. |
| `rated` | String | Dimension | Rated or Casual. Filter on every page. |
| `turns` | Number (whole) | Measure | Number of turns in the game (game length). |
| `victory_status` | String | Dimension | Checkmate, Resignation, Out of time or Draw. |
| `winner` | String | Dimension | White, Black or Draw. |
| `time_control` | String | Dimension | Time-control code, base minutes + increment seconds (e.g. 15+2). Relates to dim_time_control. |
| `white_id` | String | Dimension | White player ID. |
| `white_rating` | Number (whole) | Measure | White player rating. |
| `black_id` | String | Dimension | Black player ID. |
| `black_rating` | Number (whole) | Measure | Black player rating. |
| `avg_rating` | Number (decimal) | Measure | Mean of the two ratings. |
| `rating_diff` | Number (whole) | Measure | White rating minus black rating. |
| `abs_rating_diff` | Number (whole) | Measure | Absolute rating gap. |
| `rating_gap_band` | String | Dimension | Gap band: 0-50, 51-100, 101-200, 201-300, 301-500, 500+. Sort by rating_gap_sort. |
| `rating_gap_sort` | Number (whole) | Measure (hide) | Sort key for rating_gap_band (1-6). Hide. |
| `skill_level` | String | Dimension | Beginner, Intermediate or Advanced (from avg_rating). Relates to dim_skill_level. |
| `opening_name` | String | Dimension | Full opening name. Relates to dim_opening. |
| `opening_ply` | Number (whole) | Measure | Number of plies in the opening. |
| `first_move` | String | Dimension | White first move (e.g. e4, d4). |
| `black_response` | String | Dimension | Black first reply, or "(no reply)". |
| `white_score` | Number (decimal) | Measure | 1 = white win, 0.5 = draw, 0 = white loss. |
| `expected_white_score` | Number (decimal) | Measure | Elo-expected score for white from the rating gap. |
| `is_white_win` | Number (whole) | Measure (hide) | Flag: white won. Hide. |
| `is_black_win` | Number (whole) | Measure (hide) | Flag: black won. Hide. |
| `is_draw` | Number (whole) | Measure (hide) | Flag: draw. Hide. |
| `is_decisive` | Number (whole) | Measure (hide) | Flag: win or loss (not a draw). Hide. |
| `is_upset_eligible` | Number (whole) | Measure (hide) | Flag: decisive game between unequal ratings. Hide. |
| `is_upset` | Number (whole) | Measure (hide) | Flag: lower-rated player won (only where eligible). Hide. |
| `is_timeout` | Number (whole) | Measure (hide) | Flag: ended on time. Hide. |
| `is_resign` | Number (whole) | Measure (hide) | Flag: ended by resignation. Hide. |
| `is_mate` | Number (whole) | Measure (hide) | Flag: ended by checkmate. Hide. |
| `hour_utc` | Number (whole) | Measure -> **Convert to Dimension** | Hour of day (UTC) the game started. |

## `player_games` (player_games.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `game_id` | String | Dimension | Relates to fact_games[game_id]. Shown as "game_id (player_games)" in Tableau. |
| `player_id` | String | Dimension | Player ID (white and black players in one column). |
| `color` | String | Dimension | White or Black. |
| `player_rating` | Number (whole) | Measure | This player rating. |
| `opponent_rating` | Number (whole) | Measure | Opponent rating. |
| `player_score` | Number (decimal) | Measure | 1 = win, 0.5 = draw, 0 = loss, from this player perspective. |

## `dim_date` (dim_date.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `date` | Date | Dimension | Calendar day. Relates to fact_games[game_date]. |
| `year` | Number (whole) | Measure -> **Convert to Dimension** | Year. |
| `quarter` | String | Dimension | Q1-Q4. |
| `month_number` | Number (whole) | Measure (hide) | 1-12. Sort key for month_name. Hide. |
| `month_name` | String | Dimension | Jan-Dec. Sort by month_number. |
| `year_month` | String | Dimension | YYYY-MM text. |
| `weekday_number` | Number (whole) | Measure (hide) | 1 (Mon) - 7 (Sun). Sort key for weekday_name. Hide. |
| `weekday_name` | String | Dimension | Mon-Sun. Sort by weekday_number. |
| `is_weekend` | String | Dimension | Weekend or Weekday. |

## `dim_opening` (dim_opening.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `opening_name` | String | Dimension | Key. Relates to fact_games[opening_name]. |
| `opening_eco` | String | Dimension | ECO code. |
| `opening_family` | String | Dimension | Opening family (name before the colon). |
| `eco_group` | String | Dimension | ECO group (A-E) with description. |

## `dim_time_control` (dim_time_control.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `time_control` | String | Dimension | Key. Relates to fact_games[time_control]. |
| `base_minutes` | Number (whole) | Measure | Base time in minutes. |
| `increment_sec` | Number (whole) | Measure | Increment in seconds. |
| `time_category` | String | Dimension | Bullet, Blitz, Rapid or Classical. Sort by category_sort. |
| `estimated_minutes` | Number (decimal) | Measure | Base + 40 moves of increment, in minutes. |
| `category_sort` | Number (whole) | Measure (hide) | Sort key for time_category (1-4). Hide. |

## `dim_skill_level` (dim_skill_level.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `skill_level` | String | Dimension | Key. Relates to fact_games[skill_level]. Sort by skill_sort. |
| `skill_sort` | Number (whole) | Measure (hide) | 1-3. Hide. |
| `rating_band` | String | Dimension | Rating range in words. |

## `player_summary` (player_summary.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `player_id` | String | Dimension | Player ID (unique). |
| `games` | Number (whole) | Measure | Games played. |
| `avg_rating` | Number (decimal) | Measure | Mean rating. |
| `peak_rating` | Number (whole) | Measure | Highest rating seen. |
| `win_rate` | Number (decimal) | Measure | Share of games won. |
| `avg_score` | Number (decimal) | Measure | Mean score (win 1, draw 0.5, loss 0). |
| `expected_score` | Number (decimal) | Measure | Mean Elo-expected score. |
| `overperformance` | Number (decimal) | Measure | avg_score minus expected_score. |
| `white_share` | Number (decimal) | Measure | Share of games played as white. |
| `is_regular` | String | Dimension | "Regular (20+ games)" or "Casual (<20 games)". Filter the leaderboard to Regular. |

## `elo_calibration` (elo_calibration.csv)

| Column | Tableau type | Role | Notes |
|---|---|---|---|
| `diff_bucket` | String | Dimension | Rating-difference bucket label. |
| `bucket_midpoint` | Number (decimal) | Measure -> **Convert to Dimension** | Bucket midpoint. Use as a continuous Dimension on the x axis. |
| `games` | Number (whole) | Measure | Games in bucket. |
| `actual_white_score` | Number (decimal) | Measure | Observed white score. |
| `expected_white_score` | Number (decimal) | Measure | Elo-expected white score. |

