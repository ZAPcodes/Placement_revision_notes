# Data Interpretation

Study notes for JioStar / placement aptitude (CAT-minus-English style).

**Core idea:** DI is not a new subject. It is **percentage, ratio, average, and alligation sitting inside a table**. High scorers do **not** compute every cell. They read the **questions first**, then touch only the numbers that matter.

---

## 0. The DI weapons

**Weapon 1 — Questions first, table second**  
About 90 seconds: read all 4–5 questions. Circle which years, rows, and totals you need. Ignore the rest of the chart.

**Weapon 2 — Don’t compute the leftover**  
If Hindi = 40 and Total = 200, Hindi% = 20. You never need Tamil + English for that question.

**Weapon 3 — Ratio before actual**  
If every bar is “in thousands” and the thousands cancel, drop the zeros. 40,000 : 25,000 = 8 : 5.

**Weapon 4 — Options decide the precision**  
Options 12%, 18%, 25%, 40% → a 2-second fraction is enough.  
Options 18.2%, 18.6%, 19.1% → then calculate.

**Weapon 5 — Same base, always**  
“20% of sports viewers” and “20% of all viewers” are different people. Write in the margin: **of what?**

**Weapon 6 — Pie: 360° = 100% = total value**  
1% = 3.6°.  
Sector value = (angle / 360) × total = (% / 100) × total.

---

## 1. What every DI set is secretly asking

Almost every question is one of these eight:

1. **Read a cell** — “What is Tamil Movies?”  
2. **Row / column total** — add  
3. **Ratio** — A : B  
4. **Percentage share** — part / whole × 100  
5. **% change** — (new − old) / old × 100  
6. **Average** — sum / n (not average of averages)  
7. **Difference / how much more**  
8. **Inference** — which statement **must** be true

If you can do those eight on a **table**, you can do bar, line, pie, and caselet. They are the same table wearing makeup.

---

## 2. Type 1 — Table (the mother of all DI)

### 2.1 What to look for

- Row totals and column totals (they must agree)  
- A **grand total** (bottom-right)  
- Units: crore, %, index, “per 1000”  
- Whether a column is **already a %** (then you **cannot** add those %s across unlike bases)

### 2.2 Filling a missing-value table

Fill the cell where **the rest of that row or column is known**. Never guess two missing cells at once.

Inconsistent example (your check should catch this):

```
        A     B     C    Total
X      20     ?    30     80
Y       ?    25     ?     70
Total  50    60    40    150
```

- X-row: 20 + ? + 30 = 80 ⇒ X-B = 30  
- A-column: 20 + ? = 50 ⇒ Y-A = 30  
- C-column: 30 + ? = 40 ⇒ Y-C = 10  
- Check Y: 30 + 25 + 10 = 65, but Y-total is 70 → **inconsistent**. In a real paper the table is consistent. If the check fails, you added wrong.

**Consistent table** (use this):

```
        A     B     C    Total
X      20    30    30     80
Y      30    30    10     70
Total  50    60    40    150
```

If X-B, Y-A, Y-C were blank, the same three steps fill them. Then Y-row 30+30+10 = 70 confirms.

### 2.3 Set A — Watch hours (crore)

| Language | Sports | Movies | Shows | Total |
|----------|--------|--------|-------|-------|
| Hindi    | 40     | 30     | 20    | 90    |
| Tamil    | 15     | 25     | 10    | 50    |
| English  | 20     | 15     | 25    | 60    |
| **Total**| **75** | **70** | **55**| **200**|

**Q1.** Hindi Sports as % of all watch hours?  
40 / 200 = **20%**. Direct cell / grand total.

**Q2.** Movies : Shows?  
70 : 55 = **14 : 11**.

**Q3.** Which language has the highest share of *its own* hours in Sports?  
- Hindi 40/90 ≈ 44.4%  
- Tamil 15/50 = 30%  
- English 20/60 ≈ 33.3%  
**Hindi.**  

Trap: highest *Sports hours* is also Hindi (40), but the question was **share of own language**. For other rows the ranking can flip vs raw hours.

