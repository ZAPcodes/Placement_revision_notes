# Pattern Reasoning: Numbers, Letters, Pictures

Study notes for JioStar / placement aptitude (CAT-minus-English style).

**Core idea:** A “pattern” is not a lucky guess. Run a **fixed checklist** until one rule fits **every** given term. If two rules fit, the next term (or an option) kills one of them.

---

## 0. How to attack any series

Use the same five steps for numbers, letters, and figures.

1. **Count the terms.** 4–5 terms → likely one rule. 6+ terms → often **two series interleaved**.
2. **Write the gap** (numbers), **positions** (letters), or **what changed** (pictures).
3. Try rules in this order: **gap → gap-of-gap → ×/÷ → squares/cubes/primes → two-in-one → digit/reverse/weird**.
4. The rule must work for **all** given terms, not just the last two.
5. If the question is **odd one out**, find the rule that 4 follow and 1 breaks.

---

## 1. Number patterns

### 1.1 Checklist (look in this order)

| Order | Pattern | What you write | Typical look |
|------:|---------|----------------|--------------|
| 1 | Constant gap (AP) | +d, +d, +d | 7, 10, 13, 16 |
| 2 | Gaps themselves in AP | +2, +4, +6, +8 | 3, 5, 9, 15, 23 |
| 3 | Second difference constant | often **n²** or **n(n+1)** | 2, 6, 12, 20, 30 |
| 4 | ×k or ×k ± m | ×2, ×2+1, ×3−1 | 3, 6, 12, 24 / 5, 11, 23, 47 |
| 5 | Geometric (GP) | ×2, ×3, ×½ | 2, 6, 18, 54 |
| 6 | Squares / cubes nearby | n², n²±1, n³, n³±n | 8, 27, 64, 125 |
| 7 | Primes | 2, 3, 5, 7, 11 or prime±1 | 4, 6, 8, 12, 18 |
| 8 | Two series in one | odd places one rule, even another | 2, 5, 4, 10, 8, 20 |
| 9 | Each term from previous two | Fibonacci-like a+b | 1, 3, 4, 7, 11 |
| 10 | Digit play | sum of digits, reverse | 17, 71, 18, 81, 19, 91 |
| 11 | Factorials | 1, 2, 6, 24, 120 | obvious once you see 24, 120 |
| 12 | n²−n, n²+n, triangular | 0, 2, 6, 12, 20 | n(n+1) |

**Memory hooks**

- Second difference constant ⇒ think **squares**.
- Third difference constant ⇒ think **cubes**.
- Terms 1, 4, 9, 16: gaps +3, +5, +7 (gaps of gaps = +2) → squares.

### 1.2 Lists to recognise on sight

**Primes:** 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31  
**Squares:** 1, 4, 9, 16, 25, 36, 49, 64, 81, 100, 121, 144, 169, 196, 225  
**Cubes:** 1, 8, 27, 64, 125, 216, 343, 512, 729, 1000  
**Triangular:** 1, 3, 6, 10, 15, 21, 28, 36  
**Powers of 2:** 2, 4, 8, 16, 32, 64, 128  
**Factorials:** 1, 2, 6, 24, 120, 720

If a term is **24, 36, 48, 120, 121, 125, 128, 144**, name the family before computing gaps.

### 1.3 Eight families with examples

**A. Arithmetic (constant gap)**  
2, 7, 12, 17, 22 → **+5**. Next **27**.

**B. Increasing gap (first difference is AP)**  
2, 3, 5, 8, 12, 17 → gaps +1, +2, +3, +4, +5. Next **23**.

**C. n² or n² ± k**  
8, 15, 24, 35, 48 → 3²−1, 4²−1, 5²−1, 6²−1, 7²−1. Next **8²−1 = 63**.  
Also: 10, 17, 26, 37 → n²+1 starting from n = 3.

**D. n³ nearby**  
7, 26, 63, 124 → 2³−1, 3³−1, 4³−1, 5³−1. Next **215**.

**E. × then ± (very common OA)**  
3, 7, 15, 31, 63 → ×2+1 each time. Next **127**.  
This is the usual “looks random” series.

**F. Two series woven**  
4, 9, 6, 18, 8, 27, 10, ?  
- Odd places: 4, 6, 8, 10 → +2  
- Even places: 9, 18, 27, ? → +9 or ×1, ×2, ×3 → **36**

If you force one rule on the whole line, you waste minutes.

**G. Previous two make the next**  
2, 3, 5, 8, 13, 21 → Fibonacci. Next **34**.  
Variant: 2, 5, 7, 12, 19 → each = sum of previous two.

**H. Digit / reverse**  
17, 71, 18, 81, 19, ? → number, reverse, next, reverse. Next **91**.  
22, 24, 28, 36, 52 → +2, +4, +8, +16 (gaps doubling). Next **84**.

