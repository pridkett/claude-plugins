---
name: search-summary
description: >
  Generate a comprehensive weather summary report combining all metrics.
  Use when the user asks for a weather overview, full report, daily summary,
  or "what was the weather like" for a period.
---

# Search Summary

Run these queries via `query_weather` and compile a unified report:

**Current conditions** (last reading):

```sql
SELECT last(tempf), last(humidity), last(baromrelin), last(windspeedmph),
       last(windgustmph), last(winddir), last(hourlyrainin), last(uv), last(solarradiation)
FROM weather WHERE time > now() - 10m
```

**Period highs/lows** (adjust time range):

```sql
SELECT max(tempf), min(tempf), max(windgustmph), max(uv), sum(hourlyrainin)
FROM weather WHERE time > now() - 24h
```

Present as a structured weather report with sections: Temperature, Humidity, Wind, Precipitation, Solar/UV, Pressure.
