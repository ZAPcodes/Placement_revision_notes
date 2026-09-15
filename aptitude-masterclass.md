# Aptitude Masterclass 
> CAT-teacher style: every concept → core idea → shortcut bank → real questions.
> Don't memorise formulas blindly. Understand the **one pattern** each topic is really testing.

---

## MASTER PRINCIPLE BEFORE ANYTHING

Every aptitude problem reduces to one of two moves:

1. **Convert everything to a common unit** (rate, %, multiplier, fraction)
2. **Use alligation** — any weighted average problem is an alligation problem in disguise

Keep these two in mind and 80% of questions become mechanical.

---

# MODULE 1 — PERCENTAGES

## Core Idea
Percentage = fraction with denominator 100. The real power is the **multiplier approach** — express every change as a single multiplication.

| Operation | Multiplier |
|---|---|
| 20% increase | × 1.20 |
| 20% decrease | × 0.80 |
| 7.5% increase | × 1.075 |
| 33.33% decrease | × 2/3 |

## Formula Bank

```
% Change = (New − Old) / Old × 100

Successive changes a% then b%:
  Net change = a + b + ab/100
  (use sign: + for increase, − for decrease)

If A is P% more than B:
  B is P/(100+P) × 100 % LESS than A
```

## The Fraction Table — Memorise Cold

| Fraction | % | Fraction | % |
|---|---|---|---|
| 1/2 | 50% | 1/9 | 11.11% |
| 1/3 | 33.33% | 1/11 | 9.09% |
| 1/4 | 25% | 1/12 | 8.33% |
| 1/5 | 20% | 1/13 | 7.69% |
| 1/6 | 16.67% | 1/14 | 7.14% |
| 1/7 | 14.28% | 1/16 | 6.25% |
| 1/8 | 12.5% | 1/19 | 5.26% |

## Tricks

**Trick 1: x% of y = y% of x**
Instead of computing 16% of 25 → compute 25% of 16 = 4. Done in 2 seconds.
Instead of 36% of 75 → 75% of 36 = 27.

**Trick 2: Successive change formula**
Two changes: 20% up then 10% down:
Net = 20 + (−10) + 20×(−10)/100 = 10 − 2 = **+8%**
Same % up then down is ALWAYS a loss: net = −(x²/100)%

**Trick 3: Percentage points ≠ percentages**
Vote share went from 40% to 50%:
→ 10 percentage points change, but 10/40 × 100 = 25% increase in share

**Trick 4: Base flip**
A is 25% more than B → B is 20% less than A
A is 50% more than B → B is 33.33% less than A
Formula: if A is P% more than B → B is P/(100+P) × 100 % less than A

**Trick 5: Chain multiplier for complex % changes**
Original → ×(1+a/100) × (1+b/100) × ... → Final
No need to compute intermediate values.

## Questions

**Q1.** A number is increased by 20% then decreased by 20%. Net change?
```
Net = 20 + (−20) + 20×(−20)/100 = 0 − 4 = −4%
Answer: 4% LOSS. Always a loss when same % up and down.
```

**Q2.** JioCinema's subscribers grew from 80 lakh to 100 lakh. Free users were
420 lakh, total users constant at 500 lakh. By what % did free users fall?
```
Premium rose 20 lakh → free users fell 20 lakh (from 420 to 400 lakh)
% fall = 20/420 × 100 = 100/21 ≈ 4.76%
```

**Q3.** A is 25% of B. B is 80% of C. What % is A of C?
```
A = 0.25B and B = 0.8C
→ A = 0.25 × 0.8 × C = 0.2C = 20% of C
Multiplier chain: 0.25 × 0.80 = 0.20
```

**Q4.** 10% reduction in price lets a person buy 5 kg more for ₹900. Original price?
```
Original price = P per kg. New price = 0.9P.
900/(0.9P) − 900/P = 5
(1000 − 900)/P = 5
P = ₹20 per kg
```

**Q5.** A shopkeeper marks up 40% above cost and gives 25% discount. Profit%?
```
Net multiplier = 1.40 × 0.75 = 1.05 → 5% profit
No intermediate computation needed.
```

**Q6.** If 120% of x = 80% of y, find x:y.
```
1.2x = 0.8y → x/y = 0.8/1.2 = 2/3 → x:y = 2:3
```

---

# MODULE 2 — PROFIT–LOSS, DISCOUNTS, SUCCESSIVE CHANGE

