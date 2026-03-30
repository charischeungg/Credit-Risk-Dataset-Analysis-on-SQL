# Analysis

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
Aggregate and summarize key credit risk metrics across industry sectors within a loan portfolio
```
SELECT
    sector,
    COUNT(*) AS loan_count,
    ROUND(SUM(ead), 0) AS total_exposure,
    ROUND(AVG(pd_annual) * 100, 4) AS avg_pd_pct,
    ROUND(AVG(lgd) * 100, 4) AS avg_lgd_pct,
    ROUND(SUM(el), 0) AS total_expected_loss,
    ROUND(SUM(unexpected_loss), 0) AS total_unexpected_loss,
    ROUND(AVG(credit_score), 0) AS avg_credit_score,
    -- CASE statement to create a dynamic risk rating
    CASE
        WHEN AVG(pd_annual) < 0.01 THEN 'Low Risk'
        WHEN AVG(pd_annual) BETWEEN 0.01 AND 0.03 THEN 'Medium Risk'
        ELSE 'High Risk'
    END AS sector_risk_rating,
    -- Calculate % of total portfolio exposure
    ROUND(SUM(ead) / (SELECT SUM(ead) FROM loan_portfolio) * 100, 2) AS pct_of_total_exposure
FROM
    loan_portfolio
GROUP BY
    sector
ORDER BY
    total_expected_loss DESC;
```
#### Results
| Sector | loan_count | total_exposure | avg_pd_pct | avg_lgd_pct | total_expected_loss | total_unexpected_loss | avg_credit_score | sector_risk_rating | pct_of_total_exposure |
|--------|------------|----------------|------------|-------------|---------------------|-----------------------|------------------|--------------------|-----------------------|
| Financials | 5023 | 30,166,669,294 | 2.2308 | 50.2559 | 316,107,002 | 1,555,339,098 | 714 | Medium Risk | 18.29 |
| Real_Estate | 4862 | 25,305,432,296 | 2.2656 | 45.6393 | 261,792,215 | 1,215,121,273 | 714 | Medium Risk | 15.34 |
| Energy | 5132 | 20,935,495,605 | 2.3161 | 50.5284 | 247,405,336 | 1,145,353,553 | 714 | Medium Risk | 12.69 |
| Utilities | 4959 | 22,069,987,326 | 2.2075 | 47.1739 | 235,836,808 | 1,105,816,026 | 715 | Medium Risk | 13.38 |
| Technology | 4959 | 12,773,050,126 | 2.3575 | 65.2146 | 214,071,822 | 938,378,692 | 714 | Medium Risk | 7.74 |
| Telecom | 5077 | 18,910,385,176 | 2.1696 | 55.4231 | 205,262,657 | 1,024,826,562 | 716 | Medium Risk | 11.47 |
| Industrials | 5012 | 15,712,153,595 | 2.1915 | 55.4955 | 196,375,776 | 923,688,660 | 716 | Medium Risk | 9.53 |
| Healthcare | 4982 | 10,047,744,433 | 2.2856 | 53.4153 | 118,215,403 | 567,436,497 | 715 | Medium Risk | 6.09 |
| Consumer | 5043 | 5,049,910,316 | 2.1524 | 60.2770 | 69,044,213 | 325,305,031 | 714 | Medium Risk | 3.06 |
| Retail | 4951 | 3,959,380,564 | 2.2869 | 63.0139 | 59,674,822 | 276,759,900 | 714 | Medium Risk | 2.40 |
#### Analysis
The portfolio represents a diversified cross-sector exposure totaling approximately $165 billion. While the portfolio appears homogenous in terms of credit fundamentals (tight clusters in probability of default (PD), (loss given default (LGD), and credit scores), a deeper analysis reveals significant divergence in risk concentration and actual loss realization across sectors.
<details>
<summary>Click to see/hide full analysis</summary>
    
**1. Portfolio Composition**
The portfolio is heavily concentrated in Financials (18.29%), Real Estate (15.34%), and Utilities (13.38%), which together account for nearly 47% of total exposure. Conversely, Retail (2.40%), Consumer (3.06%), and Healthcare (6.09%) represent the smallest allocations, suggesting a conservative tilt away more volatile or lower-margin sectors.

**2. Risk Uniformity**
All sectors carry a "Medium Risk" rating, and average credit scores are tightly clustered between 714 and 716. This indicates the portfolio is curated to a consistent underwriting standard, with no sectors currently classified as high risk based on initial scoring. However, this uniformity may mask underlying vulnerabilities that become apparent only through loss metrics.

**3. Expected vs. Unexpected Loss Disconnect**
While Financials has the highest expected loss ($316M), it also carries the largest unexpected loss ($1.56B), reflecting its sheer size within the portfolio. However, Technology stands out with a disproportionate risk profile:
* Second-highest PD (2.36%)
* Highest LGD (65.21%) by a significant margin
* Despite being only 7.74% of exposure, it contributes a notable share of risk capital.
* Retail, despite being the smallest sector (2.40%), has the second-highest LGD (63.01%), meaning defaults in this sector are particularly costly when they occur.


**4. Operational Metrics (Leverage & Coverage)**
Across sectors, PDs vary only modestly (2.15%–2.36%), but LGD ranges from 45.64% (Real Estate) to 65.21% (Technology) . This makes LGD the primary driver of loss severity:
*Real Estate and Utilities benefit from tangible collateral, yielding lower LGD and lower expected loss rates.
*Technology and Retail suffer from high LGD due to intangible assets, inventory depreciation, and limited recovery options in distress.
**5. Unexpected Loss Exposure**
Unexpected loss represents the capital required to cover losses beyond expectations at a given confidence level. Financials ($1.56B), Real Estate ($1.22B), and Energy ($1.15B) dominate this metric, reflecting that while these sectors are large and stable on average, they also introduce the greatest tail risk to the portfolio.
</details>
  
### High Risk Loan Identification
To identify large exposures that have already defaulted. The recovery rate indicates how much was recovered, helping to assess loss severity on major defaults.
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

### Vintage Credit Ratings
This is a vintage loan origination analysis tracking loan performance by quarter of origination. 
It shows:
* Loans originated in each quarter from 2015-2023
* Average PD at origination (predicted default probability)
* Early defaults (loans that defaulted quickly, likely within 6-12 months)
* Risk ranking (1 = worst performing vintage, 36 = best)


```
WITH loan_vintages AS (
    SELECT
        loan_id,
        sector,
        ead,
        pd_annual,
        origination_date,
        defaulted,
        default_date,
        CONCAT(DATEPART(YEAR, origination_date), 'Q', DATEPART(QUARTER, origination_date)) AS vintage,
        CASE
            WHEN defaulted = 1 AND DATEDIFF(MONTH, origination_date, default_date) <= 12 THEN 1
            ELSE 0
        END AS early_default_flag
    FROM loan_portfolio
)
SELECT
    vintage,
    COUNT(*) AS loans_originated,
    ROUND(SUM(ead), 0) AS total_originated_ead,
    ROUND(AVG(pd_annual) * 100, 4) AS avg_pd_at_origination,
    SUM(early_default_flag) AS early_defaults,
    ROUND(100.0 * SUM(early_default_flag) / COUNT(*), 2) AS early_default_rate_pct,
    ROW_NUMBER() OVER (ORDER BY (100.0 * SUM(early_default_flag) / COUNT(*)) DESC) AS risk_rank
FROM
    loan_vintages
GROUP BY
    vintage
ORDER BY
    vintage;
```
#### Results
| Vintage  | loans_originated | total_originated_ead | avg_pd_at_origination | early_defaults | early_default_rate_pct | risk_rank |
|----------|------------------|----------------------|------------------------|----------------|------------------------|-----------|
| 2015Q1   | 1415             | 4,428,744,243        | 2.1226                 | 31             | 2.1900                 | 21        |
| 2015Q2   | 1425             | 4,453,419,998        | 2.1569                 | 25             | 1.7500                 | 35        |
| 2015Q3   | 1377             | 4,867,256,283        | 2.4999                 | 37             | 2.6900                 | 11        |
| 2015Q4   | 1368             | 4,684,135,388        | 2.1406                 | 27             | 1.9700                 | 30        |
| 2016Q1   | 1376             | 4,620,480,473        | 2.1165                 | 25             | 1.8200                 | 33        |
| 2016Q2   | 1445             | 5,169,013,899        | 1.9368                 | 28             | 1.9400                 | 31        |
| 2016Q3   | 1325             | 4,164,793,240        | 2.0226                 | 31             | 2.3400                 | 16        |
| 2016Q4   | 1427             | 4,427,666,288        | 2.0313                 | 33             | 2.3100                 | 17        |
| 2017Q1   | 1432             | 4,716,694,668        | 2.3273                 | 41             | 2.8600                 | 8         |
| 2017Q2   | 1398             | 4,785,262,835        | 2.1820                 | 26             | 1.8600                 | 32        |
| 2017Q3   | 1382             | 4,926,027,137        | 1.9790                 | 34             | 2.4600                 | 12        |
| 2017Q4   | 1375             | 4,569,631,163        | 2.1052                 | 21             | 1.5300                 | 36        |
| 2018Q1   | 1427             | 4,499,319,103        | 2.1374                 | 31             | 2.1700                 | 22        |
| 2018Q2   | 1281             | 4,270,859,173        | 2.1593                 | 31             | 2.4200                 | 13        |
| 2018Q3   | 1414             | 4,567,017,911        | 2.0654                 | 30             | 2.1200                 | 23        |
| 2018Q4   | 1383             | 4,570,785,953        | 2.2258                 | 28             | 2.0200                 | 27        |
| 2019Q1   | 1347             | 4,195,121,172        | 2.0538                 | 31             | 2.3000                 | 18        |
| 2019Q2   | 1371             | 4,066,311,206        | 2.3310                 | 173            | 12.6200                | 4         |
| 2019Q3   | 1440             | 4,773,712,491        | 2.2526                 | 234            | 16.2500                | 2         |
| 2019Q4   | 1390             | 4,637,030,808        | 2.1690                 | 246            | 17.7000                | 1         |
| 2020Q1   | 1353             | 4,664,549,522        | 2.6503                 | 196            | 14.4900                | 3         |
| 2020Q2   | 1396             | 5,245,626,542        | 3.4092                 | 44             | 3.1500                 | 6         |
| 2020Q3   | 1408             | 4,737,629,209        | 3.1044                 | 48             | 3.4100                 | 5         |
| 2020Q4   | 1404             | 4,336,001,939        | 3.1418                 | 32             | 2.2800                 | 19        |
| 2021Q1   | 1401             | 4,741,456,787        | 1.9503                 | 28             | 2.0000                 | 28        |
| 2021Q2   | 1404             | 4,808,618,841        | 2.1425                 | 28             | 1.9900                 | 29        |
| 2021Q3   | 1375             | 4,559,418,099        | 2.2502                 | 33             | 2.4000                 | 14        |
| 2021Q4   | 1384             | 4,454,739,014        | 2.2231                 | 39             | 2.8200                 | 9         |
| 2022Q1   | 1437             | 4,797,428,044        | 2.1363                 | 30             | 2.0900                 | 25        |
| 2022Q2   | 1339             | 3,999,094,595        | 2.4174                 | 32             | 2.3900                 | 15        |
| 2022Q3   | 1399             | 4,242,545,889        | 2.2217                 | 39             | 2.7900                 | 10        |
| 2022Q4   | 1396             | 4,617,936,071        | 2.0541                 | 25             | 1.7900                 | 34        |
| 2023Q1   | 1262             | 4,236,132,898        | 2.0252                 | 28             | 2.2200                 | 20        |
| 2023Q2   | 1405             | 4,978,681,979        | 2.2273                 | 41             | 2.9200                 | 7         |
| 2023Q3   | 1418             | 4,712,361,860        | 1.9088                 | 30             | 2.1200                 | 24        |
| 2023Q4   | 1421             | 4,404,725,008        | 1.9768                 | 29             | 2.0400                 | 26        |
#### Analysis
The vintage analysis reveals:
1. Risk models performed well during normal economic conditions (2015-2018, 2021-2023)
2. Risk models catastrophically failed during the COVID-19 transition (2019-2020), underestimating actual defaults by 5-8x
3. The worst vintages are 2019Q4 and 2019Q3—loans originated just before the pandemic
4. Vintages originated during early pandemic (2020Q2) performed better than expected due to stimulus interventions
5. The portfolio has normalized since 2021, but the 2019-2020 vintages remain a significant loss event

### Credit Rating Migration Analysis
Show how credit ratings change over time for the same issuers.
```
WITH rating_migrations AS (
    SELECT
        cr2015.issuer_id,
        cr2015.sector,
        cr2015.to_rating AS rating_2015,
        cr2020.to_rating AS rating_2020,
        cr2015.defaulted AS defaulted_2015,
        cr2020.defaulted AS defaulted_2020,
        CASE 
            WHEN cr2020.notches_moved IS NOT NULL THEN cr2020.notches_moved
            ELSE 0
        END AS net_notches_moved,
        cr2015.upgraded AS upgraded_2015,
        cr2015.downgraded AS downgraded_2015,
        cr2020.upgraded AS upgraded_2020,
        cr2020.downgraded AS downgraded_2020
    FROM credit_ratings cr2015
    LEFT JOIN credit_ratings cr2020 
        ON cr2015.issuer_id = cr2020.issuer_id 
        AND cr2020.year = 2020
    WHERE cr2015.year = 2015
)
SELECT
    sector,
    COUNT(*) AS num_issuers,
    SUM(CAST(upgraded_2015 AS INT) + CAST(upgraded_2020 AS INT)) AS total_upgrades,
    SUM(CAST(downgraded_2015 AS INT) + CAST(downgraded_2020 AS INT)) AS total_downgrades,
    SUM(CAST(defaulted_2015 AS INT) + CAST(defaulted_2020 AS INT)) AS total_defaults,
    ROUND(AVG(net_notches_moved), 2) AS avg_net_notches_moved,
    SUM(
        CASE 
            WHEN COALESCE(rating_2015, 'NR') LIKE 'A%' 
                 AND COALESCE(rating_2020, 'NR') LIKE 'B%' 
            THEN 1 
            ELSE 0 
        END
    ) AS a_to_b_migrations
FROM
    rating_migrations
GROUP BY
    sector
HAVING
    COUNT(*) > 5 
ORDER BY
    total_defaults ASC; 
```
#### Results
| Sector | num_issuers | total_upgrades | total_downgrades | total_defaults | avg_net_notches_moved | a_to_b_migrations |
|--------|-------------|----------------|------------------|----------------|----------------------|-------------------|
| Technology | 177 | 16 | 23 | 3 | 0 | 7 |
| Energy | 209 | 26 | 34 | 5 | 0 | 17 |
| Telecom | 202 | 26 | 28 | 6 | 0 | 12 |
| Financials | 216 | 27 | 21 | 7 | 0 | 13 |
| Real_Estate | 178 | 29 | 31 | 8 | 0 | 10 |
| Healthcare | 221 | 29 | 30 | 8 | 0 | 9 |
| Utilities | 197 | 24 | 29 | 9 | 0 | 10 |
| Consumer | 187 | 28 | 16 | 9 | 0 | 8 |
| Industrials | 222 | 25 | 41 | 11 | 0 | 18 |
| Retail | 191 | 17 | 21 | 11 | 0 | 7 |
#### Analysis

<details>
<summary>Click to hide/see description of column headers</summary>
 <br>num_issuers: The number of distinct corporate entities (borrowers) in each sector.
 <br>total_upgrades / total_downgrades: The number of rating improvements or deteriorations assigned by credit analysts.
 <br>total_defaults: The count of issuers that actually defaulted.
 <br>avg_net_notches_moved: The average net change in rating steps (e.g., from BBB to BB). A value of zero indicates that upgrades and downgrades balanced out on average, even if individual moves occurred.
 <br>a_to_b_migrations: The number of issuers that migrated from the "A" rating category to the "BBB" or lower category (often a key threshold signaling transition from investment grade to non-investment grade).
</details>

When layered alongside the earlier portfolio exposure and loss data, this migration table reveals the forward-looking stress signals that expected loss metrics alone do not capture.

<details>
<summary>Click to hide/see full analysis.</summary>

**1. Industrials: The Highest Stress Sector** 
 <br>Industrials stands out as the most troubled sector. It has the highest number of downgrades (41), the highest number of defaults (11), and the highest volume of severe migrations from A to B (18)—all while having the second-largest issuer base (222). This indicates broad-based deterioration rather than isolated incidents. Combined with its earlier expected loss rate of 1.25% and recovery rate of just 45.5%, Industrials emerges as a clear risk hotspot requiring immediate underwriting scrutiny.

**2. Energy & Telecom: High Migration, Moderate Defaults**
 <br>Energy shows significant stress with 34 downgrades and 17 A-to-B migrations—the second highest in that severe migration category. However, defaults (5) are relatively contained for now. This suggests the sector is pre-default stressed; rating agencies have acted preemptively. Given Energy's large portfolio share (12.7%) and moderate recovery rates (48.95%), these downgrades are a warning of potential future default acceleration if commodity prices remain volatile.
Telecom mirrors this pattern with 28 downgrades and 12 A-to-B migrations, signaling structural pressure in that sector.

**3. Financials & Real Estate: Stability Amid Size**
 <br>Despite being the largest sectors by exposure, Financials and Real Estate show relatively balanced migration activity. Financials actually have more upgrades (27) than downgrades (21)—the only sector with a positive net migration balance. This aligns with their low expected loss rates (1.05% and 1.03% respectively) and high recovery rates (50%+). These sectors are acting as portfolio anchors rather than sources of emerging stress.

**4. Retail & Consumer: Divergent Stories**
 <br>Retail shows high defaults (11) but relatively modest A-to-B migrations (7). This suggests defaults may be concentrated among already-low-rated issuers rather than investment-grade names falling from grace. The earlier data showing Retail's low recovery rate (36.7%) amplifies the impact of these defaults.

Consumer, by contrast, has high defaults (9) but more upgrades than downgrades (28 vs. 16). This indicates a polarized sector—winners are improving while laggards are failing outright, with little middle ground.

**5. Technology: Quiet Deterioration**
 <br>echnology shows 23 downgrades versus only 16 upgrades, with a net negative migration trend. Defaults remain low (3) for now, but the negative momentum is notable given Technology's already poor recovery profile (34.95%) and highest expected loss rate (1.68%). If downgrades continue, future defaults in this sector could carry severe loss severity.
</details>

### Stress Testing & Scenario Analysis
Joins the loan portfolio with the stress scenario data to project losses under different economic conditions
```
WITH scenario_impact AS (
    SELECT
        ms.scenario,
        lp.sector,
        SUM(lp.ead * ms.stressed_pd * ms.stressed_lgd) AS stressed_el,
        SUM(lp.ead * lp.pd_annual * lp.lgd) AS base_el,
        COUNT(*) AS loan_count
    FROM loan_portfolio lp
    INNER JOIN macro_stress_scenarios ms ON lp.sector = ms.sector
    WHERE ms.scenario != 'baseline' -- Exclude baseline for this specific part, or combine via UNION ALL
    GROUP BY ms.scenario, lp.sector
)
SELECT
    'baseline' AS scenario,
    sector,
    ROUND(SUM(ead * pd_annual * lgd), 0) AS expected_loss,
    COUNT(*) AS loan_count
FROM loan_portfolio
GROUP BY sector
UNION ALL
SELECT
    scenario,
    sector,
    ROUND(stressed_el, 0) AS expected_loss,
    loan_count
FROM scenario_impact
ORDER BY expected_loss DESC;
```
| scenario     | sector       | expected_loss | loan_count |
|--------------|--------------|---------------|------------|
| covid_like   | Financials   | 925,065,074   | 5023       |
| covid_like   | Real_Estate  | 752,524,217   | 4862       |
| severe       | Financials   | 698,479,634   | 5023       |
| covid_like   | Energy       | 689,391,082   | 5132       |
| gfc_like     | Financials   | 619,528,399   | 5023       |
| covid_like   | Utilities    | 609,191,303   | 4959       |
| covid_like   | Telecom      | 583,214,160   | 5077       |
| severe       | Real_Estate  | 560,273,649   | 4862       |
| severe       | Energy       | 517,211,424   | 5132       |
| adverse      | Financials   | 494,636,849   | 5023       |
| gfc_like     | Real_Estate  | 493,091,425   | 4862       |
| covid_like   | Technology   | 491,361,238   | 4959       |
| covid_like   | Industrials  | 489,466,010   | 5012       |
| severe       | Utilities    | 462,164,303   | 4959       |
| gfc_like     | Energy       | 457,094,024   | 5132       |
| severe       | Telecom      | 446,804,199   | 5077       |
| gfc_like     | Utilities    | 411,032,626   | 4959       |
| gfc_like     | Telecom      | 399,465,498   | 5077       |
| adverse      | Real_Estate  | 388,675,007   | 4862       |
| severe       | Technology   | 380,046,126   | 4959       |
| mild         | Financials   | 377,432,111   | 5023       |
| severe       | Industrials  | 374,979,971   | 5012       |
| adverse      | Energy       | 362,767,589   | 5132       |
| gfc_like     | Technology   | 341,513,988   | 4959       |
| gfc_like     | Industrials  | 335,250,554   | 5012       |
| adverse      | Utilities    | 329,674,795   | 4959       |
| adverse      | Telecom      | 323,108,170   | 5077       |
| baseline     | Financials   | 316,107,016   | 5023       |
| mild         | Real_Estate  | 290,795,476   | 4862       |
| covid_like   | Healthcare   | 284,383,378   | 4982       |
| adverse      | Technology   | 278,491,282   | 4959       |
| mild         | Energy       | 274,232,491   | 5132       |
| adverse      | Industrials  | 271,172,934   | 5012       |
| baseline     | Real_Estate  | 261,792,357   | 4862       |
| mild         | Utilities    | 253,379,176   | 4959       |
| mild         | Telecom      | 251,428,456   | 5077       |
| baseline     | Energy       | 247,405,532   | 5132       |
| baseline     | Utilities    | 235,836,905   | 4959       |
| severe       | Healthcare   | 222,336,784   | 4982       |
| mild         | Technology   | 219,292,524   | 4959       |
| baseline     | Technology   | 214,071,782   | 4959       |
| mild         | Industrials  | 211,015,196   | 5012       |
| baseline     | Telecom      | 205,262,807   | 5077       |
| gfc_like     | Healthcare   | 200,980,018   | 4982       |
| baseline     | Industrials  | 196,375,701   | 5012       |
| adverse      | Healthcare   | 165,471,747   | 4982       |
| covid_like   | Consumer     | 157,946,921   | 5043       |
| covid_like   | Retail       | 139,764,052   | 4951       |
| mild         | Healthcare   | 132,179,764   | 4982       |
| severe       | Consumer     | 122,879,474   | 5043       |
| baseline     | Healthcare   | 118,215,412   | 4982       |
| gfc_like     | Consumer     | 110,778,298   | 5043       |
| severe       | Retail       | 108,579,162   | 4951       |
| gfc_like     | Retail       | 97,809,566    | 4951       |
| adverse      | Consumer     | 90,809,624    | 5043       |
| adverse      | Retail       | 80,072,979    | 4951       |
| mild         | Consumer     | 72,073,900    | 5043       |
| baseline     | Consumer     | 69,044,201    | 5043       |
| mild         | Retail       | 63,424,641    | 4951       |
| baseline     | Retail       | 59,674,791    | 4951       |
```

#### Analysis
* Financials and Real Estate dominate absolute loss exposure due to their size, but their loss escalation multiples are in line with the portfolio average.
* Energy is the most consistently vulnerable sector, appearing in the top five for loss exposure across all stress scenarios, driven by commodity price sensitivity.
* Technology exhibits scenario-specific vulnerability, performing worse under gfc_like (credit-driven) than under covid_like (operational-driven), reflecting its high LGD and reliance on intangible assets.
* Retail and Consumer are small but severe, meaning their absolute losses are low due to limited exposure, but their loss rates relative to size are high, warranting close monitoring.
* Healthcare is uniquely sensitive to pandemic-type shocks, with covid_like being its worst-case scenario by a significant margin.
* Utilities and Telecom act as portfolio stabilizers, maintaining lower and more stable loss projections across all scenarios.
