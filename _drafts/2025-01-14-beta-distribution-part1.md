---
title: "Beta Distribution: Part 1 - Intuition and Properties"
date: 2025-01-14
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution, beysian-statistics]
math: true
image:
  path: assets/headers/
  alt: Beta Distribution
---

This post kicks off a series on the Beta distribution, a versatile probability distribution often used in statistics and machine learning. In this first part, I will develop an intuitive understanding of the Beta distribution and examine its fundamental properties.

## General Properties of the Beta Distribution

The Beta distribution is defined on the interval $$[0,1]$$. This makes it particularly convenient for modeling quantities naturally constrained between 0 and 1—such as probabilities and proportions. It is governed by two **shape parameters**, $$\alpha$$ and $$\beta$$, which determine how the distribution is skewed:

- $$\alpha$$: Determines how heavily the distribution is weighted toward “success.”
- $$\beta$$: Determines how heavily the distribution is weighted toward “failure.”

### Probability Density Function (PDF)

The PDF of the Beta distribution is given by

$$
\text{Beta}(x \mid \alpha, \beta) = \frac{x^{\alpha - 1}(1 - x)^{\beta - 1}}{B(\alpha, \beta)},
$$

where $$B(\alpha, \beta)$$ is the Beta function, a normalizing constant ensuring the PDF integrates to 1.

### Shape of the Curve

Here’s how changes in $$\alpha$$ and $$\beta$$ affect the shape of the distribution:

- $$\alpha = \beta = 1$$: Uniform distribution on $$[0,1]$$.
- $$\alpha > \beta$$: Skewed toward 1 (i.e., “success”).
- $$\alpha < \beta$$: Skewed toward 0 (i.e., “failure”).
- $$\alpha, \beta > 1$$: Bell-shaped.
- $$\alpha, \beta < 1$$: U-shaped curve.

A helpful interpretation is to view the Beta distribution as a way of encoding different levels of “success” vs. “failure” weights:

$$
p^{\alpha - 1} (1-p)^{\beta - 1}.
$$

- **No prior information**: $$\alpha = \beta = 1$$ yields a uniform distribution.
- **Heavier weight on success** ($$\alpha > \beta$$): The distribution skews toward 1.
- **Heavier weight on failure** ($$\alpha < \beta$$): The distribution skews toward 0.

![Shape of the Curves of Beta Distribution](assets/img/2025-01-14-beta-distribution-part1/Beta_curve_bell_vs_u.png)

We can also evaluate the **mean (expected value)** of the Beta distribution, which helps us interpret its shape and deepen our quantitative understanding of how $$\alpha$$ and $$\beta$$ influence the distribution:

$$ 
\begin{align*}
  \mu = \mathbf{E}[X] &= \int_{0}^{1} x \, \frac{x^{\alpha - 1} (1 - x)^{\beta - 1}}{B(\alpha, \beta)} \, dx \\
      &= \frac{\alpha}{\alpha + \beta} = \frac{1}{1 + \frac{\beta}{\alpha}}.
\end{align*} 
$$

This shows that:

- As $$\frac{\beta}{\alpha} \to 0$$, the mean $$\mu \to 1$$.
- As $$\frac{\beta}{\alpha} \to \infty$$, the mean $$\mu \to 0$$.

Hence, when $$\alpha$$ is much larger than $$\beta$$, the distribution concentrates near 1, and vice versa.

## Beta as a Conjugate Prior in Bayesian Inference

A key feature of the Beta distribution is that it serves as a **conjugate prior** for the Bernoulli, binomial, and other distributions in the exponential family. Conjugacy means that if we start with a Beta prior for a parameter $$p$$ (probability of success) and observe binomial data, the **posterior** for $$p$$ will also be a Beta distribution—just with updated parameters.

### Conjugacy with the Bernoulli/Binomial Likelihood

Consider a Binomial likelihood:

$$
k \mid p \sim \text{Binomial}(n, p),
$$

and let the prior for $$p$$ be