## Core Idea
Everything revolves around CP (Cost Price) as the base. Train yourself to always compute the multiplier relative to CP.

## Formula Bank

```
Profit% = Profit/CP × 100        Loss% = Loss/CP × 100
SP = CP × (100 + P%)/100          CP = SP × 100/(100 + P%)
SP = CP × (100 − L%)/100          CP = SP × 100/(100 − L%)

Discount% = (MP − SP)/MP × 100
SP = MP × (1 − d/100)

Effective single discount from two successive discounts d1, d2:
  = d1 + d2 − d1×d2/100
```

## Tricks

**Trick 1: Same SP, x% profit and x% loss → always a loss**
```
Loss% = x²/100
x = 10% → Loss = 1%
x = 20% → Loss = 4%
x = 25% → Loss = 6.25%
```
Why: Total CP = SP/1.x + SP/0.(1-x). Total SP = 2×SP. Do the algebra once and memorise the pattern.

**Trick 2: False weights (classic trap)**
Shopkeeper uses W grams instead of (W + e) grams:
```
Gain% = e/W × 100
```
Example: Uses 960g instead of 1kg:
Gain% = 40/960 × 100 = 4.17%

**Trick 3: When markup and discount are both given, always use the multiplier**
```
CP → × (1 + markup%) = MP → × (1 − discount%) = SP
Profit% = SP/CP − 1 = (1+markup%)(1−discount%) − 1
```

**Trick 4: Buy at A% profit, sell at B% profit on ORIGINAL CP**
```
Net SP = CP_original × (1 + A/100) × (1 + B/100)
```

**Trick 5: When CP is not given, assign CP = 100 and work numerically**
This turns every % into an absolute number and eliminates variable confusion.

## Questions

**Q1.** Two articles sold at ₹1200 each — 20% profit on one, 20% loss on other. Overall?
```
Loss% = 20²/100 = 4%
CP₁ = 1200/1.2 = 1000
CP₂ = 1200/0.8 = 1500
Total CP = 2500, Total SP = 2400 → Loss = ₹100
Loss% = 100/2500 × 100 = 4% ✓
```

**Q2.** Markup 60% above CP, two successive discounts 10% and 25%. Profit%?
```
SP = CP × 1.6 × 0.9 × 0.75 = CP × 1.08 → 8% profit
```

**Q3.** Dishonest dealer uses 950g instead of 1kg. Sells at cost price. Gain%?
```
He gives 950g but charges for 1000g.
Gain on 950g = 50g
Gain% = 50/950 × 100 = 5.26% ≈ 100/19 %
(use the fraction table: 1/19 = 5.26%)
```

**Q4.** After 20% discount on MP, shopkeeper still earns 20% profit. CP = ₹600. Find MP.
```
SP = 600 × 1.20 = ₹720
MP × 0.80 = 720 → MP = ₹900
```

**Q5.** Ramesh buys at 10% discount, sells at 20% profit on what he paid. % over original?
```
He pays = 0.9 × OP
He sells = 1.20 × 0.9 × OP = 1.08 × OP → 8% above original price
```

**Q6.** A trader marks price 25% above CP and gives a discount of d%. He makes no profit.
Find d.
```
1.25 × (1 − d/100) = 1
1 − d/100 = 0.8
d/100 = 0.2 → d = 20%
```

---

# MODULE 3 — MIXTURES AND ALLIGATION

## Core Idea
Alligation solves ANY weighted average problem — the ratio in which two ingredients
(with values c1 and c2) must be mixed to achieve a target mean m.

## The Cross Rule (Universal)

```
         c1                c2
           \              /
            \            /
             \          /
                  m
             /          \
            /            \
(c2 − m)               (m − c1)

Ratio of quantity at c1 : quantity at c2 = (c2 − m) : (m − c1)
```

**Important:** c1 < m < c2 must hold. If not, swap.

## Tricks

**Trick 1: Alligation works for ANYTHING**
Marks, salaries, speeds, concentrations, ages, ARPUs — if two groups have two different averages and you want the combined average or the ratio, use alligation.

**Trick 2: Repeated dilution formula**
A vessel of volume V has concentration C. Replace r litres with water k times:
```
Final concentration = C × (1 − r/V)^k
```