**Q4.** If Sports grows 20% and others unchanged, new grand total?  
Sports 75 × 1.2 = 90. Others 70 + 55 = 125. Total **215**.  
Sports share now = 90/215 ≈ 41.9% (was 75/200 = 37.5%).

**Q5.** Average watch hours per language?  
200 / 3. That is the average of the three language totals.  

Trap: averaging the three “category averages” is safe here only because each language has the same number of categories. If one language had two categories and another had four, averaging averages would be wrong.

**More from Set A**

- English as % of total hours: 60/200 = **30%**  
- Sports is what % more than Shows: (75 − 55) / **55** = 20/55 ≈ **36.36%** (not 20/75)  
- Tamil Movies as % of all Movies: 25/70 ≈ **35.71%**  
- Drop English: remaining 140, Hindi 90 → **64.29%** (was 45%)

---

## 3. Type 2 — Bar / line (time series)

### 3.1 What to look for

- **YoY %** = (this − last) / last  
- A peak is not “highest growth.” Highest **value** ≠ highest **% increase**  
- Line chart: slope = change, not the height  
- If two lines, read the **gap** (difference) and **crossover** (who overtakes)

**Successive %:** +20% then −10% is **not** +10%.  
Net = 1.2 × 0.9 = 1.08 → **+8%**.

### 3.2 Set B — DAU (lakh)

| Year | 2018 | 2019 | 2020 | 2021 | 2022 |
|------|------|------|------|------|------|
| DAU  | 40   | 48   | 36   | 54   | 60   |

**Q6.** YoY growth 2018 → 2019?  
(48 − 40) / 40 = **20%**.

**Q7.** Maximum **%** increase?  
- 2018→19: +20%  
- 2019→20: (36 − 48) / 48 = −25%  
- 2020→21: (54 − 36) / 36 = **50%**  
- 2021→22: 6/54 ≈ 11.1%  

Max increase **2020→21**, even though 2022 is the highest DAU. **Classic trap.**

**Q8.** 2022 as % of 2018?  
60/40 = 150% of 2018, i.e. **+50% vs 2018**.  
“50% of 2018” would mean 20. Language: “50% **more than**” vs “50% **of**.”

**Q9.** Average DAU?  
(40+48+36+54+60) / 5 = 238/5 = **47.6**.  

If they ask average **growth rate**, that is **not** (60 − 40)/5 (that is average *absolute* increase of 4 per year). CAGR is different again. In most OAs, “average” = arithmetic mean of the listed numbers unless they say CAGR.

**More from Set B**

- 2022 is what % more than 2020: (60 − 36) / 36 = **66.67%**  
- DAU fell in **2019→2020** (48→36)  
- 2023 is +10% on 2022 → 60 × 1.1 = **66**  
- Average of 2018 and 2022 is 50; five-year average is 47.6 → **not equal**

---

## 4. Type 3 — Pie chart

### 4.1 What to look for

- Is the pie in **%**, **degrees**, or **actual rupees**?  
- Two pies (2019 vs 2020): the same sector % can hide a **bigger pie**. 30% of 100 vs 25% of 200: the 25% is **more rupees**.  
- You **cannot** add pie % from two different pies.

### 4.2 Set C — Revenue pie

Total revenue = ₹2400 crore. Sectors in **degrees**:

| Ads | Sub | Pay-per | Other |
|-----|-----|---------|-------|
| 90° | 108° | 72°    | 90°   |

Check: 90 + 108 + 72 + 90 = 360.

Value of a sector = (θ / 360) × 2400.  
90/360 = 25% of 2400 = **600**.

**Q10.** Subscription revenue?  
108/360 = 30% of 2400 = **720**.

**Q11.** Ads are how much % more than Pay-per?  
Ads 600, Pay-per 72/360 = 20% = 480.  
(600 − 480) / **480** = 120/480 = **25% more**.  

Trap: (600 − 480) / 600 = 20% — that is “pay-per is 20% **less than** ads.” Base flipped.

**Q12.** Next year total becomes 3000, Ads still 90°. Ads value?  
90° is still 25%, of **3000** = **750**.  
Unchanged angle ≠ unchanged rupees.

