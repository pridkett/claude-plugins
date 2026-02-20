---
name: search-pressure
description: >
  Search barometric pressure data from the weather station.
  Use when the user asks about barometric pressure, air pressure, weather fronts, or pressure trends.
---

# Search Pressure

1. Parse the time range.
2. Call `query_weather`:

```sql
SELECT mean(baromrelin), mean(baromabsin)
FROM weather WHERE time > now() - 24h GROUP BY time(30m)
```

3. Note pressure trend (rising/falling) by comparing first and last values.
4. Present in inHg. Typical range: 29.5–30.5 inHg. Below 29.5 = low pressure/storm risk.