**Trick 3: When mixing two mixtures (not pure ingredients)**
Mixture 1: Volume V₁, concentration c₁ of component A
Mixture 2: Volume V₂, concentration c₂ of component A
```
Final concentration = (V₁c₁ + V₂c₂) / (V₁ + V₂)
```
If they want ratio V₁:V₂ for a target concentration → use alligation on c₁ and c₂.

**Trick 4: "How much to add" questions**
To dilute a mixture of concentration c down to target t by adding pure water:
Let current volume = V.
```
New volume needed = V × c/t
Water to add = V(c/t − 1)
```

## Questions

**Q1.** Milk at ₹30/L mixed with milk at ₹40/L to get ₹36/L mixture. Ratio?
```
Alligation:
30       40
    \  /
    36
   /  \
 4     6
Ratio cheaper:dearer = 4:6 = 2:3
```

**Q2 (Analytics context).** Premium users ARPU = ₹150, free users ARPU = ₹20.
Overall ARPU = ₹50. Ratio of free:premium users?
```
20             150
      \     /
      50
     /     \
(150−50)  (50−20)
   100        30
Ratio free:premium = 100:30 = 10:3
```

**Q3.** 40L vessel full of milk. 8L removed and replaced with water. Done twice.
Milk remaining?
```
After each replacement: factor = (1 − 8/40) = 4/5
After 2 replacements: 40 × (4/5)² = 40 × 16/25 = 25.6 litres
```

**Q4.** Mixture of acid:water = 4:1 in 20L. Water to add to make it 2:3?
```
Current: acid = 16L, water = 4L.
Need acid:water = 2:3. Acid stays at 16L.
16/(16+water_new) = 2/5 → water_new = 24L
Water to add = 24 − 4 = 20 litres
```

**Q5.** 25 kg sugar at ₹20/kg mixed with 15 kg at ₹30/kg. Selling price for 25% profit?
```
Avg CP = (25×20 + 15×30)/(40) = (500+450)/40 = ₹23.75/kg
SP = 23.75 × 1.25 = ₹29.69/kg
```

**Q6.** A, B, C three solutions with concentrations 20%, 30%, 40%. Mixed in ratio 3:2:1.
Final concentration?
```
= (3×20 + 2×30 + 1×40) / 6
= (60 + 60 + 40)/6 = 160/6 = 26.67%
```

---

# MODULE 4 — TIME–WORK AND PIPES

## Core Idea
Work is a RATE problem. Rate = Work/Time. All combined-work questions reduce to:
**add the rates, then find time.**

The LCM method makes this entirely arithmetic — no fractions.

## The LCM Method (Always Use This)

```
Step 1: Take LCM of all "days" mentioned — this is your TOTAL WORK in units.
Step 2: Each person's daily work = LCM / their days.
Step 3: Combined daily work = sum of individual daily works.
Step 4: Time together = Total Work / Combined daily work.
```

Example: A takes 12 days, B takes 18 days.
LCM(12,18) = 36. A = 3 units/day, B = 2 units/day.
Together = 5 units/day → time = 36/5 = 7.2 days.

## Formula Bank

```
A and B together: ab/(a+b) days

If A is n times as efficient as B:
  A takes (1/n)th of B's time.

Man-Days: M × D = constant total work
  If M men take D days → M' men take D' = MD/M' days
```

## Tricks

**Trick 1: LCM method avoids fractions entirely**
Prefer this over the fraction formula for 3+ workers.

**Trick 2: Negative work**
Person who damages, pipe that drains, or person who works AGAINST the team = negative rate.
Subtract from the combined rate.

**Trick 3: Leaving/Joining midway**
Compute total work. Subtract work done in the joint/partial period. Remaining → single worker.

**Trick 4: Pipes**
Filling pipes → positive rates. Emptying pipes → negative rates.
Tank fills when net rate > 0.

**Trick 5: Efficiency**
"A is 50% more efficient than B" → if B = 1 unit/day, A = 1.5 units/day.
A takes 2/3 of B's time.

## Questions

**Q1.** A does job in 12 days, B in 18 days. Work together 4 days, A leaves. How long for B to finish?
```
LCM(12,18) = 36. A = 3/day, B = 2/day.
Work in 4 days together = 4 × 5 = 20 units.
Remaining = 36 − 20 = 16 units.
B alone: 16/2 = 8 more days.
```