$$
p \sim \text{Beta}(\alpha, \beta).
$$

After observing $$k$$ successes in $$n$$ trials, the posterior for $$p$$ is

$$
p \mid k \sim \text{Beta}(\alpha + k, \beta + n - k).
$$

Because the posterior remains a Beta distribution, we say that the Beta is a conjugate prior for Bernoulli/binomial likelihoods.

### Numeric Example

Suppose we had a prior belief that the success rate is about $$20\%$$, based on historical data with 2 successes and 8 failures. Then we observe 5 successes and 5 failures. In Bayesian inference:

1. **Prior**: $$\text{Beta}(\alpha_0 = 2, \beta_0 = 8)$$  
2. **New Data**: $$k=5$$ successes, $$n-k=5$$ failures.  
3. **Posterior**: $$\text{Beta}(\alpha_0 + 5, \beta_0 +5) = \text{Beta}(\alpha_1 = 7, \beta_1 = 13)$$

The new posterior mean is

$$
\frac{\alpha_1}{\alpha_1 + \beta_1} = \frac{7}{7+13} = \frac{7}{20} = 0.35 \quad (35\%).
$$

This updated belief logically falls between the **prior** mean (20%) and the **naive** sample estimate of 50%.

<img src="assets/img/2025-01-14-beta-distribution-part1/prior_vs_posterior.png" alt="Prior vs. Posterior" style="display: block; margin: 0 auto; width: 80%; height: auto;">

## Iterative (Online) Updating

One of the most powerful features of Bayesian methods is **iterative (or online) updating**. In reality, data often arrives in batches over time, rather than all at once. When modeling a Bernoulli process with a Beta prior, each new set of observations updates the posterior, which then becomes the new prior for subsequent data.

### How It Works

1. Begin with $$\text{Beta}(\alpha_0, \beta_0)$$.
2. Observe $$n$$ Bernoulli trials with $$k$$ successes.
3. Update the prior to obtain the posterior:

   $$
   \mathrm{Beta}\bigl(\alpha_0 + k,\;\beta_0 + (n - k)\bigr).
   $$

4. The new posterior can be used as the prior for the next incoming batch of data.

### Example: Iterative Updating Simulation

Below is a simulation of iterative updating for a Bernoulli process with a true success probability $$p_{\text{true}} = 0.7$$. We repeatedly observe small batches of data, update our Beta prior, and watch the posterior converge to $$0.7$$.

<img src="assets/img/2025-01-14-beta-distribution-part1/Iterative_Bayesian.png" alt="Iterative Bayesian" style="display: block; margin: 0 auto; width: 80%; height: auto;">

- **The black curve** is the initial $$\text{Beta}(\alpha_0, \beta_0)$$.
- **Colored curves** show updated posteriors as more data arrives.
- The dashed vertical line at $$p_{\text{true}} = 0.7$$ highlights how the posterior concentrates around the true value over time.

## Beta Distribution in Frequentist Inference

While the Beta distribution is often highlighted for its role as a conjugate prior in Bayesian statistics, it also has important connections to classical (frequentist) inference—particularly in dealing with **proportions** and **binomial data**. Below are two key examples of how the Beta distribution appears in a purely frequentist context.

### Confidence Intervals for Proportions (Clopper–Pearson)

When we observe a binomial outcome—say, $$ k $$ successes out of $$ n $$ trials—the most straightforward estimate of the underlying probability of success $$ p $$ is the sample proportion $$ \hat{p} = k/n $$. Constructing a confidence interval for $$ p $$ can be done in various ways, including the **Wald interval**, **Wilson interval**, and others. However, one of the classic “exact” methods is the **Clopper–Pearson interval**, which directly leverages the Beta distribution.

#### The Clopper–Pearson Interval

Given observed data $$ (k, n) $$, the **Clopper–Pearson** $$ 100(1 - \alpha)\% $$ confidence interval for $$ p $$ is given by:

$$
\Bigl( p_{\text{lower}}, \; p_{\text{upper}} \Bigr),
$$

