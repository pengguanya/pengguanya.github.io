---
title: "Beta Distribution: Part 2.2 – Frequentist Perspectives – Exact Hypothesis Testing"
date: 2025-01-17
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution]
math: true
---

In [Part 2.1]({% post_url 2025-01-16-beta-distribution-part2-1 %}) of this series, we examined Clopper–Pearson intervals for binomial proportions, a method guaranteeing exact coverage by leveraging the Beta distribution. One of the key insights was the [duality](https://www.its.caltech.edu/~zuev/teaching/2012Spring/Math408-Lecture-33.pdf) between confidence intervals and hypothesis tests in frequentist statistics: a $$100(1 - \alpha)\%$$ confidence interval corresponds to all parameter values $$p$$ that would *not* be rejected in a two-sided $$\alpha$$-level test.

This connection naturally paves the way for **hypothesis testing** in a frequentist setting. Specifically, we will look at two classic exact tests:

1. **Exact Binomial Test**  
2. **Fisher’s Exact Test** (for $$2 \times 2$$ contingency tables)

Both are exact in the sense that they do not rely on large-sample approximations (such as the normal approximation to the binomial) and can handle small sample sizes or extreme proportions. As we will see, each test’s calculations align closely with the Beta distribution—often through the [incomplete Beta function]({% post_url _posts/2025-01-16-beta-distribution-part2-1.md#3-from-binomial-sums-to-the-incomplete-beta-function %}—mirroring the mechanism we saw with Clopper–Pearson intervals.

---

## 1. Exact Binomial Test

### 1.1 Motivation and Setup

The **Exact Binomial Test** addresses the question: “Is the true probability of success $$p$$ equal to (or less/greater than) some hypothesized value $$p_0$$?” Given:

- $$n$$: total number of independent Bernoulli trials.
- $$k$$: number of observed successes.
- $$p_0$$: hypothesized probability of success.

We set up hypotheses. For a one-sided test, for example:

$$
\begin{aligned}
H_0 &: \quad p = p_0, \\
H_1 &: \quad p < p_0.
\end{aligned}
$$

(One can also test two-sided or the alternative $$p > p_0$$.)

### 1.2 p-Value via the Binomial Distribution

Under $$H_0$$, the test statistic $$X \sim \mathrm{Binomial}(n, p_0)$$. The p-value is the probability of observing a result *as extreme or more extreme* than $$k$$. For the one-sided test $$p < p_0$$:

$$
\text{p-value}
= P(X \le k)
= \sum_{i=0}^{k} \binom{n}{i} \, p_0^{\,i} \, (1 - p_0)^{\,n - i}.
$$

If this p-value is less than significance level $$\alpha$$, we reject $$H_0$$.

### 1.3 The Beta Connection

As in [Part 2.1](../2025-01-16-beta-distribution-part2-1/), binomial cumulative probabilities can be rewritten via the *regularized incomplete Beta function*. Specifically,

$$
P(X \le k)
= 1 - I_{\,1 - p_0}(k + 1,\; n - k),
$$

where $$I_{\,1 - p_0}(\alpha,\beta)$$ is the CDF of $$\mathrm{Beta}(\alpha,\beta)$$ evaluated at $$(1 - p_0)$$. This equivalence underpins the *exactness* of the test, since we are not invoking a normal approximation.

### 1.4 Example in R — Comparing Three Approaches

Suppose a pharmaceutical company claims that a new medication has a success probability $$p_0 = 0.70$$. We conduct a clinical trial with $$n = 20$$ patients and observe $$k = 8$$ successes. We want to test:

$$
\begin{aligned}
H_0 &: \quad p = 0.70,\\
H_1 &: \quad p < 0.70.
\end{aligned}
$$

Below, we demonstrate three ways to compute the exact p-value in R:

1. **Using `binom.test()`** (R’s built-in exact test)
2. **Direct Binomial CDF** (summing binomial probabilities)
3. **Beta CDF** (via the regularized incomplete Beta function)

```r
# Parameters
k  <- 8
n  <- 20
p0 <- 0.70

# (A) Using binom.test in R:
binom_res <- binom.test(x = k, n = n, p = p0, alternative = "less")

# (B) Using direct Binomial CDF (pbinom):
# P(X <= k) under X ~ Binomial(n, p0).
p_value_binomial <- pbinom(k, size = n, prob = p0, lower.tail = TRUE)

# (C) Using the Beta CDF (pbeta):
# The relationship: P(X <= k) = I_{1-p0}(n-k, k+1).
# Or specifically: I_{0.30}(12, 9) here, since 1 - p0 = 0.30, n-k=12, k+1=9.
p_value_beta <- pbeta(0.30, shape1 = 12, shape2 = 9, lower.tail = TRUE)

cat("\nExact binom.test P-value:", binom_res$p.value,
    "\nDirect Binomial CDF:", p_value_binomial,
    "\nBeta function approach:", p_value_beta, "\n")
```

**Output:**
```
Exact binom.test P-value: 0.005138162 
Direct Binomial CDF: 0.005138162 
Beta function approach: 0.005138162
```


- **(A) Using `binom.test()`** internally uses a method equivalent to summing binomial probabilities or leveraging the incomplete Beta function.  
- **(B) Direct Binomial CDF** simply computes $$\sum_{i=0}^{k} \binom{n}{i} p_0^i (1-p_0)^{n-i}$$.  
- **(C) Beta CDF** exploits the link to the incomplete Beta function, showing explicitly that the same sum can be represented by $$I_{0.30}(12, 9)$$.

All three methods yield same results (minor differences can occur due to floating-point precision). If the p-value is below the chosen $$\alpha$$ (e.g., 0.05), we reject $$H_0$$ and conclude that the medication’s success rate is likely below 70%. Otherwise, we do not reject $$H_0$$.

---

## 2. Fisher’s Exact Test

### 2.1 When to Use Fisher’s Exact Test

While the **Exact Binomial Test** deals with a *single* proportion (or comparing it to a hypothesized value), **Fisher’s Exact Test** addresses a scenario of comparing *two* proportions in a contingency table. Suppose we have two groups (e.g., “Treatment” vs. “Control”) and each subject’s outcome is a success/failure. A $$2 \times 2$$ table might look like:

$$
\begin{array}{c|cc|c}
& \text{Success} & \text{Fail} & \text{Total} \\
\hline
\text{Group A} & a & b & a + b \\
\text{Group B} & c & d & c + d \\
\hline
\text{Total}   & a + c & b + d & n
\end{array}
$$

For small sample sizes or heavily unbalanced data, the usual chi-square test’s large-sample approximations can be unreliable. Fisher’s Exact Test does not rely on those approximations.

### 2.2 Core Principle and the Beta Link

Under $$H_0$$ (no difference in success probabilities across groups), the distribution of the table counts relates to a [hypergeometric](https://en.wikipedia.org/wiki/Hypergeometric_distribution) setup. Interestingly, if we isolate one group’s success count (say $$a$$ in Group A), conditional on the table’s margins, it follows a binomial distribution. This again means sums of binomial PMFs—and therefore **incomplete Beta functions**—can underpin the exact p-value calculation.

### 2.3 Fisher’s p-Value

For a two-sided Fisher’s Exact Test, the p-value is the sum of probabilities of *all* contingency tables at least as extreme as the observed one, according to a suitable measure of “extremity.” Symbolically:

$$
\text{p-value}
\;=\;
\sum_{\substack{\text{all tables } (a^*, b^*, c^*, d^*) \\ \text{as or more extreme}}}
P(A=a^*,\,B=b^*,\,C=c^*,\,D=d^*),
$$

where

$$
P(A=a^*, B=b^*, C=c^*, D=d^*)
=\;
\frac{
\binom{\,a^* + b^*}{\,a^*}\,\binom{\,c^* + d^*}{\,c^*}
}{
\binom{\,n}{\,a^* + c^*}
}.
$$

In practice, software automates summation over all relevant tables.

### 2.4 Example: Two Clinical Treatments

Imagine a simplified clinical trial scenario comparing two treatments, **T1** and **T2**, each tested on a small group of patients. Suppose our outcomes are recorded as “Success” or “Failure”:

|                  | Success | Failure | Total |
|------------------|---------|---------|-------|
| **Treatment T1** | 15      | 5       | 20    |
| **Treatment T2** | 10      | 10      | 20    |
| **Total**        | 25      | 15      | 40    |

We want to determine if the two treatments have significantly different success rates.

```{r}
table_2x2 <- matrix(c(15, 5, 10, 10), nrow = 2, byrow = TRUE)
fisher.test(table_2x2, alternative = "two.sided")

# Output:
# Fisher's Exact Test for Count Data
# data:  table_2x2
# p-value = 0.1908
# alternative hypothesis: true odds ratio is not equal to 1
# 95 percent confidence interval:
#   0.6613959 14.4997314
# sample estimates:
# odds ratio 
#   2.915714
```
A p-value of 0.1908  means that, assuming T1 and T2 truly have the same success probability, there is about a 19% chance of observing data this extreme. If the chosen significance level $$\alpha$$ is 0.05, we do not reject the null hypothesis. The 95% confidence interval for the odds ratio (approximately [0.66, 14.50]) also includes 1, suggesting we lack sufficient evidence to conclude a significant difference between the two treatments, even though the sample-based odds ratio is 2.92.

---

## 3. Where the Beta Distribution Shines

Both Exact Binomial and Fisher’s Exact rely (directly or indirectly) on the same binomial/Beta machinery seen in [Clopper–Pearson intervals]({% post_url 2025-01-16-beta-distribution-part2-1.md %}}):

- **Exactness**: They work with full binomial/hypergeometric probabilities—no normal approximations—yielding valid results even for small $$n$$ or extreme proportions.
- **Incomplete Beta Function**: Binomial PMFs map neatly to Beta CDFs, which are used to derive critical values or p-values.
- **Small-Sample Utility**: These methods are especially valuable when data are limited, where approximate methods can be misleading.

In **Bayesian** inference, the Beta distribution acts as a conjugate prior for binomial data. In the **frequentist** approach, the symmetric relationship between the CDF of binomial and Beta distributions bridges the discrte and continuous worlds, providing exact inferences for proportions and small samples.
underlies Clopper–Pearson intervals and exact tests. These **mathematical symmetries** unite both Bayesian and Frequentist's perspectives on proportion-based inference.

---

**References & Further Reading**  
- Casella, G. and Berger, R. (2002). [Statistical Inference](https://www.routledge.com/Statistical-Inference/Casella-Berger/p/book/9781032593036?srsltid=AfmBOop65n4g_y2w4_3cVrhSLYa02tUqcFEhz4L171Za_UPp47WgDv4O), 2nd ed. Duxbury.  
- Agresti, A. (2002). [Categorical Data Analysis](https://www.wiley.com/en-us/Categorical+Data+Analysis%2C+2nd+Edition-p-9780471458760), 2nd ed. John Wiley & Sons.  
- [Clopper–Pearson Intervals (Part 2.1)]({% post_url _posts/2025-01-16-beta-distribution-part2-1.md %})
