# Queries

### High Risk Loan Identification
#### Identifies large exposures that have already defaulted. The recovery rate indicates how much was recovered, helping to assess loss severity on major defaults.
```
SELECT loan_id, sector, ead, default_date, recovery_rate
FROM loan_portfolio
WHERE defaulted = 1 AND ead > 1000000
ORDER BY ead DESC;
```
#### Results
| loan_id | sector | ead | default_date | recovery_rate |
|---|---|---|---|---|
| 49124.00 | Industrials | 178550752 | 2020-11-01 | 0.376500010490417 |
| 25749.00 | Energy | 129342600 | 2020-05-01 | 0.478799998760223 |
| 17424.00 | Utilities | 111527872 | 2017-07-01 | 0.690999984741211 |
| 5608.00 | Energy | 107779928 | 2022-02-01 | 0.171499997377396 |
| 44903.00 | Financials | 106865496 | 2020-04-01 | 0.629999995231628 |


<details>
<summary>Click to see all/hide</summary>
  
| loan_id | sector | ead | default_date | recovery_rate |
|---|---|---|---|---|

</details>

### Sector Risk Profile
#### Portfolio composition and inherent risk by industry. Higher average PD indicates riskier sectors, while credit score shows borrower quality distribution.