**Extra: n(n+1)**  
2, 6, 12, 20, 30, ? → 1×2, 2×3, 3×4, 4×5, 5×6, **6×7 = 42**.  
Same series as gaps +4, +6, +8, +10, +12.

### 1.4 Number analogies (a : b :: c : ?)

Look for the **same operation**, not “nearby numbers.”

Test in this order: ×k, n², n²±n, n³, reverse digits, sum of digits, n²±1, next prime.

| Stem | Think |
|------|--------|
| 5 : 25 :: 7 : ? | square → **49** |
| 5 : 26 :: 7 : ? | n²+1 → **50** |
| 6 : 18 :: 8 : ? | n²/2 → **32** |
| 8 : 28 :: ? | try n×3+4 → 8×3+4=28; apply to the third term the same way |

### 1.5 Worked mixed drill (numbers)

1. 2, 6, 12, 20, 30, ? → **42** (n(n+1) or gaps +4,+6,+8,+10,+12)  
2. 3, 7, 15, 31, 63, ? → **127** (×2+1)  
3. 5, 6, 9, 14, 21, ? → **30** (gaps +1,+3,+5,+7,+9)  
4. 2, 10, 4, 12, 8, 16, 16, ? → two series: 2,4,8,16 (×2) and 10,12,16,? (gaps +2,+4,+8) → **24**

---

## 2. Letter patterns

Convert **every letter to a number** on scrap paper. That is most of the topic.

### 2.1 Position table

```
A  B  C  D  E  F  G  H  I  J  K  L  M
1  2  3  4  5  6  7  8  9 10 11 12 13

N  O  P  Q  R  S  T  U  V  W  X  Y  Z
14 15 16 17 18 19 20 21 22 23 24 25 26
```

**Opposite letters** (A↔Z, B↔Y): position + opposite = **27**.

```
A-Z  B-Y  C-X  D-W  E-V  F-U  G-T
H-S  I-R  J-Q  K-P  L-O  M-N
```

Landmarks:

- **EJOTY** = positions 5, 10, 15, 20, 25  
- **CFILORUX** = 3, 6, 9, 12, 15, 18, 21, 24

### 2.2 Checklist (look in this order)

| Order | Pattern | Example |
|------:|---------|---------|
| 1 | Constant skip | A, C, E, G → +2 |
| 2 | Increasing skip | A, C, F, J → +2, +3, +4 |
| 3 | Reverse alphabet | Z, X, V, T → −2 |
| 4 | Opposite letters | A, Z, B, Y, C, X |
| 5 | Two letter-series interleaved | A, Z, C, X, E, V |
| 6 | Vowels only / consonants only | A, E, I, O, ? → U |
| 7 | Each letter +1 or −1 in a group | CAT, DBU, ECV |
| 8 | Reverse the group | ABCD → DCBA |
| 9 | Position arithmetic | D=4, H=8, L=12 → +4 |
| 10 | Skip as primes/squares | A, B, D, G, K → +1,+2,+3,+4 |

### 2.3 Letter series examples

**Constant skip**  
B, E, H, K, N → +3. Next **Q**.

**Increasing skip**  
A, C, F, J, O → +2, +3, +4, +5. Next **U**.

**Square positions**  
A, D, I, P, ? → positions 1, 4, 9, 16, **25 = Y**.

**Opposite pair chain**  
AZ, BY, CX, DW → next **EV**.

**Interleaved**  
A, R, C, P, E, N, G, ?  
- Odd: A, C, E, G → +2  
- Even: R, P, N, ? → −2 → **L**

**Word-like groups**  
ACE, BDF, CEG, ? → each letter +1. Next **DFH**.  
Also: ACE, GIK, MOQ → skip 1 inside the triple; groups jump +6.

**Reverse group**  
MNOP, PONM, QRST, TSRQ → next block **UVWX**.

**Opposites + skip**  
AZ, GT, MN, ?  
- First letters: A, G, M → +6 → **S**  
- Each pair sums to 27 (opposites). Opposite of S is **H**.  
Answer **SH**.

### 2.4 Letter coding (“in a certain code”)

Look for **one** of these; do not invent two operations at once unless the stem forces it.

1. **Shift:** CAT → DBU (+1 each)  
2. **Reverse then shift:** CAT → TAC → UBD  
3. **Opposite:** CAT → XZG (C↔X, A↔Z, T↔G)  
4. **Split and reverse:** ORANGE → ROE GNA (first 3 reversed, last 3 reversed)  
5. **Alternate +1 / −1**  
6. **Position numbers:** CAT → 3-1-20  
7. **Next vowel / next consonant**

For “PENCIL is coded as … write PAPER as …”: apply the **same steps in the same order**.

---

## 3. Alphanumeric (letters + numbers + symbols)

These look hard; they are usually **two or three independent tracks**. Split, solve, glue back.

**Example:** `A2C, D5F, G10I, J17L, ?`