**Q2.** Pipes fill tank in 10h and 15h. Leak empties in 20h. All open. Time to fill?
```
LCM(10,15,20) = 60. Rates: 6 + 4 − 3 = 7 units/hour.
Time = 60/7 = 8 4/7 hours.
```

**Q3.** A does 1/3 of job in 5 days. Calls B. They finish rest in 3 days. B alone?
```
A's rate: 1/3 of job in 5 days → full job in 15 days.
Remaining = 2/3 job in 3 days → combined rate = (2/3)/3 = 2/9 per day.
B's rate = 2/9 − 1/15 = 10/45 − 3/45 = 7/45.
B alone = 45/7 ≈ 6.4 days.
```

**Q4 (Man-Days).** 8 men do work in 20 days. 16 women do it in 12 days.
4 men + 12 women work 10 days. Men to finish in 8 more days?
```
Total work = 8 × 20 = 160 man-units.
1 woman's rate relative to 1 man:
  16 women × 12 = 192 woman-days = 160 man-days
  → 1 woman = 160/192 = 5/6 man-unit/day.

Work done = (4 + 12 × 5/6) × 10 = (4 + 10) × 10 = 140 units.
Remaining = 160 − 140 = 20 units.
Men needed in 8 days = 20/8 = 2.5 → 3 men.
```

**Q5 (Efficiency).** A is 50% more efficient than B. Together: 10 days. A alone?
```
Let B = b units/day. A = 1.5b. Together = 2.5b = 1/10 → b = 1/25.
A alone: 1/(1.5/25) = 25/1.5 = 50/3 ≈ 16.67 days.
```

---

# MODULE 5 — TIME–SPEED–DISTANCE & RELATIVE SPEED

## Core Idea
D = S × T. Everything else is a restatement of this. The key insight is always:
**what is the relevant distance and what is the relevant speed (often a relative speed)?**

## Formula Bank

```
D = S × T (all in consistent units)

Unit conversion:
  km/h → m/s: multiply by 5/18
  m/s → km/h: multiply by 18/5

Average speed for EQUAL DISTANCES: 2ab/(a+b) [harmonic mean]
Average speed for EQUAL TIMES: (a+b)/2 [arithmetic mean]
```

## Relative Speed

```
Same direction:    |S₁ − S₂|
Opposite direction: S₁ + S₂
```

## Trains

```
Cross a pole/man (stationary):  t = L_train / S_train
Cross a platform:               t = (L_train + L_platform) / S_train
Two trains cross each other:    t = (L₁ + L₂) / relative speed
```

## Boats in Streams

```
Downstream speed = B + R
Upstream speed = B − R
Boat speed = (Down + Up)/2
River speed = (Down − Up)/2
```

## Tricks

**Trick 1: Always identify what "crosses" means**
Object disappears from view after its FULL LENGTH has passed the reference point.
So the relevant distance = sum of lengths, not just one.

**Trick 2: Meeting problems**
Two people start simultaneously from A and B:
- First meeting: they cover a COMBINED distance of AB.
- Their individual distances are proportional to their speeds.

**Trick 3: Circular track meeting**
- Opposite directions: first meet after AB_track / (S₁+S₂) time.
- Same direction: first meet after AB_track / |S₁−S₂| time.
- Meet at START (same direction): LCM of their individual lap times.

**Trick 4: Chase problems**
Head start = H. Speed of chaser = S₁. Speed of fugitive = S₂ (S₁ > S₂).
```
Time to catch = H / (S₁ − S₂)
```

**Trick 5: Escalator/Conveyor belt**
Moving stairs: staircase has its own speed. Same direction = add, opposite = subtract.
Treat like boats in streams.

## Questions

**Q1.** Train 150m long crosses a pole in 15s. Crosses an opposite train (100m long) in 12s.
Speed of second train?
```
Speed of first train = 150/15 = 10 m/s.
Relative speed (opposite) = (150+100)/12 = 250/12 m/s.
Speed of second = 250/12 − 10 = 130/12 ≈ 10.83 m/s = 39 km/h.
```

**Q2.** Half journey at 30 km/h, other half at 70 km/h. Average speed?
```
Equal distances → use harmonic mean: 2×30×70/(30+70) = 4200/100 = 42 km/h.
```

**Q3.** A and B 300 km apart, start toward each other. A at 60, B at 90. Distance from A at meeting?
```
Combined speed = 150. Time to meet = 300/150 = 2h.
A covers 60 × 2 = 120 km from A.
```