where

$$
p_{\text{lower}} = 
\begin{cases}
0, & \text{if } k = 0, \\
\mathrm{BetaInv}\bigl(\tfrac{\alpha}{2};\, k,\; n-k+1\bigr), & \text{if } 0 < k < n, \\
\end{cases}
$$

$$
p_{\text{upper}} = 
\begin{cases}
1, & \text{if } k = n, \\
\mathrm{BetaInv}\bigl(1 - \tfrac{\alpha}{2};\, k+1,\; n-k\bigr), & \text{if } 0 < k < n, \\
\end{cases}
$$

$$ \mathrm{BetaInv}\bigl(p; \alpha, \beta\bigr) $$ denotes the **inverse** of the cumulative distribution function (CDF) of the  $$ \mathrm{Beta}(\alpha, \beta) $$ distribution. In many statistical software packages, this is referred to as `qbeta()` or an equivalent function.

The Clopper–Pearson interval is considered “exact” in the sense that the coverage probability is guaranteed to be at least $$ (1 - \alpha) $$ for every possible true $$ p $$. This comes at the cost of sometimes being more conservative (wider) than other approximate intervals (e.g., the Wilson interval).

#### Relationship to the Beta Distribution

##### 1. Binomial Cumulative Probabilities

Given a Binomial $$(n, p)$$  random variable  $$X$$ , the probability of seeing *up to*  $$k$$  successes is

$$
P(X \le k) \;=\; \sum_{i=0}^{k} \binom{n}{i}\, p^{i}\,(1 - p)^{n - i}.
$$

In the **Clopper–Pearson** method, we want to find the values of  $$p$$  (the true success probability) for which observing  $$k$$  successes is *not too surprising*. Concretely, we ensure  $$p$$  satisfies a chosen significance threshold ( $$\alpha$$ ) on both the lower and upper tails.

##### 2. Constructing the Interval by “Slicing”

To build a confidence interval, we solve for  $$p_{\text{lower}}$$  and  $$p_{\text{upper}}$$  such that:

- The lower bound  $$p_{\text{lower}}$$  is where the cumulative probability  $$P(X \le k-1)$$  just hits  $$\alpha/2$$  (or similarly where  $$P(X \ge k)$$  is  $$\alpha/2$$ ).  
- The upper bound  $$p_{\text{upper}}$$  is found by setting  $$P(X \le k)$$  equal to  $$1 - \alpha/2$$ .

Mathematically, we solve $p$ for:

$$
\sum_{i=0}^{k} \binom{n}{i} p^i (1-p)^{n-i} = \frac{\alpha}{2} \quad \text{or} \quad \sum_{i=k}^{n} \binom{n}{i} p^i (1-p)^{n-i} = \frac{\alpha}{2},
$$

Intuitively, imagine **scanning** through  $$p \in [0,1]$$ . For each candidate $$p$$ , we measure:

$$
\sum_{i=0}^{k} \binom{n}{i}\, p^{i}\,(1 - p)^{n - i}.
$$

This value tells us how likely it is to see  $$k$$  or fewer successes if  $$p$$  were the true success rate. In **Clopper–Pearson**, we identify precisely which  $$p$$ -values make that cumulative probability match our chosen cutoffs ($$\alpha/2$$  for the lower tail,  $$1-\alpha/2$$  for the upper tail). By “slicing” the binomial distribution *exactly* at these points, we form the boundaries of our confidence interval—no normal approximation required—leading to “exact” coverage in repeated sampling.

##### 3. From Binomial Sums to the Incomplete Beta Function

A key step is recognizing that:

$$
\sum_{i=0}^{k} \binom{n}{i}\, p^{i}\,(1 - p)^{n - i}
\;=\;
1 - I_{p}(k + 1,\, n - k),
$$

where  $$I_{p}(\alpha,\beta)$$  is the **regularized incomplete Beta** function which is the CDF of Beta distribution, defined by

