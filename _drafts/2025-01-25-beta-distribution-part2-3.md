---
title: "Beta Distribution: Part 2.2 - Frequentist Perspectives - Beta-Binomial Model"
date: 2025-01-16
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution]
math: true
---

In a previous post, I explored the Beta distribution from a frequentist perspective, focusing on its role in constructing exact confidence intervals for binomial proportions. This post delves deeper into the frequentist applications of the Beta distribution, particularly in addressing **over-dispersion** and **group-level variation**. We introduce the **Beta-Binomial model**, a two-stage process that accounts for variability in success probabilities across groups, and examine its implications in real-world scenarios like clinical research.

### Beta-Binomial Model for Over-Dispersion

The **Beta-Binomial model** extends the simple binomial framework by incorporating **over-dispersion**—a phenomenon where the observed variance exceeds what is expected under a standard binomial model. This is achieved through a two-stage process:

#### The Two-Stage Process
1. Draw a probability $$p$$ from a Beta distribution:
   $$
   p \sim \text{Beta}(\alpha, \beta),
   $$
   where $$\alpha$$ and $$\beta$$ are the shape parameters.
2. Given $$p$$, generate $$k$$ successes out of $$n$$ trials from a binomial distribution:
   $$
   k \mid p \sim \text{Binomial}(n, p).
   $$

The resulting marginal distribution of $$k$$ follows a **Beta-Binomial** distribution. We treat $$ \alpha $$ and $$ \beta $$ as model parameters that account for unobserved heterogeneity in the probability $$ p $$. This effectively inflates the variance of $$ k $$ relative to a simple $$ \mathrm{Binomial}(n, p) $$ model.

#### Frequentist Interpretation

From a frequentist perspective, the Beta-Binomial model allows for variability in the success probability $$p$$ across groups without assuming $$p$$ is a random variable (as in Bayesian frameworks). Key aspects include:

- **Parameter Estimation**: One can use likelihood-based methods (e.g., maximum likelihood estimation) to fit the Beta-Binomial model. Here, we do not treat $$ p $$ as a random variable in the Bayesian sense, but we do acknowledge that each sampled unit (or group) may have a different “true” $$ p $$.  
- **Over-Dispersion**: The model adjusts for variability not captured by the binomial model, offering a better fit when data exhibit higher variance than expected.

#### Common Use Cases

- **Group-Level Variation**: In clinical trials or manufacturing quality checks, each batch (group) may have its own success probability. Modeling that “batch effect” with Beta-Binomial can improve goodness-of-fit.  
- **Count Data with Extra Variance**: Any scenario where the binomial model systematically underestimates variance can be tested against a Beta-Binomial alternative.

### Case Study: Grouop-Level Variation and Over-Dispersion in Clinical Trials

In clinical trials, we often measure the response rate (success probability) across multiple sites or groups. A key assumption of the binomial model is that the success probability ($$p$$) is constant across all sites. However, in real-world settings, this assumption is frequently violated due to:

1. **Site-to-Site Variability**: Differences in patient demographics, site protocols, or environmental factors.
2. **Unobserved Heterogeneity**: Factors influencing $$p$$ that are not explicitly measured or modeled.

This variability leads to **over-dispersion**, where the observed variance in success rates exceeds what is predicted under a simple binomial model. The **Beta-Binomial model** accounts for this variability by allowing the success probability $$p$$ to vary across groups according to a Beta distribution.

#### The Model: Two-Stage Process

1. Each site's success probability $$p_i$$ is drawn from a Beta distribution:
   $$
   p_i \sim \text{Beta}(\alpha, \beta),
   $$
   where $$\alpha$$ and $$\beta$$ are shape parameters controlling the mean and variability of $$p$$.

2. Given $$p_i$$, the number of successes $$Y_i$$ at site $$i$$ follows a Binomial distribution:
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

#### Simulation: Site-Level Response Rates in Clinical Trials

We simulate a clinical trial with:
- $$n = 20$$ patients per site.
- $$30$$ sites.
- Success probabilities ($$p_i$$) varying across sites, drawn from $$\text{Beta}(\alpha = 2, \beta = 5)$$.

We compare:
1. **Binomial Model**: Assumes a constant $$p$$ (mean of the Beta distribution).
2. **Beta-Binomial Model**: Incorporates variability in $$p$$ across sites.

The simulation is done in R, and we visualize the response counts across sites and the distribution as following:

{::options parse_block_html="true" /}
<details><summary markdown="span">R Code for Simulation - Click to Expand</summary>

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
```
</details>

{::options parse_block_html="false" /}

<img src="assets/img/2025-02-02-beta-distribution-part2-2/Response_vs_site.png" alt="Response Counts Across Sites" style="display: block; margin: 0 auto; width: 70%; height: auto;" />

To have a look at the site-specific response probabilities, we plot the histogram of $$p$$ (Beta distribution) as following:

{::options parse_block_html="true" /}
<details><summary markdown="span">R Code for Plotting Site-specific Response Probabilities - Click to Expand</summary>

```r
beta_df <- data.frame(p = site_probs)
ggplot(beta_df, aes(x = p)) +
  geom_histogram(bins = 15, fill = "blue", alpha = 0.6) +
  labs(title = "Site-Specific Response Probabilities (Beta Distribution)",
       x = "Probability of Response (p)",
       y = "Frequency") +
  theme_minimal()
```
</details>

{::options parse_block_html="false" /}

<img src="assets/img/2025-02-02-beta-distribution-part2-2/Site_specific_response.png" alt="Site-Specific Response Probabilities" style="display: block; margin: 0 auto; width: 70%; height: auto;" >

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

### Summary

The Beta distribution models the variability of success probabilities $$p$$ across groups, while the Beta-Binomial model accounts for over-dispersion by combining the Beta and Binomial distributions. This post demonstrates how the Beta-Binomial model inflates variance compared to a simple Binomial model, reflecting real-world heterogeneity in data. The shape parameters $$\alpha$$ and $$\beta$$ of the Beta distribution control the mean and variability of $$p$$, allowing for more accurate modeling of group-level variation and extra-binomial variability in frequentist contexts.

### References

Key concepts and properties of the Beta-Binomial model are based on:
- [Statistical Inference 2nd Edition - Page 161](https://www.routledge.com/Statistical-Inference/Casella-Berger/p/book/9781032593036?srsltid=AfmBOorWRaRL6Hh8DUN9waZP0yR68HQIMi0SrgTZZz-nHZr1UxQTjZFJ) by **George Casella**, **Roger L. Berger** (2024), CRC Press
- [Beta-binomial distribution](https://en.wikipedia.org/wiki/Beta-binomial_distribution) - Wikipedia
- [Chapter 3: The Beta-Binomial Bayesian Model - STAT 535 Lecture Notes](https://people.stat.sc.edu/Hitchcock/stat535slides3BRBhandout.pdf) by **David B. Hitchcock** (2022) - University of South Carolina

The discussion on over-dispersion and simulation is inspired by:  

- [Simulate data from the beta-binomial distribution in SAS](https://blogs.sas.com/content/iml/2017/11/20/simulate-beta-binomial-sas.html) by **Rick Wicklin** (2017) - The DO Loop blog
- [Everything You Don't Need to Know About Variance Inflation Factors](https://www.blasbenito.com/post/variance-inflation-factor/) by **Blas M. Benito, PhD** - blasbenito.com
- [Dispersion and binomial vs beta-binomial](https://discourse.mc-stan.org/t/dispersion-and-binomial-vs-beta-binomial/29879) by **adamConnerSax, et al.** (2022) - The Stan Forums