**Q4.** Boat: 30 km upstream in 3h, 30 km downstream in 2h. Time for 60 km downstream?
```
Upstream = 10 km/h. Downstream = 15 km/h.
Time = 60/15 = 4 hours.
(No need to find B and R separately.)
```

**Q5 (Chase).** Thief steals a car at 1 PM, drives at 60 km/h. Police start at 2 PM at 80 km/h. Catch time?
```
Head start = 1h × 60 = 60 km.
Relative speed = 80 − 60 = 20 km/h.
Time = 60/20 = 3 hours after police start = 5 PM.
```

**Q6.** A train passes a man in 8 seconds and a 264m platform in 20 seconds. Length and speed of train?
```
Let length = L, speed = S.
L/S = 8 ... (i)
(L+264)/S = 20 ... (ii)
(ii) − (i): 264/S = 12 → S = 22 m/s = 79.2 km/h
L = 8 × 22 = 176m.
```

---

# MODULE 6 — SIMPLE & COMPOUND INTEREST

## Core Idea
SI: interest is always on original principal. CI: interest compounds — you earn interest on interest.
The gap between them grows with time and rate.

## Formula Bank

```
Simple Interest:
  SI = PRT/100
  Amount = P(1 + RT/100)

Compound Interest:
  Amount = P(1 + R/100)^n
  CI = A − P

For 2 years, CI expressed via SI:
  CI = SI + SI²/(P×... )
  Simpler: CI − SI (for 2 years) = P(R/100)²

For 3 years: CI − SI = P(R/100)²(3 + R/100)

Half-yearly compounding:
  Use R/2 and 2n in the CI formula.
```

## Tricks

**Trick 1: The 2-year difference formula is your fastest tool**
CI − SI (2 years) = P(R/100)²
If you know any two of the three values, you can find the third instantly.

**Trick 2: Rule of 72**
Money doubles in approximately 72/R years at compound interest.
R = 8% → doubles in 9 years.
R = 12% → doubles in 6 years.

**Trick 3: Successive amounts in CI form a Geometric Progression**
A₁ = P(1+r), A₂ = P(1+r)², A₃ = P(1+r)³...
Ratio between consecutive years = (1+r) = constant.
So if you know two consecutive amounts, you know the rate.

**Trick 4: "Doubles in n years at SI" questions**
If doubles at SI: SI = P in n years → P×R×n/100 = P → R = 100/n%

**Trick 5: "Trebles" or "becomes k times" at SI**
Amount = kP → SI = (k-1)P
(k-1)P = PRT/100 → T = (k-1)×100/R

## Questions

**Q1.** Difference between CI and SI on ₹8000 for 2 years = ₹20. Find rate.
```
P(R/100)² = 20
8000(R/100)² = 20
(R/100)² = 20/8000 = 1/400
R/100 = 1/20 → R = 5%
```

**Q2.** A sum doubles in 5 years at SI. How many years to triple?
```
Doubles → SI = P in 5 years → R = 100/5 = 20%
Triple → SI = 2P → 2P = P×20×T/100 → T = 10 years
(Linear relationship: doubles in 5, triples in 10, quadruples in 15...)
```

**Q3.** CI on ₹10,000 at 10% p.a. compounded half-yearly for 1 year?
```
Half-yearly: rate = 5%, periods = 2
A = 10,000 × (1.05)² = 10,000 × 1.1025 = ₹11,025
CI = ₹1,025
(Vs SI = ₹1,000. Difference = ₹25 = P(R/100)² = 10000×(0.05)² = 25 ✓)
```

**Q4.** Sum at CI 20% p.a. becomes ₹14,400 in 2 years. Find principal.
```
A = P(1.2)² = 1.44P = 14,400
P = ₹10,000
```

**Q5 (Trick 3 — GP property).** A sum at CI gives ₹1,100 after 1 year and ₹1,210 after 2 years.
Find principal and rate.
```
Ratio A₂/A₁ = 1210/1100 = 1.1 → (1 + R/100) = 1.1 → R = 10%
A₁ = P × 1.1 = 1100 → P = ₹1,000
```

**Q6.** At what SI rate does ₹5,000 become ₹6,500 in 3 years?
```
SI = 1500. 5000 × R × 3/100 = 1500 → R = 10%
```

---