**More from Set C**

- Other + Ads = 180° = 50% of 2400 = **1200**  
- Pay-per : Subscription = 480 : 720 = **2 : 3**

---

## 5. Type 4 — Two charts together

Pie of **share** + table of **totals**, or two bars (volume vs price).

**Rule:** % × **that year’s** total = actual.  
Never multiply 2022’s % by 2021’s total.

### 5.1 Set D — Users and paid share

| Year | Total users (crore) |
|------|---------------------|
| 2021 | 20 |
| 2022 | 25 |

Share of **Paid** users: 2021 = 20%, 2022 = 24%.

**Q13.** Paid users in 2022?  
0.24 × 25 = **6 crore**.

**Q14.** Growth in **Paid users** 2021→22?  
2021 paid = 0.20 × 20 = 4. 2022 = 6. Growth = 2/4 = **50%**.  

Paid *share* only went 20 → 24 (**+4 percentage points**, which is +20% *relative*).  
Paid *headcount* grew 50% because the pie also grew.

**Write this down for an analytics OA**

> Share ↑ and base ↑ ⇒ actual explodes.  
> Share ↓ but base ↑ a lot ⇒ actual can still ↑.

**Percentage points ≠ percent.**  
20% to 24% is +4 pp, and +20% relative to 20.

---

## 6. Type 5 — Caselet (paragraph, no table)

**First job: draw the table yourself.** Do not solve from the English.

### 6.1 Set E — Production and sales

A factory makes three models P, Q, R.

- Production: P = 40%, Q = 35%, rest R. Total production = 8000 units.  
- Sales: P sold 90% of its production, Q sold 80%, R sold 100%.  
- Unsold is inventory.

**Table (30 seconds)**

|   | Prod | Sold | Unsold |
|---|------|------|--------|
| P | 0.4 × 8000 = 3200 | 0.9 × 3200 = 2880 | 320 |
| Q | 2800 | 2240 | 560 |
| R | 1600 | 1600 | 0 |
| Tot | 8000 | 6720 | 880 |

**Q15.** Unsold as % of production? 880/8000 = **11%**.  

**Q16.** Which model has highest unsold **share of its own production**?  
P 10%, Q 20%, R 0% → **Q**.  

**Q17.** Do not mix months if unsold P is sold next month.

### 6.2 Set F — Survey / Venn in DI clothing

250 users. 60% watched Sports. Of Sports watchers, 40% also watched Movies. 30% of **all** users watched Movies.

- Sports = 150  
- Sports ∩ Movies = 0.4 × 150 = **60**  
- Movies = 0.3 × 250 = 75  
- Only Movies = 75 − 60 = **15**  
- Only Sports = 150 − 60 = **90**  
- Neither = 250 − (90 + 60 + 15) = **85**

**Q18.** Movies but not Sports? **15**.  
**Q19.** % who watched at least one of the two? (250 − 85) / 250 = **66%**.  
**Q20.** % who watched neither? 85/250 = **34%**.  
Both = **60**.

**Trap:** “40% also watched Movies” is 40% **of Sports**, not of 250. Of 250 that would be 100 — wrong.

---

## 7. Type 6 — Index numbers (“2015 = 100”)

Index 125 in 2018 vs 100 in 2015 means **+25% vs 2015**, not vs last year.

If 2015 = 100, 2018 = 125, 2019 = 150:

- 2019 vs 2015: +50%  
- 2019 vs 2018: 25/125 = **20%**, not 25%

When they give only indices, actual rupees cancel. Growth = ratio of indices.

---

## 8. Calculation speed

### 8.1 Fraction → % table

| Fraction | % | Fraction | % |
|----------|---|----------|---|
| 1/2 | 50 | 1/6 | 16.67 |
| 1/3 | 33.33 | 1/7 | 14.28 |
| 2/3 | 66.67 | 1/8 | 12.5 |
| 1/4 | 25 | 3/8 | 37.5 |
| 3/4 | 75 | 5/8 | 62.5 |
| 1/5 | 20 | 2/5 | 40 |

