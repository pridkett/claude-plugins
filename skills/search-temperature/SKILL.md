---
name: search-temperature
description: >
  Search historical outdoor and indoor temperature data from the weather station.
  Use when the user asks about temperature, heat, cold, how hot/cold it was,
  daily highs and lows, or indoor vs outdoor temperature over a time range.
---

# Search Temperature

When the user asks about temperature data:

1. Parse the requested time range (default: last 24 hours if not specified).
2. Call the `query_weather` MCP tool with an InfluxQL query like:

```sql
SELECT mean(tempf), min(tempf), max(tempf), mean(tempinf)
FROM weather
WHERE time > now() - 24h
GROUP BY time(1h)
```

Adjust the time window (`now() - 24h`, `now() - 7d`, etc.) to match the request.
For a specific date range use: `WHERE time >= '2024-01-01T00:00:00Z' AND time <= '2024-01-31T23:59:59Z'`

3. Present results clearly with timestamps (local time), °F units, and highlight min/max.