$$
I_{p}(\alpha,\beta)
\;=\;
\frac{1}{B(\alpha,\beta)}
\int_{0}^{p}
t^{\alpha - 1}\,(1 - t)^{\beta - 1}\,dt,
$$
and  $$B(\alpha,\beta)$$  is the complete Beta function. Essentially, **summing** binomial probabilities can be recast as **integrating** a function of the form  $$t^{\alpha-1}(1-t)^{\beta-1}$$ . This leads to the **CDF of Beta distribution**.   

Therefore solving $p_{\text{lower}}$ for equation $$\sum_{i=0}^{k} \binom{n}{i} p^i (1-p)^{n-i} = \frac{\alpha}{2}$$ is equivalent to solving a **Beta quantile** as for:

$$
I_{p}(k + 1,\, n - k) = 1 - \frac{\alpha}{2} \quad \text{or} \quad \mathbb{P}\left(p > p_{\text{lower}} \,\mid \, p \sim \text{Beta}(k+1, n-k)\right) = \frac{\alpha}{2}
$$

##### 4. Why the Beta Distribution?

Binomial expressions contain terms like  $$p^{k}(1-p)^{n-k}$$ . When we integrate or “accumulate” these terms over  $$[0,1]$$ , we naturally arrive at the Beta family of functions as discussed above. In the Bayesian context we discused previously, this is often described as “conjugacy” but even in a frequentist procedure like Clopper–Pearson, the same Beta integrals emerge once we:

1. Convert binomial cumulative sums to an integral expression, and  
2. Invert that integral to solve for  $$p$$ .

Hence, the Beta distribution acts as a continuous bridge between discrete binomial counts and probabilities in  $$[0,1]$$ . By pinpointing the exact **Beta quantiles** that correspond to certain binomial tail probabilities, we avoid normal approximations altogether. This gives Clopper–Pearson its **exact** coverage guarantee—even in scenarios with small  $$n$$  or extreme  $$p$$ .

In short, **integrating** binomial probabilities leads us to Beta functions; **inverting** those Beta integrals produces the interval bounds. This tight connection is what powers the Clopper–Pearson interval—ensuring it captures the true  $$p$$  at least  $$(1 - \alpha) \times 100\%$$  of the time, *no matter* the sample size or the true  $$p$$ .

#### Simulation: Overall Coverage and Interval Width

To illustrate how the Clopper–Pearson interval behaves compared to other intervals (e.g., Wald, Wilso), I run a quick simulation in R. It works like following:

- In each of $$N_{\text{sims}}$$ runs, I simulate a single binomial outcome $$k$$ from $$\mathrm{Binomial}(n, p_{\text{true}})$$.
- Calculatre intervals:
  1. **Clopper–Pearson (Exact):** Obtained using `binom.confint(..., methods = "exact")`.  
  2. **Wald (Naive):** Calculated by $$\hat{p} \pm z \sqrt{\frac{\hat{p}\,(1 - \hat{p})}{n}}$$.
- Evaluate each method’s **coverage** (the fraction of intervals that contain $$p_{\text{true}}$$) and **width**, and compare the results across both methods.

Results of coverages show that Clopper–Pearson interval achieves the desired coverage probability of $$ 1 - \alpha $$, while the Wald interval can be slightly lower. 
```
Coverage (Clopper–Pearson): 0.957 
Coverage (Wald):            0.939
```
Comparision of interval widths shows that Clopper–Pearson intervals tend to be wider than Wald intervals, reflecting the trade-off between exact coverage and interval width.

<img src="assets/img/2025-01-14-beta-distribution-part1/Interval_width_Clopper_vs_Wald.png" alt="Distribution of Interval Width" style="display: block; margin: 0 auto; width: 80%; height: auto;">

<details>
<summary>R Code for Comparison of Interval Widths - Click to Expand</summary>

{% highlight r %}
library(binom)      
library(ggplot2)    

set.seed(123)

p_true  <- 0.3    
n       <- 100  
N_sims  <- 1000 
alpha   <- 0.05