40/200 = 1/5 = 20%. Do not long-divide.

### 8.2 % of a number

- 10% = move the decimal  
- 5% = half of 10%  
- 1% = 1/10 of 10%  
- 15% = 10% + 5%  
- 35% of 80 = 80% of 35 = 28 (swap)

### 8.3 Approximation when options are far

32.4% of 487.  
30% of 490 ≈ 147, 2.4% of 490 ≈ 12, total ~159.  
Options 120, 158, 200, 240 → **158**. Stop.

### 8.4 Alligation inside DI

Two plans: Free ARPU ₹30, Paid ARPU ₹180, overall ₹60.

```
     30                    180
              60
    120                     30      →  4 : 1
```

Free : Paid users = **4 : 1**. Same lever as the arithmetic class.

---

## 9. Traps (the real test)

| Trap | Wrong | Right |
|------|--------|--------|
| % more vs % of | “A is 25% of B” vs “25% more than B” | more than = ×1.25 |
| Wrong base | (new − old) / **new** | always / **old** |
| % points vs % | 20% → 24% is “+4%” | +4 **pp**, or +20% relative |
| Average of % | (20% + 40%) / 2 = 30% overall | only if **equal bases** |
| Two pies | 30% + 30% = 60% of something | convert to actuals first |
| Highest bar = fastest growth | 2022 is tallest | check **%** YoY |
| Adding growth rates | +20 then −10 = +10 | 1.2 × 0.9 = +8% |
| Caselet “of which” | 40% of all | 40% of the **previous group** |
| Inventory | treat unsold as “loss” | Production − sales = unsold unless they say loss |
| Index | treat 125 as rupees | only a **scale** |

**Average of averages:** 2 cities, averages 20 and 40. Combined is 30 **only if** equal n. If 90 people at 20 and 10 people at 40, combined = (1800 + 400) / 100 = **22**. DI hides this as two rows with different totals.

**Funnel tables (JioStar flavour):**  
CTR = clicks / **impressions**.  
Play rate might be plays / clicks **or** plays / impressions. Read the footnote. Never assume.

---

## 10. How to sit a 4-question DI set (~8 minutes)

1. **0:00–0:40** — skim questions. Mark “easy direct” vs “inference.”  
2. **0:40–1:30** — totals, units, whether numbers are % or actual. Add a grand total if missing.  
3. **1:30–6:00** — easy questions first (one cell, simple ratio).  
4. **6:00–8:00** — one calculation-heavy or “which is true.”  
5. If a question needs many multiplications and options are close, skip and return.

**Order inside the set:** one cell → ratio of two → % of total → YoY → “cannot be determined.”

**Cannot be determined** is often correct when they ask actuals but gave **only %** with **no total**.

---

## 11. JioStar dressing (same maths, OTT nouns)

| Set type | How it may appear |
|----------|-------------------|
| Table (Set A) | Watch hours by language × genre |
| Bar/line (Set B) | DAU / MAU by year |
| Pie (Set C) | Revenue: ads / subscription / pay-per |
| Two charts (Set D) | Paid share % × total users |
| Caselet F | Sports vs Movies overlap |
| Funnel table | Impression → click → play → complete |

---

## 12. Mixed drill (12 minutes) — cover answers

Use Set A (total 200), Set B (DAU), Set C (pie 2400), Set F (250 users).

1. English as % of total hours? **30%**  
2. Sports is what % more than Shows? **36.36%** (20/55)  
3. Tamil Movies as % of all Movies? **35.71%** (25/70)  
4. Drop English: Hindi’s share of remaining? **64.29%** (90/140)  
5. DAU 2022 % more than 2020? **66.67%**  
6. Year DAU fell? **2019→2020**  
7. 2023 = +10% on 2022? **66**  
8. Avg(2018, 2022) = 5-year avg? **No** (50 vs 47.6)  
9. Other + Ads together? **1200**  
10. Pay-per as fraction of Subscription? **2 : 3**  
11. Watched both? **60**  
12. % watched neither? **34%**

If you miss 2 or 5, the error is **wrong base**, not the table. That is the whole DI course in one mistake.
