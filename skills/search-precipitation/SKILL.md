---
name: search-precipitation
description: >
  Search rain and precipitation data from the weather station.
  Use when the user asks about rain, rainfall, precipitation, how much it rained,
  or daily/weekly/monthly rain totals.
---

# Search Precipitation

1. Parse the time range from the user's request.

2. For **daily/accumulated totals**, prefer reading the pre-aggregated fields:

```sql
SELECT last(dailyrainin), last(weeklyrainin), last(monthlyrainin), last(totalrainin)
FROM weather WHERE time > now() - 1h
```

3. For understanding the rate of rain at a particular time, use `hourlyrainin` which takes a point estimate of the rain at that sample and extrapolates to see how much rain would accrue if it kept raining at that rate for a full hour. For example, to get the heaviest rate of rain from the last 30 days:

```sql
SELECT max(hourlyrainin) FROM weather WHERE time > now() - 30d
```

4. Present results in inches, noting accumulation periods.
