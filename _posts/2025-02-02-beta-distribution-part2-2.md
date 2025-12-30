---
title: "Beta Distribution: Part 2.2 – Frequentist Perspectives – Exact Hypothesis Testing"
date: 2025-02-02
author: peng
categories: [Quantitative Research]
tags: [statistics, beta-distribution]
math: true
---

In [Part 2.1]({% post_url 2025-01-25-beta-distribution-part2-1 %}), we explored how the Beta distribution enables exact confidence intervals for binomial proportions through the Clopper-Pearson method. This post completes the picture by demonstrating how the **exact binomial test** emerges naturally from the same Beta distribution framework.

## 1. The Duality of Confidence Intervals and Hypothesis Testing

Recall our key insight from [Part 2.1]({% post_url 2025-01-25-beta-distribution-part2-1 %}): 

> 📌 Every confidence interval corresponds to a set of plausible null hypotheses. Specifically, a $$100(1-\alpha)\%$$ Clopper-Pearson interval contains all values $$p_0$$ that would not be rejected by an $$\alpha$$-level exact binomial test.

This duality transforms interval estimation into hypothesis testing. Given:
- Observed successes: $$k = 8$$
- Total trials: $$n = 20$$
- Null hypothesis: $$H_0: p = 0.70$$

We can either:
1. Calculate the 95% Clopper-Pearson interval and check if 0.70 lies outside it, **or**
2. Perform an exact binomial test directly

Both approaches yield identical conclusions because they share the same Beta distribution foundation.

## 2. Exact Binomial Test Implementation

### 2.1 Test Formulation

For $$X \sim \text{Binomial}(n, p_0)$$, the one-sided p-value is:

$$P(X \leq k) = \sum_{i=0}^k \binom{n}{i}p_0^i(1-p_0)^{n-i} = 1 - I_{1-p_0}(k+1, n-k)$$

where $$I_x(a,b)$$ is the Beta CDF ([derived in Part 2.1]({% post_url 2025-01-25-beta-distribution-part2-1 %}#3-from-binomial-sums-to-the-incomplete-beta-function)). This identity lets us compute p-values using Beta distribution quantiles - exactly as we computed Clopper-Pearson bounds.

## 2.2 Practical Example: Pharmaceutical Trial

Reusing our [clinical trial example]({% post_url 2025-01-25-beta-distribution-part2-1 %}#simulations-coverage-and-interval-width) where $$k=8$$ successes out of $$n=20$$ trials:

```r
# 95% Clopper-Pearson CI
binom.confint(8, 20, method = "exact") 
# Lower = 0.191, Upper = 0.584

# Exact binomial test
binom.test(8, 20, p=0.70, alternative="less")$p.value
# 0.005138162
```

**Key Findings:**

1. The null value $$p_0 = 0.70$$ lies outside the CI $$(0.191, 0.584)$$
2. The p-value (0.0051) < 0.05 significance level


**Three Paths to Exactness:**

We confirm consistency through different implementations:
- `binom.test()` internal implementation in R
- Direct binomial CDF summation
- Beta CDF transformation ([mathematical basis]({% post_url 2025-01-25-beta-distribution-part2-1 %}#3-from-binomial-sums-to-the-incomplete-beta-function))

and compare the outputs:

```
Exact binom.test P-value: 0.005138162 
Direct Binomial CDF: 0.005138162 
Beta function approach: 0.005138162
```

{::options parse_block_html="true" /}

<details>
<summary markdown="span">Implementation Details</summary>

```r
k <- 8; n <- 20; p0 <- 0.70

# Method 1: Built-in exact test
binom.test(k, n, p0, alternative="less")$p.value

# Method 2: Binomial CDF
pbinom(k, n, p0)

# Method 3: Beta CDF 
pbeta(1-p0, n-k, k+1)  # Beta(12,9) CDF at 0.30
```
</details>

{::options parse_block_html="false" /}

**Conclusion:**  

Identical results (p-values: 0.0051) across methods demonstrate:
- Practical equivalence of binomial sums and Beta CDFs
- Consistent exact implementation in statistical software
- Clear rejection of $$H_0: p = 0.70$$ at $$ \alpha =0.05 $$


## 3. Beta Connection: Unifying Intervals and Tests

The [Clopper-Pearson interval]({% post_url 2025-01-25-beta-distribution-part2-1 %}) bounds $$p_{\text{lower}}$$ and $$p_{\text{upper}}$$ satisfy:

$$
\begin{aligned}
I_{p_{\text{lower}}}(k, n-k+1) &= 1 - \alpha/2 \\
I_{p_{\text{upper}}}(k+1, n-k) &= \alpha/2
\end{aligned}
$$

Compare this to the exact test's p-value equation:

$$\text{p-value} = 1 - I_{1-p_0}(k+1, n-k)$$

The same [incomplete Beta function]({% post_url 2025-01-25-beta-distribution-part2-1 %}#3-from-binomial-sums-to-the-incomplete-beta-function) governs both procedures. When $$p_0$$ equals a Clopper-Pearson bound, the p-value equals $$\alpha$$ - demonstrating their inverse relationship.

## 4. Coverage and Precision Tradeoffs

Our [simulation analysis in Part 2.1]({% post_url 2025-01-25-beta-distribution-part2-1 %}#simulations-coverage-and-interval-width) showed that Clopper-Pearson intervals achieve exact coverage at the cost of wider intervals. Similarly, exact binomial tests:
- Guarantee type I error ≤ α (conservative for discrete distributions)
- Are more reliable than approximate tests (e.g., Wald test) for small $$n$$

This mirrors our interval coverage findings - the price of exactness is reduced precision, a fundamental frequentist tradeoff.

## 5. Summary

Through Clopper-Pearson intervals and exact binomial tests, we've seen how the Beta distribution provides exact inference for binomial proportions while connecting interval estimation and hypothesis testing through mathematical duality. In my next post, we'll explore its role in modeling over-dispersed data through the Beta-Binomial framework.
