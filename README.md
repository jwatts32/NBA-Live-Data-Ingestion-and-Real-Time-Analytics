# 🏀 NBA Live Stats Ingestion & Real-Time Analytics

Real-time data pipeline that ingests live NBA player and game stats from the [BallDontLie API](https://www.balldontlie.io/) into Delta Lake using PySpark in Databricks. This pipeline ignites real-time dashboards and in-game predictive modeling.

---

## 📊 Features

- ✅ Real-time ingestion from BallDontLie API for live stats updates
- ✅ Delta Lake storage with schemas
- ✅ History tables to align with in-game machine learning models
- ✅ SQL + Python friendly querying in Databricks
---

## 🚀 How It Works

1. Starts a Spark session in Databricks
2. Pulls live data for active NBA games and player stats
3. Writes data to:
   - "games" and "player_stats": current state tables that uses upsert to provide most recent data
   - "games_hist" and "player_stats_hist": historical tables that append data to analyze the progression of the game
4. Data is saved to Delta Lake
5. Tables are auto-registered for sql queries
   
---

## 🔍 Example Queries

%sql
-- Gets the number of points by player for each loop and how many points added since last loop
WITH points_progression AS (
  SELECT
    player_id,
    game_id,
    run_timestamp,
    points,
    LAG(points) OVER (
      PARTITION BY player_id, game_id
      ORDER BY run_timestamp
    ) AS previous_points
  FROM player_stats_hist
)
SELECT
  player_id,
  game_id,
  run_timestamp,
  points,
  COALESCE(points - previous_points, points) AS points_scored_this_run
FROM points_progression
ORDER BY player_id, game_id, run_timestamp;

