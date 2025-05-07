**scheduled ETL process**

**modular SQL expressions** 

---

### 🧮 **1.1 – Performance Metrics**

```sql
-- Assumes ERA, xFIP, K_per_9, BB_per_9, K_pct, BB_pct are columns
(
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
) AS RRI_1_1
```

---

### 🧱 **1.2 – Leverage & Usage**

```sql
CASE 
  WHEN gmLI >= 2.0 THEN 85
  WHEN gmLI >= 1.5 THEN 75
  WHEN gmLI >= 1.0 THEN 65
  WHEN gmLI >= 0.85 THEN 50
  ELSE 35
END AS RRI_1_2
```

---

### 🧷 **1.3 – Saves + Holds Efficiency**

```sql
LEAST(
  (Save_pct * 1.5 + Hold / G) + 
  CASE WHEN IR_pct_stranded > 0.85 THEN 2
       WHEN IR_pct_stranded >= 0.80 THEN 1
       WHEN IR_pct_stranded < 0.70 THEN -2
       ELSE 0 END,
10
) AS RRI_1_3
```

---

### ⚡ **1.4 – Stuff + Pitching Quality**

```sql
(
  CASE 
    WHEN PQ_Avg >= 141 THEN 6
    WHEN PQ_Avg >= 136 THEN 5
    WHEN PQ_Avg >= 131 THEN 4
    WHEN PQ_Avg >= 126 THEN 3
    WHEN PQ_Avg >= 121 THEN 2
    WHEN PQ_Avg >= 116 THEN 1
    WHEN PQ_Avg >= 111 THEN 0.5
    WHEN PQ_Avg >= 100 THEN 0.25
    ELSE 0 END * 0.075
+
  CASE 
    WHEN CSW_pct > 0.35 THEN 3
    WHEN CSW_pct >= 0.33 THEN 2
    WHEN CSW_pct >= 0.30 THEN 1
    WHEN CSW_pct >= 0.27 THEN 0
    ELSE -1 END * 0.0375
+
  CASE WHEN Spin_Rank = 'Top 5%' THEN 2.5
       WHEN Spin_Rank = 'Top 10%' THEN 1
       ELSE 0 END * 0.0375
) AS RRI_1_4
```

---

### 🧱 **1.5 – Batted Ball Performance**

```sql
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
) AS RRI_1_5
```

---

### 🔥 **1.6 – Clutch + Inherited Runners**

```sql
(
  CASE 
    WHEN IR_pct_scored < 0.15 THEN 3
    WHEN IR_pct_scored <= 0.25 THEN 2
    WHEN IR_pct_scored <= 0.35 THEN 1
    WHEN IR_pct_scored <= 0.45 THEN 0
    ELSE -2 END * 0.6
+
  CASE 
    WHEN Clutch > 0.5 AND gmLI > 1.5 THEN 2.5
    WHEN Clutch > 0.2 THEN 1
    WHEN Clutch < -0.3 THEN -1
    ELSE 0 END * 0.4
) AS RRI_1_6
```

Here’s the full **SQL formula breakdown** for final RRI calculation, including **Current Season (50%)**, **Non-Playing (25%)**, and **Previous Season (25%)**:

---

### 🧮 **Current Season RRI (50%)**: Combine Sections 1.1 through 1.6

```sql
(
  RRI_1_1 * 0.30 +
  RRI_1_2 * 0.20 +
  RRI_1_3 * 0.15 +
  RRI_1_4 * 0.15 +
  RRI_1_5 * 0.10 +
  RRI_1_6 * 0.10
) AS RRI_Current_Season
```

---

### 🧠 **Non-Playing RRI (25%)**

Assumes columns:

* `Prospect_Type`: string
* `IL_Stints`: int
* `Team_Tier`: string
* `Role_Type`: string
* `Age`: int

```sql
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
```

---

### 🕰 **Previous RRI (25%)**

If you’ve stored last year’s score in a column:

```sql
Previous_RRI * 0.25 AS RRI_Previous_Weighted
```

---

### ✅ **Final RRI Total Formula**

```sql
(
  RRI_Current_Season * 0.50 +
  RRI_Non_Playing * 0.25 +
  COALESCE(RRI_Previous, 0) * 0.25
) AS RRI_Total
```

---

