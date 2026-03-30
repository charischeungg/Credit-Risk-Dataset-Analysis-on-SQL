# Queries

### Loan Portfolio 
The size and risk of entire portolio
```
SELECT 
    COUNT(*) AS total_loans,
    ROUND(SUM(ead), 0) AS total_exposure,
    ROUND(AVG(pd_annual) * 100, 4) AS avg_default_prob_pct,
    ROUND(AVG(lgd) * 100, 4) AS avg_loss_given_default_pct,
    ROUND(SUM(el), 0) AS total_expected_loss,
    SUM(CASE WHEN defaulted = 1 THEN 1 ELSE 0 END) AS defaulted_loans
FROM loan_portfolio;
```
#### Results
|total_loans	| total_exposure	| avg_default_prob_pct	| avg_loss_given_default_pct	| total_expected_loss	defaulted_loans | 
|---|---|---|---|---|
|50000	| 164930233731	| 2.2461	| 54.6516	1923786054	| 6950|

### Sector Risk Profile
Portfolio composition and  risk by industry. Higher average PD indicates riskier sectors, while credit score shows borrower quality distribution.
```
SELECT sector, COUNT(*) AS loan_count, AVG(pd_annual) AS avg_pd, AVG(credit_score) AS avg_credit_score
FROM loan_portfolio
GROUP BY sector
ORDER BY loan_count DESC;
```
#### Results
| Sector | loan_count | total_exposure | pct_of_portfolio | avg_pd_pct | avg_lgd_pct | avg_credit_score | avg_leverage | avg_interest_coverage | total_expected_loss | expected_loss_rate_pct | defaulted_count | defaulted_exposure | avg_recovery_rate_pct |
|--------|------------|----------------|------------------|------------|-------------|------------------|--------------|----------------------|---------------------|------------------------|-----------------|--------------------|----------------------|
| Technology | 4959 | 12,773,050,126 | 7.74 | 2.3575 | 65.2146 | 714 | 6.91 | 3.74 | 214,071,822 | 1.676 | 676 | 1,821,600,252 | 34.95 |
| Energy | 5132 | 20,935,495,605 | 12.69 | 2.3161 | 50.5284 | 714 | 6.89 | 3.73 | 247,405,336 | 1.1818 | 742 | 3,124,769,738 | 48.95 |
| Retail | 4951 | 3,959,380,564 | 2.40 | 2.2869 | 63.0139 | 714 | 6.92 | 3.80 | 59,674,822 | 1.5072 | 696 | 586,573,467 | 36.71 |
| Healthcare | 4982 | 10,047,744,433 | 6.09 | 2.2856 | 53.4153 | 715 | 6.91 | 3.79 | 118,215,403 | 1.1765 | 657 | 1,169,078,505 | 46.02 |
| Real_Estate | 4862 | 25,305,432,296 | 15.34 | 2.2656 | 45.6393 | 714 | 6.96 | 3.78 | 261,792,215 | 1.0345 | 678 | 3,392,011,780 | 54.51 |
| Financials | 5023 | 30,166,694,294 | 18.29 | 2.2308 | 50.2559 | 714 | 6.92 | 3.76 | 316,107,002 | 1.0479 | 741 | 4,330,575,544 | 50.09 |
| Utilities | 4959 | 22,069,987,326 | 13.38 | 2.2075 | 47.1739 | 715 | 6.89 | 3.78 | 235,836,808 | 1.0686 | 662 | 2,773,427,598 | 53.09 |
| Industrials | 5012 | 15,712,153,595 | 9.53 | 2.1915 | 55.4955 | 716 | 6.96 | 3.72 | 196,375,776 | 1.2498 | 722 | 2,351,033,901 | 45.52 |
| Telecom | 5077 | 18,910,385,176 | 11.47 | 2.1696 | 55.4231 | 716 | 6.93 | 3.80 | 205,262,657 | 1.0854 | 685 | 2,482,441,792 | 44.79 |
| Consumer | 5043 | 5,049,910,316 | 3.06 | 2.1524 | 60.2770 | 714 | 6.84 | 3.74 | 69,044,213 | 1.3672 | 691 | 730,145,922 | 39.68 |
#### Analysis

<details>
<summary>Click to see/hide</summary>
  bbbb
</details>
  
### High Risk Loan Identification
#### Identifies large exposures that have already defaulted. The recovery rate indicates how much was recovered, helping to assess loss severity on major defaults.
```
SELECT loan_id, sector, ead, default_date, recovery_rate
FROM loan_portfolio
WHERE defaulted = 1 AND ead > 1000000
ORDER BY ead DESC;
```
#### Results (First 5 rows)
| loan_id | sector | ead | default_date | recovery_rate |
|---|---|---|---|---|
| 49124.00 | Industrials | 178550752 | 2020-11-01 | 0.376500010490417 |
| 25749.00 | Energy | 129342600 | 2020-05-01 | 0.478799998760223 |
| 17424.00 | Utilities | 111527872 | 2017-07-01 | 0.690999984741211 |
| 5608.00 | Energy | 107779928 | 2022-02-01 | 0.171499997377396 |
| 44903.00 | Financials | 106865496 | 2020-04-01 | 0.629999995231628 |
<details>
<summary>Click for link to full file</summary>
<a href="./query%20results/High%20Risk%20Loan%20Identification%20Output.csv">View Full Results</a>
</details>
