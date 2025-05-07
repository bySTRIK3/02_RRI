---

### 3️⃣ `rri_total_score` – Final Output View

```sql
CREATE OR REPLACE VIEW rri_total_score AS
SELECT
  cs.player_id,
  cs.name,
  cs.RRI_Current_Season,
  np.RRI_Non_Playing,
  COALESCE(pr.RRI_Previous, 0) AS RRI_Previous,
  (
    cs.RRI_Current_Season * 0.50 +
    np.RRI_Non_Playing * 0.25 +
    COALESCE(pr.RRI_Previous, 0) * 0.25
  ) AS RRI_Total
FROM rri_current_season cs
JOIN rri_non_playing np ON cs.player_id = np.player_id
LEFT JOIN rri_previous_season pr ON cs.player_id = pr.player_id;
```