- Letters: A_C, D_F, G_I, J_L → skip 1 in the middle; starts A, D, G, J (+3)  
- Middle number: 2, 5, 10, 17 → gaps +3, +5, +7 → next +9 = **26**  
- Next letters: **M_O**

Answer **M26O**.

**What to split**

- Letter progression  
- Number progression  
- Symbol cycle (`* # @`) if present  
- Position of the number (start vs middle vs end)

Never treat `A2C` as one object until the tracks are split.

---

## 4. Picture / figure patterns

Non-verbal reasoning. Ask: **what changed from box 1 to 2, and does the same change happen 2 → 3?**

### 4.1 Twelve things a figure can do (look in this order)

1. **Rotation** — 45°, 90°, 180° CW or ACW; sometimes alternate CW / ACW. Watch a marked corner or arrow walking around the square.  
2. **Reflection** — left-right mirror, or water image (up-down). If rotation fails, flip.  
3. **Number of sides / shapes** — triangle → square → pentagon → hexagon, or 1 circle, 2, 3.  
4. **Count of elements** — dots, lines, arrows: 1,2,3,4 or 1,3,5 or 2,4,8.  
5. **Shading** — empty → half → full, or black sector moving like a clock.  
6. **Movement on a path** — a dot on a triangle’s vertices; an arrow around a square’s sides.  
7. **Add / delete a line** — each step adds one line, or a stick figure gains an arm then a leg.  
8. **Element swap** — two shapes trade places; outer becomes inner.  
9. **Overlap** — two shapes move closer, overlap, then one sits inside the other.  
10. **Alternate figures** — 1,3,5 follow rule A; 2,4,6 follow rule B (same as interleaved numbers).  
11. **Paper folding / punch holes** — unfold in **reverse** order. Holes multiply by powers of 2 only if they are not on the fold line.  
12. **Embedded / completion** — pick the option that continues lines and shading without extra ink.

### 4.2 3×3 figure matrix

Find the missing bottom-right. Test:

- Each row: same transformation (rotate 90°, then 90° again)  
- Each row: count of lines = 2, 3, 4  
- Three shapes in a row are rotations of one another  
- Columns copy the row rule  
- **Overlay / XOR:** figure 3 = lines of 1 combined with 2; overlapping line cancelled

**Fast count (ignore art):**

- How many lines?  
- How many closed regions?  
- How many black parts?  
- Where is the unique mark?

If counts go 2, 3, 4 across a row, the blank is 5 of that thing.

### 4.3 Picture analogies (A is to B as C is to ?)

Ask: **what did they do to A to get B?** Do **exactly that** to C.

Typical operations: rotate 90° CW, mirror, increase sides by 1, invert shading, move the inner shape to the corner the outer arrow points to.

Try **one** operation first. Only then rotate **and** shade.

### 4.4 Odd one out in figures

Four follow one rule, one does not. Check:

1. Number of sides  
2. Open vs closed  
3. Rotation of the same shape (odd one is **flipped** — mirror is not a rotation)  
4. Number / direction of arrows  
5. Shading count  
6. Extra line

A right-handed spiral, mirrored, is a common odd one.

### 4.5 Figure series in words (mental drill)

A triangle rotates 90° clockwise each step. A dot sits on the vertex that is “on top after each rotation.” After 3 steps the original top vertex has moved **270° CW**; the dot is on that vertex. Draw it; do not do the first week in your head.

---

## 5. Related pattern types (short)

These are not always “series,” but they sit in the same OA section.

| Type | What to look for |
|------|------------------|
| Coding–decoding | Shift, reverse, opposite, split-reverse (see §2.4) |
| Mirror image of a word | Reverse left–right |
| Water image | Reverse up–down (b↔p, q↔d style) |
| Analogy (words) | Doctor : Hospital :: Teacher : School |
| Classification | Three in a class, one odd |

Direction sense, blood relations, seating, syllogism are **separate LR chapters**, not series. See the remaining-topics list in the session notes.

---

## 6. One-page recap

**Numbers:** gap → gap of gap → ×k±m → square/cube/prime → two series → digit reverse.

**Letters:** A=1 … Z=26 → skip size → opposite (sum 27) → two series → vowels → reverse block.

**Pictures:** rotate → flip → count sides/dots → shade moves → piece moves one step → add/remove line → two-series of figures → overlay.

**Alphanumeric:** split letter track and number track. Solve separately. Glue back.

---

## 7. Mini drill (cover answers)

**Questions**

1. 2, 6, 12, 20, 30, ?  
2. 3, 7, 15, 31, 63, ?  
3. 5, 6, 9, 14, 21, ?  
4. 2, 10, 4, 12, 8, 16, 16, ?  
5. A, D, I, P, ?  
6. AZ, GT, MN, ?  
7. ACE, BDF, CEG, ?  

**Answers**

1. **42**  
2. **127**  
3. **30**  
4. **24**  
5. **Y**  
6. **SH**  
7. **DFH**
