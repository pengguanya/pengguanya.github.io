---
title: "Beta Distribution: Part 2 - Frequentist Perspectives"
date: 2025-01-16
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution, freqeuntist-statistics]
math: true
---

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

{::options parse_block_html="true" /}

<details><summary markdown="span">R Code for Distribution of Interval Width - Click to Expand</summary>

```r
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
```

</details>
<br/>

{::options parse_block_html="false" /}


#### Simulation: Coverage vs. Sample Size

Another simulation we can do is to **measure coverage across different sample sizes**. To do this, I fix a true probability of success, say $$p_{\text{true}} = 0.3$$, and vary $$n$$ (for example, $5, 10, 30, 50, 100, 500$). For each value of $$n$$, I simulate a large number of binomial experiments, compute both Clopper–Pearson and Wald confidence intervals, and then calculate how often these intervals contain the true $$p_{\text{true}}$$. By plotting **coverage** against the sample size $$n$$, we can visualize how each method performs as we collect more data.

<img src="assets/img/2025-01-14-beta-distribution-part1/Coverage_n_Clopper_vs_Wald.png" alt="Coverage vs. n" style="display: block; margin: 0 auto; width: 80%; height: auto;">

As shown with simulated data, Clopper–Pearson remains at or above 95% coverage (reflecting its “exact” nature), while Wald intervals can dip below 95% for smaller sample sizes. Seeing how both curves evolve as $$n$$ grows helps us understand the **trade-off** between accuracy and interval width for each method.

{::options parse_block_html="true" /}

<details><summary markdown="span">R Code for Coverage vs. n - Click to Expand</summary>

```r
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
```

</details>
<br/>

{::options parse_block_html="false" /}

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

### Case Study: Grouop-Level Variation and Over-Dispersion in Clinical Trials - An Application of Beta-Binomial Model

In clinical trials, we often measure the **response rate** (success probability) across multiple sites or groups. A key assumption of the **binomial model** is that the success probability ($$p$$) is constant across all sites. However, in real-world settings, this assumption is frequently violated due to:

1. **Site-to-Site Variability**: Differences in patient demographics, site protocols, or environmental factors.
2. **Unobserved Heterogeneity**: Factors influencing $$p$$ that are not explicitly measured or modeled.

This variability leads to **over-dispersion**, where the observed variance in success rates exceeds what is predicted under a simple binomial model. The **Beta-Binomial model** accounts for this variability by allowing the success probability $$p$$ to vary across groups according to a **Beta distribution**.

---

### The Model: Connecting Beta and Binomial

#### Two-Stage Process

1. **Stage 1**: Each site's success probability $$p_i$$ is drawn from a Beta distribution:
   $$
   p_i \sim \text{Beta}(\alpha, \beta),
   $$
   where $$\alpha$$ and $$\beta$$ are shape parameters controlling the mean and variability of $$p$$.

2. **Stage 2**: Given $$p_i$$, the number of successes $$Y_i$$ at site $$i$$ follows a Binomial distribution:
   $$
   Y_i \mid p_i \sim \text{Binomial}(n, p_i),
   $$
   where $$n$$ is the number of trials (e.g., patients) at each site.

#### Variance Inflation and Over-Dispersion

The marginal distribution of $$Y_i$$ (after integrating out $$p$$) is **Beta-Binomial**:
$$
P(Y_i = k) = \binom{n}{k} \frac{B(k + \alpha, n - k + \beta)}{B(\alpha, \beta)},
$$
where $$B(\alpha, \beta)$$ is the Beta function.

The variance of $$Y_i$$ under the Beta-Binomial model is:
$$
\text{Var}(Y_i) = n \cdot \mu \cdot (1 - \mu) \cdot \left(1 + \frac{n - 1}{\alpha + \beta + 1}\right),
$$
where:
- $$\mu = \frac{\alpha}{\alpha + \beta}$$ is the mean of the Beta distribution.
- The term $$\frac{n - 1}{\alpha + \beta + 1}$$ accounts for variability in $$p$$, inflating the variance compared to the Binomial model.

In contrast, the variance of $$Y_i$$ under the Binomial model is:
$$
\text{Var}_{\text{Binomial}}(Y_i) = n \cdot \mu \cdot (1 - \mu).
$$

