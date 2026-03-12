# Eric's Bayes Theorem Calculator (HTML)

A single-file, self-contained HTML/JavaScript Bayesian calculator that runs entirely in the browser — no server, no dependencies, no installation required. Developped by Eric LEGIUS with the help of Claude Sonnet 4.6 in Visual Code Studio.

## How to Use

Open `BayesT_A_B_Calculator.html` directly in any modern web browser (Chrome, Firefox, Edge, Safari). No internet connection is needed once the file is on your machine.

---

## Tabs / Methods

The calculator has **four tabs**, each offering a different way to apply Bayes' theorem.

---

### Tab 1 — Given P(A)

**Use this when you know the prior probability of A directly.**

**Formula:**

$$P(B) = P(B|A) \cdot P(A) + P(B|\neg A) \cdot P(\neg A)$$

$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)}$$

| Input | Description |
|---|---|
| P(A) | Prior probability of A (e.g. disease prevalence) |
| P(B\|A) | Sensitivity / true positive rate / likelihood 1 |
| P(B\|¬A) | False positive rate / 1-Specificity / likelihood 2 |

**Outputs:** P(B) marginal probability, P(A\|B) posterior, P(¬A\|B), and a visual probability bar.

---

### Tab 2 — Given P(B)

**Use this when you know P(B) but not P(A). P(A) is recovered algebraically.**

**Formula:**

$$P(A) = \frac{P(B) - P(B|\neg A)}{P(B|A) - P(B|\neg A)}$$

| Input | Description |
|---|---|
| P(B) | Known marginal probability of B |
| P(B\|A) | Sensitivity / true positive rate / Likelihood 1 |
| P(B\|¬A) | False positive rate / 1-Specificity / Likelihood 2 |

**Outputs:** Recovered P(A), P(A\|B) posterior, P(¬A\|B), and a probability bar.

---

### Tab 3 — CI given P(A) — Method 3

**Use this when you know P(A) and have raw count data for the likelihoods. Both a Bayesian credible interval and a frequentist bootstrap CI are computed simultaneously in a single simulation.**

| Input | Description |
|---|---|
| P(A) | Prior probability of A — enter as a **fixed probability** or as **sample counts** s/n |
| P(B\|A) | s/n counts (successes / total in A-positive group) |
| P(B\|¬A) | s/n counts (successes / total in A-negative group) |

When P(A) is entered as counts, its sampling uncertainty is propagated through both intervals.

**Simulation — 1 000 000 draws, single pass:**

Both arrays (`posterior[]` and `boot[]`) are filled inside one loop:
- **Credible draws:** P(A), P(B|A), P(B|¬A) each sampled from their Beta posterior (uniform prior Beta(s+1, f+1)); P(A|B) evaluated per draw.
- **Bootstrap replicates:** Success counts resampled from Binomial distributions centred on the observed proportions; P(A|B) evaluated per replicate.

**Outputs:**
| Result | Description |
|---|---|
| P(B\|A), P(B\|¬A) observed | Point estimates from input counts |
| P(B) & P(A\|B) point estimates | Bayes formula with observed proportions |
| Posterior mean / median | Summary statistics of the credible draws |
| **95% HPD Interval** | Shortest interval covering 95% of posterior mass (Bayesian) |
| **Bootstrap mean** | Mean of 1 M bootstrap replicates |
| **95% Bootstrap CI** | Equal-tailed 2.5th–97.5th percentile of bootstrap replicates (Frequentist) |
| P(A\|¬B), P(¬A\|¬B) | Point estimates |
| Joint probability table | Full Bayesian probability table |
| CI visualisation bars | Credible bar + Bootstrap bar, each with point-estimate marker |

**Save file:** `BayesT_Method3_CI_PA_Results.txt`

---

### Tab 4 — CI given P(B) — Method 4

**Use this when you know P(B) (not P(A)) and have raw count data. P(A) is recovered algebraically inside every simulation draw.**

| Input | Description |
|---|---|
| P(B) | Observed marginal probability of B — enter as a **fixed probability** or as **sample counts** s/n |
| P(B\|A) | s/n counts |
| P(B\|¬A) | s/n counts |

**Simulation — 1 000 000 draws, single pass:** Same dual-array structure as Method 3. Within each draw, P(A) is recovered from:

**Formula:**

$$P(A) = \frac{P(B) - P(B|\neg A)}{P(B|A) - P(B|\neg A)}$$


Draws where the recovered P(A) falls outside [0, 1] or the denominator is near zero are discarded (set to NaN).