# MODULE 7 — RATIOS AND PROPORTIONS

## Core Idea
A ratio is a comparison. Every ratio problem is secretly an algebra problem — but you can solve
most of them without setting up equations by using the fraction/multiplier directly.

## Formula Bank

```
Combining two ratios:
  a:b = p:q and b:c = r:s
  → a:b:c = pr : qr : qs
  (Make b the LCM of q and r)

Division in ratio a:b:c:
  Each share = Total × (their part)/(a+b+c)

Componendo-Dividendo:
  If a/b = c/d, then (a+b)/(a−b) = (c+d)/(c−d)
  Saves time in complex ratio equations.
```

## Tricks

**Trick 1: Always make the bridging term equal**
a:b = 2:3, b:c = 4:5.
Make b equal: LCM(3,4) = 12.
a:b = 8:12, b:c = 12:15. So a:b:c = 8:12:15.

**Trick 2: k-method**
Set a = kp, b = kq when a:b = p:q. Substitute into the equation.
Especially powerful when ratios change after adding/removing a quantity.

**Trick 3: Inverse proportions**
More men → fewer days (inverse). More speed → less time (inverse).
Word problems: identify whether direct or inverse, then set up as:
x₁/x₂ = y₂/y₁ (inverse) or x₁/x₂ = y₁/y₂ (direct)

**Trick 4: Proportionality in chains (work/man problems)**
Men × Days / Work = constant
If M men do W work in D days:
M₁D₁/W₁ = M₂D₂/W₂

**Trick 5: Componendo-Dividendo shortcut**
When (ax+b)/(cx+d) = p/q and you want x, cross-multiply.
OR if given a/b = c/d type structure, apply C&D to immediately get sums/differences.

## Questions

**Q1.** Divide ₹1900 among A, B, C where A:B = 1:2 and B:C = 3:5. Find C's share.
```
A:B = 1:2 → 3:6. B:C = 3:5 → 6:10. A:B:C = 3:6:10. Total = 19 parts.
C = 1900 × 10/19 = ₹1,000.
```

**Q2.** A's age : B's age = 4:5. After 10 years, 6:7. Find A's current age.
```
4k and 5k. (4k+10)/(5k+10) = 6/7.
28k + 70 = 30k + 60. 2k = 10. k = 5.
A = 20 years.
```

**Q3.** If (3x+4y)/(3x−4y) = 7/3, find x:y.
```
Componendo-Dividendo:
[(3x+4y)+(3x−4y)] / [(3x+4y)−(3x−4y)] = (7+3)/(7−3) = 10/4
6x / 8y = 5/2
x/y = 5×8/(2×6) = 10/3 → x:y = 10:3
```

**Q4.** Three numbers in ratio 1:2:3 have HCF 12. Find LCM.
```
Numbers = 12, 24, 36.
LCM = 72.
```

**Q5.** If x:y = 3:4, find (2x+y):(x+y).
```
x=3k, y=4k.
(6k+4k):(3k+4k) = 10k:7k = 10:7.
```

**Q6.** 6 men can do 8 units in 4 days. How many men for 12 units in 3 days?
```
M × D / W = constant.
6 × 4/8 = x × 3/12
3 = x/4 → x = 12 men.
```

---

# MODULE 8 — WEIGHTED AVERAGE & GROUP AVERAGES

## Core Idea
**This is the most relevant module for analytics roles.** In data, you constantly deal with
subgroup averages (free vs paid users, mobile vs web users, sports vs drama viewers).
The trap is using a simple average when a weighted one is needed.

## Formula Bank

```
Weighted Average = Σ(wᵢ × xᵢ) / Σwᵢ

Two groups:
  Combined avg = (n₁×A₁ + n₂×A₂) / (n₁+n₂)

Alligation for groups (to find ratio):
  n₁:n₂ = (A₂ − combined) : (combined − A₁)
```

## Tricks

**Trick 1: Deviation method (fastest for mental math)**
Pick one group's average as base. Calculate how much the other group pulls it.
```
Combined = A₁ + n₂/(n₁+n₂) × (A₂ − A₁)
```
Example: A (100 people, avg 65) + B (150 people, avg 75):
Combined = 65 + 150/250 × 10 = 65 + 6 = 71.

**Trick 2: New member effect**
New member with value x joins group of n with average A:
```
New average = (nA + x)/(n+1)
Shift in avg = (x − A)/(n+1)
```

