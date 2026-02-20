---
name: search-wind
description: >
  Search wind speed, gust, and direction data from the weather station.
  Use when the user asks about wind, gusts, breezy conditions, or wind direction.
---

# Search Wind

1. Parse the time range.
2. Call `query_weather`:

```sql
SELECT mean(windspeedmph), max(windgustmph), last(winddir), last(maxdailygust)
FROM weather WHERE time > now() - 24h GROUP BY time(1h)
```

3. Convert `winddir` degrees to compass bearing (N/NE/E/SE/S/SW/W/NW) when presenting.
4. Highlight peak gust and time of peak gust.
