Here is a ready-to-use set of **`CREATE VIEW` statements** to help you automate RRI tracking in SQL. These are compatible with **PostgreSQL** or **BigQuery** (minor tweaks needed for others like MySQL or SQL Server).

---

### 1️⃣ `rri_current_season` – View for 2025 Playing Stats

```sql
CREATE OR REPLACE VIEW rri_current_season AS
SELECT
  player_id,
  name,
  (
    -- Section 1.1
    CASE WHEN ERA <= 0 THEN 10
         WHEN ERA <= 2.5 THEN 5
         WHEN ERA <= 4.0 THEN 0
         ELSE -5 END * 0.10
    +
    CASE WHEN xFIP < 3 THEN 5
         WHEN xFIP < 3.5 THEN 2
         WHEN xFIP > 4.5 THEN -3
         ELSE 0 END * 0.05
    +
    CASE WHEN K_per_9 > 13 THEN 3
         WHEN K_per_9 >= 10 THEN 2
         WHEN K_per_9 >= 7 THEN 0
         ELSE -2 END * 0.05
    +
    CASE WHEN BB_per_9 < 2 THEN 3
         WHEN BB_per_9 <= 3 THEN 1
         WHEN BB_per_9 > 4 THEN -3
         ELSE 0 END * 0.05
    +
    CASE WHEN K_pct > 35 THEN 2
         WHEN K_pct >= 30 THEN 1
         WHEN K_pct < 18 THEN -2
         ELSE 0 END * 0.025
    +
    CASE WHEN BB_pct < 5 THEN 2
         WHEN BB_pct > 10 THEN -2
         ELSE 0 END * 0.025
  ) * 0.30
  +
  -- Section 1.2
  CASE 
    WHEN gmLI >= 2.0 THEN 85
    WHEN gmLI >= 1.5 THEN 75
    WHEN gmLI >= 1.0 THEN 65
    WHEN gmLI >= 0.85 THEN 50
    ELSE 35
  END * 0.20
  +
  -- Section 1.3 (Eff + Bonus)
  LEAST(
    (Save_pct * 1.5 + Hold_pct * 1.0) +
    CASE WHEN IR_strand_pct > 0.85 THEN 2
         WHEN IR_strand_pct >= 0.80 THEN 1
         WHEN IR_strand_pct < 0.70 THEN -2
         ELSE 0 END,
  10) * 0.15
  +
  -- Section 1.4
  (
    CASE 
      WHEN StuffAvg >= 141 THEN 6
      WHEN StuffAvg >= 136 THEN 5
      WHEN StuffAvg >= 131 THEN 4
      WHEN StuffAvg >= 126 THEN 3
      WHEN StuffAvg >= 121 THEN 2
      WHEN StuffAvg >= 116 THEN 1
      WHEN StuffAvg >= 111 THEN 0.5
      WHEN StuffAvg >= 100 THEN 0.25
      ELSE 0 END * 0.075
    +
    CASE 
      WHEN CSW_pct > 0.35 THEN 3
      WHEN CSW_pct >= 0.33 THEN 2
      WHEN CSW_pct >= 0.30 THEN 1
      WHEN CSW_pct >= 0.27 THEN 0
      ELSE -1 END * 0.0375
  ) * 0.15
  +
  -- Section 1.5
  (
    CASE 
      WHEN CSW_pct > 0.35 THEN 3
      WHEN CSW_pct >= 0.33 THEN 2
      WHEN CSW_pct >= 0.30 THEN 1
      WHEN CSW_pct >= 0.27 THEN 0
      ELSE -1 END * 0.80
    +
    CASE 
      WHEN Soft_pct >= 25 THEN 1
      WHEN Soft_pct >= 20 THEN 0.5
      WHEN Soft_pct >= 15 THEN 0
      ELSE -1 END * 0.10
    +
    CASE 
      WHEN HardHit_pct < 30 THEN 1
      WHEN HardHit_pct <= 35 THEN 0
      WHEN HardHit_pct <= 39 THEN -1
      ELSE -2 END * 0.10
  ) * 0.10
  +
  -- Section 1.6
  (
    CASE 
      WHEN IR_scored_pct < 0.15 THEN 3
      WHEN IR_scored_pct <= 0.25 THEN 2
      WHEN IR_scored_pct <= 0.35 THEN 1
      WHEN IR_scored_pct <= 0.45 THEN 0
      ELSE -2 END * 0.6
    +
    CASE 
      WHEN Clutch > 0.5 AND gmLI > 1.5 THEN 2.5
      WHEN Clutch > 0.2 THEN 1
      WHEN Clutch < -0.3 THEN -1
      ELSE 0 END * 0.4
  ) * 0.10
  AS RRI_Current_Season
FROM player_weekly_stats;

"Follow GPT creator profile on LinkedIn \[[https://linkedin.com/in/khanukov/](https://linkedin.com/in/khanukov/))"