The **over-dispersion factor** quantifies the additional variability:
$$
\text{OD} = \frac{\text{Var}_{\text{Beta-Binomial}}(Y_i)}{\text{Var}_{\text{Binomial}}(Y_i)} = 1 + \frac{n - 1}{\alpha + \beta + 1}.
$$

---

### Simulation: Site-Level Response Rates in Clinical Trials

#### Scenario

We simulate a clinical trial with:
- $$n = 20$$ patients per site.
- $$30$$ sites.
- Success probabilities ($$p_i$$) varying across sites, drawn from $$\text{Beta}(\alpha = 2, \beta = 5)$$.

We compare:
1. **Binomial Model**: Assumes a constant $$p$$ (mean of the Beta distribution).
2. **Beta-Binomial Model**: Incorporates variability in $$p$$ across sites.

---

#### R Code for Simulation

```r
library(VGAM)
library(ggplot2)

# Parameters
n_patients <- 20  # Patients per site
n_sites <- 30     # Number of sites
alpha <- 2        # Beta shape parameter 1
beta <- 5         # Beta shape parameter 2

# Simulate site-specific probabilities from Beta distribution
site_probs <- rbeta(n_sites, shape1 = alpha, shape2 = beta)

# Generate response counts per site using Binomial and Beta-Binomial models
binomial_counts <- rbinom(n_sites, size = n_patients, prob = mean(site_probs))
beta_binomial_counts <- sapply(site_probs, function(p) rbinom(1, size = n_patients, prob = p))

# Variance comparison
binom_var <- var(binomial_counts)
beta_binom_var <- var(beta_binomial_counts)

cat("Variance (Binomial):", binom_var, "\n")
cat("Variance (Beta-Binomial):", beta_binom_var, "\n")

# Plot: Response counts across sites
df <- data.frame(
  Site = rep(1:n_sites, 2),
  Responses = c(binomial_counts, beta_binomial_counts),
  Model = rep(c("Binomial", "Beta-Binomial"), each = n_sites)
)

ggplot(df, aes(x = factor(Site), y = Responses, fill = Model)) +
  geom_bar(stat = "identity", position = "dodge") +
  labs(title = "Response Counts Across Sites",
       x = "Site",
       y = "Number of Responses",
       fill = "Model") +
  theme_minimal()

# Plot: Variability in site-specific probabilities
beta_df <- data.frame(p = site_probs)
ggplot(beta_df, aes(x = p)) +
  geom_histogram(bins = 15, fill = "blue", alpha = 0.6) +
  labs(title = "Site-Specific Response Probabilities (Beta Distribution)",
       x = "Probability of Response (p)",
       y = "Frequency") +
  theme_minimal()
```

---

#### Insights from Simulation

1. **Variance Inflation**:
   - The variance of Beta-Binomial response counts is significantly higher than that of the Binomial model, reflecting the impact of site-to-site variability:
     $$
     \text{OD} = 1 + \frac{n - 1}{\alpha + \beta + 1}.
     $$
     For $$\alpha = 2, \beta = 5, n = 20$$, $$\text{OD} = 3.375$$.

2. **Heterogeneity in $$p$$**:
   - The histogram of $$p$$ (Beta distribution) illustrates how site-specific variability influences the response probabilities. Sites with higher $$p$$ produce more successes, while those with lower $$p$$ produce fewer, contributing to over-dispersion.

3. **Real-World Implications**:
   - Ignoring over-dispersion (by assuming a constant $$p$$) underestimates the variance, leading to overconfident conclusions about treatment effects.
   - The Beta-Binomial model captures this variability, improving model fit and inference.

---

### Takeaways

1. The **Beta-Binomial model** provides a robust framework to account for over-dispersion in clinical trials caused by unobserved heterogeneity.
2. By allowing $$p$$ to vary across sites, it captures real-world variability, leading to more realistic uncertainty quantification.
3. Simulations not only validate theoretical insights but also help visualize the practical impact of using the Beta-Binomial model over the Binomial model.

This example ties back to the **Beta distribution’s role in frequentist inference**, showcasing its power to address challenges in real-world data analysis.