**Trick 3: Replace a member**
Member with value x replaced by member with value y in a group of n:
```
Change in average = (y − x)/n
```

**Trick 4: Removal**
Remove a member with value x from group of n with average A:
```
New average = (nA − x)/(n−1)
```

**Trick 5: Percentile/midpoint type average**
When the problem gives two averages and asks "what proportion" — jump straight to alligation.
No need to set up equations.

## Questions

**Q1.** 5 analysts earn avg ₹40,000; 8 engineers earn avg ₹70,000. Overall avg?
```
= (5×40,000 + 8×70,000) / 13
= (200,000 + 560,000) / 13
= 760,000/13 ≈ ₹58,461
```

**Q2.** Class of 30, avg = 55. 5 new students (avg 65) join. New avg?
```
Method 1: (30×55 + 5×65)/35 = (1650+325)/35 = 1975/35 = 56.43
Method 2 (deviation): 55 + 5/(30+5) × (65−55) = 55 + 50/35 = 56.43 ✓
```

**Q3.** Boys avg = 72, girls avg = 80, overall avg = 75. Ratio boys:girls?
```
Alligation:
Boys(72)         Girls(80)
         \     /
          75
         /     \
      (80−75)  (75−72)
         5         3
boys:girls = 5:3
```

**Q4.** Average of 50 numbers = 38. Exclude two numbers 45 and 55. New avg?
```
Sum = 50×38 = 1900. New sum = 1900−45−55 = 1800.
New avg = 1800/48 = 37.5
```

**Q5 (Analytics framing).** Premium users: avg engagement 120 min/day.
Free users: 40 min/day. Overall avg = 60 min/day.
What fraction of users are premium?
```
Alligation:
Free(40)           Premium(120)
        \        /
         60
        /        \
     (120−60)   (60−40)
        60         20
free:premium = 60:20 = 3:1 → premium fraction = 1/(3+1) = 25%
```

**Q6 (Replace).** In a group of 10, average = 55. A student who scored 40 is replaced by one who scored 80. New avg?
```
Change = (80−40)/10 = 4. New avg = 55 + 4 = 59.
```

---

# MODULE 9 — NUMBER SYSTEM

## Core Idea
Number system questions are about patterns — cyclicity, remainders, factor structure.
The trick is always to reduce the problem to a SHORT CYCLE rather than brute-forcing.

## Divisibility Rules

| Divisor | Rule |
|---|---|
| 2 | Last digit even |
| 3 | Sum of digits divisible by 3 |
| 4 | Last 2 digits divisible by 4 |
| 7 | Double last digit, subtract from rest; repeat |
| 8 | Last 3 digits divisible by 8 |
| 9 | Sum of digits divisible by 9 |
| 11 | Alternating digit sum divisible by 11 |
| 12 | Divisible by both 3 and 4 |
| 25 | Last 2 digits = 00, 25, 50, or 75 |

## Unit Digit Cyclicity (Key table)

| Base | Cycle | Period |
|---|---|---|
| 1 | 1 | 1 |
| 2 | 2, 4, 8, **6** | 4 |
| 3 | 3, 9, 7, **1** | 4 |
| 4 | 4, **6** | 2 |
| 5 | 5 | 1 |
| 6 | 6 | 1 |
| 7 | 7, 9, 3, **1** | 4 |
| 8 | 8, 4, 2, **6** | 4 |
| 9 | 9, **1** | 2 |
| 0 | 0 | 1 |

For cycle-4 bases (2,3,7,8): **find exponent mod 4**.
- If mod 4 = 0 → use position 4 (the bold unit digit above)
- Else → use that position in the cycle

## Factor Counting

```
If N = pᵃ × qᵇ × rᶜ (prime factorization):

Number of factors = (a+1)(b+1)(c+1)

Sum of factors = [(p^(a+1)−1)/(p−1)] × [(q^(b+1)−1)/(q−1)] × ...

For N to be a perfect square: all exponents must be even.

Number of even factors = (total factors) − (odd factors)
Odd factors: ignore factor of 2; (b+1)(c+1)... on odd primes.
```

## Remainder Theorems

