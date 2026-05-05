# BayesT_A_B_Calculator — README

**File:** `BayesT_A_B_Calculator.html`  
**Author:** Eric L.  
**Last updated:** May 5, 2026

---

## Overview

A single-file, dependency-free HTML/JavaScript tool for calculating **Bayes' theorem** — specifically the posterior probability P(A|B) — together with **95% credible intervals** and **parametric bootstrap confidence intervals**.

Open the file in any modern browser (Chrome, Edge, Firefox, Safari). No installation, no server, no internet connection required.

---

## The Four Methods (Tabs)

### ① Given P(A) — Method 1

The standard Bayes' theorem calculation when the prior P(A) is known directly.

**Inputs:**
- P(A) — prior probability of A (e.g. disease prevalence)
- P(B|A) — likelihood of B given A (e.g. test sensitivity)
- P(B|¬A) — likelihood of B given ¬A (e.g. false-positive rate)

**Formula:**
$$P(B) = P(B|A)\cdot P(A) + P(B|\neg A)\cdot P(\neg A)$$
$$P(A|B) = \frac{P(B|A)\cdot P(A)}{P(B)}$$

**Output:** P(A|B), P(¬A|B), P(A|¬B), P(¬A|¬B), P(B), plus a joint probability table and visual bar.

---

### ② Given P(B) — Method 2

Useful when you know the overall marginal probability P(B) but not P(A) directly. P(A) is **recovered algebraically**.

**Inputs:**
- P(B) — overall (marginal) probability of B
- P(B|A) and P(B|¬A) as direct probabilities

**Formula:**
$$P(A) = \frac{P(B) - P(B|\neg A)}{P(B|A) - P(B|\neg A)}$$

**Output:** same as Method 1, plus the recovered P(A).

---

### ③ CI given P(A) — Method 3

Runs **both** a Bayesian credible interval and a parametric bootstrap CI in a single simulation of 1,000,000 draws. P(B|A) and P(B|¬A) are entered as **raw counts** (successes / total), giving them their own sampling uncertainty.

**Inputs:**
- P(A) — as a direct probability *or* as counts (successes / total)
- P(B|A) — as counts: successes and group size
- P(B|¬A) — as counts: successes and group size

**Credible interval (Bayesian, HPD):**  
Samples P(B|A), P(B|¬A), and optionally P(A) from Beta posteriors (Beta-Binomial model with uniform priors: Beta(s+1, f+1)). Propagates uncertainty through the Bayes formula. Reports the **95% Highest Posterior Density (HPD)** interval — the shortest interval containing 95% of the posterior mass.

**Bootstrap CI (frequentist):**  
Resamples counts from Binomial distributions centred on observed proportions. Reports the **equal-tailed 95% percentile interval**.

**Point estimate:**  
Always uses raw observed proportions (s/n), so boundary cases like P(B|¬A) = 0/n correctly yield P(A|B) = 1.0.

**Output:** point estimate, posterior mean, posterior median, 95% HPD interval, 95% bootstrap CI, visual CI bars, full probability table.

---

### ④ CI given P(B) — Method 4

Same simulation approach as Method 3, but P(A) is **recovered from P(B)** algebraically in each draw rather than supplied as an input. Useful when you only have the marginal prevalence of B, not A.

**Inputs:**
- P(B) — as a direct probability *or* as counts
- P(B|A) and P(B|¬A) — as counts

**Output:** same as Method 3, including the recovered P(A) point estimate.

---

## Results Displayed for All Tabs

| Quantity | Description |
|---|---|
| P(A\|B) | Posterior probability — the main result |
| P(¬A\|B) | Complement of P(A\|B) |
| P(A\|¬B) | Posterior probability given B is absent |
| P(¬A\|¬B) | Complement of P(A\|¬B) |
| P(B) | Marginal probability of B |
| Joint table | Full 2×2 table of joint probabilities P(row ∩ col) |
| Probability bar | Visual representation of P(A\|B) |
| 95% HPD interval | Tabs 3 & 4: Bayesian credible interval |
| 95% Bootstrap CI | Tabs 3 & 4: Frequentist bootstrap interval |

Results can be saved to a `.txt` file using the **Save Results** button on each tab.

---

## Technical Notes

### Simulation
- All simulations use **1,000,000 draws** with `Float64Array` for memory efficiency.
- Random Beta samples use the Johnk method (sum of Gamma variates).
- Random Binomial samples use exact enumeration for n ≤ 50, Poisson approximation for small expected counts (μ < 30), and normal approximation otherwise.

### Jeffreys-prior boundary correction
- **Point estimates** always use raw proportions s/n. A boundary count of 0/n correctly yields P = 0, and if P(B|¬A) = 0 exactly, then P(A|B) = 1.0 (mathematically correct — B never occurs in the ¬A group).
- **Bootstrap hats** (the p values passed to `randBinomial`) apply the Jeffreys prior correction `(s + 0.5) / (n + 1)` when s = 0 or s = n. This prevents `Binomial(n, 0)` from collapsing to an all-zero distribution, which would produce a degenerate upper CI of 1.0.
- **Beta-Binomial credible interval**: uses `Beta(s+1, f+1)`, which is already well-defined at s = 0 or f = 0 and needs no correction.

### HPD interval algorithm
The 95% HPD interval is found by sliding a window of width = 0.95 × N over the sorted posterior samples and selecting the narrowest such window.

---

## Usage Examples

**Example 1 — Diagnostic test (Method 1)**

A genetic condition has prevalence 1 in 1000 (P(A) = 0.001). A screening test has 99% sensitivity (P(B|A) = 0.99) and 5% false-positive rate (P(B|¬A) = 0.05).

→ P(A|B) ≈ 0.0194 (~2%): even a positive test result leaves the condition unlikely due to the low prior.

**Example 2 — With uncertainty (Method 3)**

P(A) = 0.001; in the A-positive group, 4 of 12039 had the outcome; in the A-negative group, 0 of 269885 had the outcome.

→ Point estimate P(A|B) = 1.0 (correct, since the outcome never occurred in ¬A); the 95% HPD interval will reflect the uncertainty around this boundary estimate.

---

## Changelog

| Date | Change |
|---|---|
| May 5, 2026 | Jeffreys prior correction scoped to bootstrap sampling only; point estimates now always use raw s/n. Added `jeffreysP()` function. |
| — | Initial release with 4-tab layout, Beta-Binomial credible interval, parametric bootstrap CI, HPD algorithm, save-to-file, visual CI bars. |

---

## Related Files

- `BayesT_A_B_Calculator_1.html` — Earlier version of the same calculator (same logic)
- `BayesT_A_B_Claude.py` — Python CLI version with identical statistical methods
- `Bayes.js` — Node.js script generating a PowerPoint slide deck on Bayes' theorem
