---
name: search-humidity
description: >
  Search humidity data from the weather station.
  Use when the user asks about humidity, moisture, dew point context, indoor vs outdoor air quality.
---

# Search Humidity

1. Parse the time range.
2. Call `query_weather`:

```sql
SELECT mean(humidity), min(humidity), max(humidity),
       mean(humidityin), min(humidityin), max(humidityin)
FROM weather WHERE time > now() - 24h GROUP BY time(1h)
```

3. Present outdoor vs indoor humidity side-by-side with timestamps.
