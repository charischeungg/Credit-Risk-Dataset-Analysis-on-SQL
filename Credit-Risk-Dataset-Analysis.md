# Solution

### High Risk Loan Identification
#### Identifies large exposures that have already defaulted. The recovery rate indicates how much was recovered, helping to assess loss severity on major defaults.
```
SELECT loan_id, sector, ead, default_date, recovery_rate
FROM loan_portfolio
WHERE defaulted = 1 AND ead > 1000000
ORDER BY ead DESC;
```