cp_lower  <- numeric(N_sims)  
cp_upper  <- numeric(N_sims)  
wald_lower <- numeric(N_sims) 
wald_upper <- numeric(N_sims) 
p_hat_vals <- numeric(N_sims) 

for(i in 1:N_sims) {
  k <- rbinom(1, size = n, prob = p_true)
  
  p_hat <- k / n
  p_hat_vals[i] <- p_hat
  
  cp_int <- binom.confint(x = k, n = n, conf.level = 1 - alpha, methods = "exact")
  cp_lower[i] <- cp_int$lower
  cp_upper[i] <- cp_int$upper
  
  z_val <- qnorm(1 - alpha/2)  # ~ 1.96 for alpha=0.05
  se_hat <- sqrt(p_hat * (1 - p_hat) / n)
  wald_lower[i] <- max(0, p_hat - z_val * se_hat)
  wald_upper[i] <- min(1, p_hat + z_val * se_hat)
}

# Coverage: fraction of intervals that contain p_true
cp_coverage   <- mean(p_true >= cp_lower & p_true <= cp_upper)
wald_coverage <- mean(p_true >= wald_lower & p_true <= wald_upper)

cat("Coverage (Clopper–Pearson):", cp_coverage, "\n")
cat("Coverage (Wald):           ", wald_coverage, "\n")

cp_widths   <- cp_upper   - cp_lower
wald_widths <- wald_upper - wald_lower

df_widths <- data.frame(
  Method = factor(rep(c("Clopper–Pearson", "Wald"), each = N_sims)),
  Width  = c(cp_widths, wald_widths)
)

# Histogram/frequency polygon to compare widths side-by-side
ggplot(df_widths, aes(x = Width, fill = Method)) +
  geom_histogram(alpha = 0.4, position = "identity", bins = 30) +
  labs(title = "Distribution of Interval Widths",
       x = "Interval Width", 
       y = "Frequency") +
  theme_minimal()
{% endhighlight %}

</details>

#### Simulation: Coverage vs. Sample Size

Another simulation we can do is to **measure coverage across different sample sizes**. To do this, I fix a true probability of success, say $$p_{\text{true}} = 0.3$$, and vary $$n$$ (for example, $5, 10, 30, 50, 100, 500$). For each value of $$n$$, I simulate a large number of binomial experiments, compute both Clopper–Pearson and Wald confidence intervals, and then calculate how often these intervals contain the true $$p_{\text{true}}$$. By plotting **coverage** against the sample size $$n$$, we can visualize how each method performs as we collect more data.

<img src="assets/img/2025-01-14-beta-distribution-part1/Coverage_n_Clopper_vs_Wald.png" alt="Coverage vs. n" style="display: block; margin: 0 auto; width: 80%; height: auto;">

As shown with simulated data, Clopper–Pearson remains at or above 95% coverage (reflecting its “exact” nature), while Wald intervals can dip below 95% for smaller sample sizes. Seeing how both curves evolve as $$n$$ grows helps us understand the **trade-off** between accuracy and interval width for each method.

<details>
<summary>R Code for Coverage vs. n - Click to Expand</summary>

{% highlight r %}
library(binom)
library(ggplot2)

set.seed(42)

p_true    <- 0.3
n_values  <- c(5, 10, 30, 50, 100, 500)  # Example range
N_sims    <- 2000
alpha     <- 0.05

results <- data.frame(
  n = integer(),
  method = character(),
  coverage = numeric()
)

