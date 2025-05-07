### 2️⃣ `rri_non_playing` – View for Background Context

```sql
CREATE OR REPLACE VIEW rri_non_playing AS
SELECT
  player_id,
  name,
  (
    CASE Prospect_Type
      WHEN 'Top 10' THEN 5
      WHEN 'Top 100' THEN 2.5
      WHEN 'Team Top 10' THEN 1
      ELSE 0 END * 0.05
  +
    CASE 
      WHEN IL_Stints >= 2 THEN -3
      ELSE 0 END * 0.05
  +
    CASE Team_Tier
      WHEN '1st' THEN 4
      WHEN '2nd' THEN 2
      WHEN '3rd' THEN 1
      WHEN '4th' THEN 0
      WHEN '5th' THEN -2
      WHEN 'Top Record' THEN 5
      WHEN 'Time to Tank' THEN -5
      ELSE 0 END * 0.05
  +
    CASE Role_Type
      WHEN 'Closer' THEN 75
      WHEN 'Setup' THEN 70
      WHEN 'HL Reliever' THEN 70
      WHEN 'Middle Relief' THEN 60
      WHEN 'Long Relief' THEN 60
      WHEN 'Mop-Up' THEN 35
      ELSE 0 END * 0.05
  +
    CASE 
      WHEN Age >= 38 THEN -20
      WHEN Age = 37 THEN -15
      WHEN Age = 36 THEN -10
      WHEN Age = 35 THEN -5
      ELSE 0 END * 0.04
  ) AS RRI_Non_Playing
FROM player_bio_and_team;
```
