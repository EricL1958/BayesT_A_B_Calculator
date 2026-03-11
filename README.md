# Eric's Bayes Theorem Calculator (HTML)

A single-file, self-contained HTML/JavaScript Bayesian calculator that runs entirely in the browser — no server, no dependencies, no installation required. Developped by Eric LEGIUS with the help of Claude Sonnet 4.6 in Visual Code Studio.

## How to Use

Open `BayesT_A_B_Calculator.html` directly in any modern web browser (Chrome, Firefox, Edge, Safari). No internet connection is needed once the file is on your machine.

---

## Tabs / Methods

The calculator has four tabs, each offering a different way to apply Bayes' theorem.

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

### Tab 3 — Bayesian 95% Credible Interval (Beta-Binomial)

**Use this when you have raw count data and want a Bayesian uncertainty range.**

Instead of entering probabilities directly, you enter the observed **counts** from which P(B|A) and P(B|¬A) were estimated. Each likelihood is modelled as a **Beta distribution** (conjugate prior for the binomial) and the posterior distribution of P(A|B) is obtained by propagating 1,000,000 Monte Carlo draws through Bayes' theorem.

| Input | Description |
|---|---|
| P(A) | Prior probability of A |
| Successes (A-positive group) | Observed positive outcomes among A-positives |
| Total size (A-positive group) | Total count in the A-positive group |
| Successes (A-negative group) | Observed positive outcomes among A-negatives |
| Total size (A-negative group) | Total count in the A-negative group |

**Outputs:**
- P(B|A) and P(B|¬A) observed rates
- P(B) point estimate
- P(A|B) point estimate
- Posterior mean and median
- **95% Credible Interval** — there is a 95% probability that the true P(A|B) lies within this range
- Visual CI bar

> **Interpretation:** A credible interval is a Bayesian concept. The interval directly expresses probability about the parameter itself (e.g. "there is a 95% chance P(A|B) is between 0.12 and 0.35").

---

### Tab 4 — Frequentist 95% Bootstrap Confidence Interval

**Use this when you want a frequentist uncertainty estimate via resampling.**

Uses a **parametric bootstrap**: new success counts are resampled from Binomial distributions (1,000,000 replicates), simulating how the observed data could vary by chance.

| Input | Description |
|---|---|
| P(A) | Prior probability of A |
| Successes / Total (A-positive group) | Same as Tab 3 |
| Successes / Total (A-negative group) | Same as Tab 3 |

**Outputs:**
- Point estimates (same as Tab 3)
- Bootstrap mean
- **95% Bootstrap CI** — 95% of such intervals from repeated experiments would contain the true P(A|B)
- Visual CI bar

> **Interpretation:** A frequentist confidence interval is a property of the procedure, not of a single interval. It means: if the experiment were repeated many times, 95% of the computed intervals would capture the true value.

---

## Bayesian vs. Bootstrap — Which Tab to Use?

| | Tab 3 (Credible Interval) | Tab 4 (Bootstrap CI) |
|---|---|---|
| Framework | Bayesian | Frequentist |
| Prior on likelihoods | Beta(1,1) uniform | None (fixed at observed rates) |
| Interpretation | Direct probability statement about P(A\|B) | Long-run coverage guarantee |
| Best for | Small samples, uncertainty propagation | Larger samples, frequentist reporting |

---

## Example: Medical Diagnostic Test

Suppose a disease has a **1% prevalence** (P(A) = 0.01). A test has **90% sensitivity** (P(B|A) = 0.90) and a **5% false positive rate** (P(B|¬A) = 0.05).

**Tab 1 inputs:**
- P(A) = 0.01
- P(B|A) = 0.90
- P(B|¬A) = 0.05

**Result:** P(A|B) ≈ 0.1538 — even with a positive test, there is only ~15% chance the patient has the disease, because the disease is rare.

---

## Technical Notes

- **Simulation size:** Tabs 3 and 4 each use 1,000,000 Monte Carlo / bootstrap replicates for high precision.
- **RNG:** The Beta distribution is sampled using the Marsaglia-Tsang Gamma method; the Binomial uses exact sampling for small n, Poisson approximation for moderate expected counts, and normal approximation for large expected counts.
- **No external libraries:** All math and rendering is implemented in vanilla JavaScript. No frameworks, no CDN calls.
- **Responsive layout:** The UI adapts to mobile screen widths.

---

## File

| File | Description |
|---|---|
| `BayesT_A_B_Calculator.html` | The complete application — open in a browser to run |