**Outputs:** Same structure as Method 3 but shows the recovered P(A) instead of an input P(A).

**Save file:** `BayesT_Method4_CI_PB_Results.txt`

---

## Bayesian HPD vs. Bootstrap — Interpretation

| | Credible interval (Methods 3 & 4) | Bootstrap CI (Methods 3 & 4) |
|---|---|---|
| Framework | Bayesian | Frequentist |
| Prior on likelihoods | Beta(1,1) uniform | None (fixed at observed rates) |
| Interval method | **HPD — shortest 95% interval** | Equal-tailed 2.5th–97.5th percentile |
| Interpretation | Direct probability: "95% chance the true P(A\|B) is in this range" | Long-run coverage: "95% of such intervals would capture the true value" |
| Best for | Small samples, uncertainty propagation, skewed posteriors | Larger samples, frequentist reporting |

> The HPD interval is always at least as narrow as the equal-tailed interval and is the optimal interval for unimodal posteriors.

---

## HPD Interval — Algorithm

The Highest Posterior Density (HPD) interval is computed from the sorted array of 1 000 000 posterior draws using a sliding window:

```javascript
function hpd(sorted, coverage) {
  const n = sorted.length;
  const w = Math.floor(coverage * n);   // window width = 950 000 draws
  let best = Infinity, lo = 0;
  for (let i = 0; i + w < n; i++) {
    const width = sorted[i + w] - sorted[i];
    if (width < best) { best = width; lo = i; }
  }
  return [sorted[lo], sorted[lo + w]];
}
```

The result is the contiguous block of 95% of the draws with the smallest range.

---

## Example: Medical Diagnostic Test

Suppose a disease has a **1% prevalence** (P(A) = 0.01). A test has **90% sensitivity** (P(B|A) = 0.90) and a **5% false positive rate** (P(B|¬A) = 0.05).

**Tab 1 inputs:**
- P(A) = 0.01
- P(B|A) = 0.90
- P(B|¬A) = 0.05

**Result:** P(A|B) ≈ 0.1538 — even with a positive test, there is only ~15% chance the patient has the disease, because the disease is rare.

For uncertainty estimation (Tab 3), enter the counts behind the sensitivity and false positive rate estimates; the HPD credible interval will reflect the additional uncertainty due to finite sample sizes.

---

## Technical Notes

- **Simulation size:** 1 000 000 draws per calculation (both intervals in one pass for Methods 3 & 4).
- **Array type:** `Float64Array` for performance.
- **Beta sampler:** Custom `randBeta(α, β)` using the Marsaglia-Tsang Gamma method.
- **Binomial sampler:** Custom `randBinomial(n, p)`.
- **Credible interval:** HPD via sliding window on sorted posterior draws.
- **Bootstrap CI:** Equal-tailed 2.5th/97.5th percentiles.
- **Invalid draws:** NaN-filtered before statistics; an error is shown if fewer than 100 valid draws remain.
- **Threading:** Calculation deferred via `setTimeout(..., 20)` to keep the UI responsive; a spinner is shown during computation.
- **No external libraries:** All math and rendering is implemented in vanilla JavaScript. No frameworks, no CDN calls.
- **Responsive layout:** The UI adapts to mobile screen widths.

---

## File

| File | Description |
|---|---|
| `BayesT_A_B_Calculator.html` | The complete application — open in any modern browser |

---

## Change History

### March 12, 2026

#### Tab consolidation: 6 tabs → 4 tabs

The calculator previously had six separate tabs:

| Old tab | Description |
|---|---|
| 1 | Given P(A) — point estimate |
| 2 | Given P(B) — point estimate |
| 3 | Credible interval given P(A) |
| 4 | Bootstrap CI given P(A) |
| 5 | Credible interval given P(B) |
| 6 | Bootstrap CI given P(B) |

Old tabs 3 + 4 were merged into the new **Method 3** tab, and old tabs 5 + 6 into the new **Method 4** tab. Both CIs are now computed inside a single 1 000 000-draw loop, reducing computation time and combining the results into one view and one output file per prior type.

#### HPD credible interval

The credible interval method was changed from the **equal-tailed interval** (2.5th and 97.5th percentiles) to the **Highest Posterior Density (HPD) interval** — the shortest contiguous interval covering 95% of the posterior mass. A `hpd()` helper function was added. All UI labels, tooltip text, and TXT save output were updated to reflect the new method. The bootstrap CI retains the equal-tailed percentile approach.