for (n in n_values) {
  cp_cover <- 0
  wald_cover <- 0
  
  for (i in seq_len(N_sims)) {
    k <- rbinom(1, size = n, prob = p_true)
    p_hat <- k / n
    
    # Clopper-Pearson
    cp_int <- binom.confint(x = k, n = n, methods = "exact", conf.level = 1 - alpha)
    cp_lower <- cp_int$lower
    cp_upper <- cp_int$upper
    cp_cover <- cp_cover + (p_true >= cp_lower & p_true <= cp_upper)
    
    # Wald
    z_val <- qnorm(1 - alpha/2)
    se_hat <- sqrt(p_hat*(1 - p_hat)/n)
    wald_lower <- max(0, p_hat - z_val*se_hat)
    wald_upper <- min(1, p_hat + z_val*se_hat)
    wald_cover <- wald_cover + (p_true >= wald_lower & p_true <= wald_upper)
  }
  
  results <- rbind(results, data.frame(n = n,
                                       method = "Clopper-Pearson",
                                       coverage = cp_cover / N_sims))
  results <- rbind(results, data.frame(n = n,
                                       method = "Wald",
                                       coverage = wald_cover / N_sims))
}

ggplot(results, aes(x = factor(n), y = coverage, color = method, group = method)) +
  geom_line(aes(linetype = method)) +
  geom_point() +
  geom_hline(yintercept = 0.95, linetype = "dashed") +
  labs(title = "Coverage vs. n (p_true = 0.3)",
       x = "Number of Trials (n)",
       y = "Coverage") +
  theme_minimal()
{% endhighlight %}

</details>

### Beta-Binomial Model for Over-Dispersion

Another area where the Beta distribution shows up in frequentist analysis is the **Beta-Binomial model**, which can be thought of as a “two-stage” binomial process that introduces **over-dispersion**—a phenomenon where observed variance is higher than expected under a simple binomial assumption.


#### The Two-Stage Process

1. **Stage 1**: Draw a probability $$ p $$ from a Beta distribution, say $$ \mathrm{Beta}(\alpha, \beta) $$.
2. **Stage 2**: Given that $$ p $$, generate $$ k $$ successes out of $$ n $$ trials from a binomial distribution $$ \mathrm{Binomial}(n, p) $$.

Mathematically, $$ k $$ follows a **Beta-Binomial** distribution. From a purely frequentist standpoint, we treat $$ \alpha $$ and $$ \beta $$ as **model parameters** that account for unobserved heterogeneity in the probability $$ p $$. This effectively inflates the variance of $$ k $$ relative to a simple $$ \mathrm{Binomial}(n, p) $$ model.

#### Frequentist Interpretation

- **Parameter Estimation**: One can use likelihood-based methods (e.g., maximum likelihood estimation) to fit the Beta-Binomial model. Here, we do not treat $$ p $$ as a random variable in the Bayesian sense, but we do acknowledge that each sampled unit (or group) may have a different “true” $$ p $$.  
- **Over-Dispersion**: If a simple binomial model does not capture all the variability in our data—for example, if we see a higher spread in the number of successes than the binomial assumption would predict—fitting a Beta-Binomial model can account for that extra variability without moving to a fully Bayesian framework.

#### Common Use Cases

- **Group-Level Variation**: In clinical trials or manufacturing quality checks, each batch (group) may have its own success probability. Modeling that “batch effect” with Beta-Binomial can improve goodness-of-fit.  
- **Count Data with Extra Variance**: Any scenario where the binomial model systematically underestimates variance can be tested against a Beta-Binomial alternative.


### Practical Tips and Software

- **R Functions**:  
  - `binom::binom.confint()` in R offers multiple methods for confidence intervals of proportions, including Clopper–Pearson.  
  - The **Beta-Binomial** model can be fit in R via packages like `VGAM` (function `vglm()` with family `betabinomial`).

- **Python**:  
  - Statsmodels and other libraries offer “exact” binomial intervals.  
  - For Beta-Binomial modeling, one can use `scipy.stats.betabinom` for probability mass function (PMF) and cumulative distribution function (CDF) calculations, though advanced fitting functions may require additional libraries or custom coding.

- **Interpretation**:  
  - In both the Clopper–Pearson and Beta-Binomial contexts, the Beta distribution provides the “shape” governing how proportions or probabilities vary.  
  - Even outside Bayesian methods, it remains a fundamental building block in understanding binomial variability and constructing exact inference procedures.