```
Key properties:
(a + b) mod n = ((a mod n) + (b mod n)) mod n
(a × b) mod n = ((a mod n) × (b mod n)) mod n
aⁿ mod m: find the cycle of a mod m, then use exponent mod cycle-length.

Fermat's Little Theorem:
  If p is prime and gcd(a,p)=1: a^(p−1) ≡ 1 (mod p)

Wilson's Theorem:
  (p−1)! ≡ −1 (mod p) for prime p
```

**HCF and LCM:**
```
HCF × LCM = product of two numbers (only for exactly TWO numbers)
HCF always divides LCM.
LCM/HCF = product / HCF²
```

## Tricks

**Trick 1: Unit digit — find exponent mod (cycle length)**
7^75: cycle of 7 has period 4. 75 mod 4 = 3. 3rd in cycle {7,9,3,1} = 3.

**Trick 2: Large remainders — Fermat + cycle reduction**
For prime moduli, Fermat reduces the exponent dramatically.
For non-prime moduli, find the actual pattern of aⁿ mod m.

**Trick 3: "Which power gives remainder 1" — find the order**
Just compute a², a³, a⁴... mod m until you hit 1. The first such power is the "order."

**Trick 4: Perfect square condition**
If N must be divisible by some number AND be a perfect square:
Find LCM of the divisibility constraint, then make all exponents even.

**Trick 5: Number of trailing zeros in n!**
= ⌊n/5⌋ + ⌊n/25⌋ + ⌊n/125⌋ + ...
This counts the factor of 5 (the limiting factor when pairing with 2s).

## Questions

**Q1.** Unit digit of 7^75 + 6^83.
```
7^75: 75 mod 4 = 3 → position 3 in {7,9,3,1} = 3.
6^83 = always 6.
Sum's unit digit = 3 + 6 = 9.
```

**Q2.** Remainder when 2^70 divided by 7.
```
Fermat: 2^6 ≡ 1 (mod 7).
70 = 6×11 + 4 → 2^70 ≡ 2^4 = 16 ≡ 2 (mod 7).
Answer: 2.
```

**Q3.** How many factors does 720 have?
```
720 = 2^4 × 3^2 × 5
Factors = (4+1)(2+1)(1+1) = 5×3×2 = 30.
```

**Q4.** Number gives remainder 3 ÷ 5, and remainder 4 ÷ 7. Smallest such number?
```
N = 5a + 3 = 7b + 4.
Trying: b=2 → 7×2+4=18. 18/5 = 3 rem 3 ✓.
Answer: 18.
(Or use Chinese Remainder: solve 5a ≡ 1 (mod 7) → a ≡ 3, so N = 5×3+3=18.)
```

**Q5.** N divisible by 8 and 15. Smallest N that's also a perfect square.
```
LCM(8,15) = 120 = 2^3 × 3 × 5.
For perfect square: all exponents must be even.
→ 2^4 × 3^2 × 5^2 = 16 × 9 × 25 = 3600.
```

**Q6.** How many trailing zeros in 100!?
```
⌊100/5⌋ + ⌊100/25⌋ + ⌊100/125⌋ = 20 + 4 + 0 = 24 zeros.
```

**Q7.** Find the HCF of 36 and 84. Then find LCM.
```
36 = 2^2 × 3^2. 84 = 2^2 × 3 × 7.
HCF = 2^2 × 3 = 12. LCM = 36×84/12 = 252.
```

---

# QUICK REFERENCE CARD

## Must-Know Formulas at a Glance

```
% successive change:     a + b + ab/100
Profit-Loss (same SP):   loss = x²/100 %
Alligation ratio:        (c2−m) : (m−c1)
Repeated dilution:       C × (1 − r/V)^k
A+B time:                ab/(a+b)
CI−SI (2 years):         P(R/100)²
Avg speed (equal dist):  2ab/(a+b)
Avg speed (equal time):  (a+b)/2
Trailing zeros in n!:    ⌊n/5⌋ + ⌊n/25⌋ + ...
Number of factors:       (a+1)(b+1)(c+1) for pᵃ×qᵇ×rᶜ
Catch-up time:           Head start / Relative speed
```

## The 5-Step OA Attack Strategy

1. **Read the question end-first** — see what's being asked before reading the full problem
2. **Identify the type** from the first 5 words — doesn't need full reading to classify
3. **Assign CP=100 or LCM=total** before opening a variable
4. **Use multipliers, not formulas** — write 1.2 × 0.8 not P + P×20/100 − ...
5. **Sanity check the answer** — does the magnitude make sense? Is it positive when it should be?
