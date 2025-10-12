# What is PromQL, MetricsQL? Explained (formulas + SQL)
## rate() function
```
rate = (last_value_in_range - first_value_in_range_adjusted_for_resets) / duration_of_range_in_seconds
```
- last_value_in_range: The value of the counter at the end of the specified time range.
- first_value_in_range_adjusted_for_resets: The value of the counter at the beginning of the specified time range, with automatic adjustments made to account for any counter resets (e.g., if an application restarted and the counter value dropped to zero). Prometheus intelligently handles these resets to provide a continuous rate of increase.
- duration_of_range_in_seconds
