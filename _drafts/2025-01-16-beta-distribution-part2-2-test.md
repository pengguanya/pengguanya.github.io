---
title: "Beta Distribution: Part 2.2 - Frequentist Perspectives - Beta-Binomial Model"
date: 2025-01-16
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution]
math: true
---

In previous post, I have explored the Beta distribution from a frequentist perspective, focusing on its role in constructing exact confidence intervals for binomial proportions. In this post, I delve deeper into the frequentist applications of the Beta distribution, particularly in the context of **over-dispersion** and **group-level variation**. We introduce the **Beta-Binomial model**, a two-stage process that accounts for variability in success probabilities across groups, and discuss its implications in clinical trials and other real-world scenarios. 

### Beta-Binomial Model for Over-Dispersion

Another area where the Beta distribution shows up in frequentist analysis is the **Beta-Binomial model**, which can be thought of as a “two-stage” binomial process that introduces **over-dispersion**—a phenomenon where observed variance is higher than expected under a simple binomial assumption.


#### The Two-Stage Process

1. Draw a probability $$ p $$ from a Beta distribution, say $$ \mathrm{Beta}(\alpha, \beta) $$.
2. Given that $$ p $$, generate $$ k $$ successes out of $$ n $$ trials from a binomial distribution $$ \mathrm{Binomial}(n, p) $$.

Mathematically, $$ k $$ follows a **Beta-Binomial** distribution. From a purely frequentist standpoint, we treat $$ \alpha $$ and $$ \beta $$ as model parameters that account for unobserved heterogeneity in the probability $$ p $$. This effectively inflates the variance of $$ k $$ relative to a simple $$ \mathrm{Binomial}(n, p) $$ model.

#### Frequentist Interpretation

- **Parameter Estimation: One can use likelihood-based methods (e.g., maximum likelihood estimation) to fit the Beta-Binomial model. Here, we do not treat $$ p $$ as a random variable in the Bayesian sense, but we do acknowledge that each sampled unit (or group) may have a different “true” $$ p $$.  
- **Over-Dispersion**: If a simple binomial model does not capture all the variability in our data—for example, if we see a higher spread in the number of successes than the binomial assumption would predict—fitting a Beta-Binomial model can account for that extra variability without moving to a fully Bayesian framework.

#### Common Use Cases

- **Group-Level Variation**: In clinical trials or manufacturing quality checks, each batch (group) may have its own success probability. Modeling that “batch effect” with Beta-Binomial can improve goodness-of-fit.  
- **Count Data with Extra Variance**: Any scenario where the binomial model systematically underestimates variance can be tested against a Beta-Binomial alternative.

### Case Study: Grouop-Level Variation and Over-Dispersion in Clinical Trials

In clinical trials, we often measure the response rate (success probability) across multiple sites or groups. A key assumption of the binomial model is that the success probability ($$p$$) is constant across all sites. However, in real-world settings, this assumption is frequently violated due to:

1. **Site-to-Site Variability**: Differences in patient demographics, site protocols, or environmental factors.
2. **Unobserved Heterogeneity**: Factors influencing $$p$$ that are not explicitly measured or modeled.

This variability leads to **over-dispersion**, where the observed variance in success rates exceeds what is predicted under a simple binomial model. The **Beta-Binomial model** accounts for this variability by allowing the success probability $$p$$ to vary across groups according to a Beta distribution.

---

### The Model: Connecting Beta and Binomial

#### Two-Stage Process

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

### Simulation: Site-Level Response Rates in Clinical Trials

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

### Takeaways

1. The **Beta-Binomial model** provides a robust framework to account for over-dispersion in clinical trials caused by unobserved heterogeneity.
2. By allowing $$p$$ to vary across sites, it captures real-world variability, leading to more realistic uncertainty quantification.
3. Simulations not only validate theoretical insights but also help visualize the practical impact of using the Beta-Binomial model over the Binomial model.

This example ties back to the **Beta distribution’s role in frequentist inference**, showcasing its power to address challenges in real-world data analysis.



