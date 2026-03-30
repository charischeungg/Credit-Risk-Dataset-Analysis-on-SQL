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
#### Shows portfolio composition and inherent risk by industry. Higher average PD indicates riskier sectors, while credit score shows borrower quality distribution.
```
SELECT sector, COUNT(*) AS loan_count, AVG(pd_annual) AS avg_pd, AVG(credit_score) AS avg_credit_score
FROM loan_portfolio
GROUP BY sector
ORDER BY loan_count DESC;
```
#### Results
| sector | loan_count | avg_pd | avg_credit_score | 
|---|---|---|---|
| Energy	| 5132	| 0.023160882903005	| 714| 
| Telecom	| 5077	| 0.0216962964218923	| 716| 
| Consumer	| 5043	| 0.0215236313682699	| 714| 
| Financials	| 5023	| 0.0223081520755852	| 714| 
| Industrials	| 5012	| 0.0219149646568378	| 716| 
| Healthcare	| 4982	| 0.0228557591519898	| 715| 
| Utilities	| 4959	| 0.022074914903457	| 715| 
| Technology	| 4959	| 0.0235746279305457	| 714| 
| Retail	| 4951	| 0.0228694150975067	| 714| 
| Real_Estate	| 4862	| 0.0226562871331607	| 714| 
