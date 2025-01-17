---
title: "Beta Distribution: Part 3 - Extensions of the Beta Distribution"
date: 2025-01-17
author: peng
categories: [Blogging, DataScience]
tags: [statistics, beta-distribution, beysian-statistics]
math: true
---

While the Beta distribution is versatile for modeling probabilities and proportions, there are cases where more flexibility is needed. In these scenarios, we turn to extensions such as the **Beta Prime distribution** and the various of extended Beta distributions. These extensions provide additional flexibility for modeling various types of data.

### Beta and Gamma Distributions

The Beta distribution is closely related to the Gamma distribution, which provides a foundation for understanding more complex extensions.

#### Relationship Between Beta and Gamma

If $$ Y_1 \sim \text{Gamma}(\alpha, 1) $$ and $$ Y_2 \sim \text{Gamma}(\beta, 1) $$ are two independent Gamma-distributed random variables, the Beta distribution can be derived by normalizing $$ Y_1 $$ with the sum of $$ Y_1 $$ and $$ Y_2 $$:

$$
X = \frac{Y_1}{Y_1 + Y_2}
$$

Here, $$ X $$ represents the proportion of $$ Y_1 $$ in the total $$ Y_1 + Y_2 $$, ensuring that $$ X $$ is bounded between 0 and 1.

#### Intuitive Insight

- **Gamma Distribution:** Models waiting times for events in a Poisson process.
- **Normalization:** By dividing $$ Y_1 $$ by the total $$ Y_1 + Y_2 $$, we obtain a proportion that reflects the relative contribution of $$ Y_1 $$ to the total waiting time.
- **Why Beta**: Since this proportion is bounded between 0 and 1 and $$ Y_1 $$ and $$ Y_2 $$ are independent, this closely resembles the proportion of successes in a Bernoulli process, leading to the Beta distribution.

#### Simulation Example

{::options parse_block_html="true" /}
<details><summary markdown="span">R Code for Simulating Beta as Gamma Ratio - Click to Expand</summary>

```r
set.seed(123)

alpha <- 3
beta <- 2

n_simulations <- 10000

Y1 <- rgamma(n_simulations, shape = alpha, rate = 1)
Y2 <- rgamma(n_simulations, shape = beta, rate = 1)

X <- Y1 / (Y1 + Y2)

hist(X, probability = TRUE, main = "Simulated X = Y1 / (Y1 + Y2) and Beta Distribution",
     xlab = "X", ylab = "Density", col = "lightblue", breaks = 30, xlim = c(0, 1))
curve(dbeta(x, alpha, beta), col = "darkred", lwd = 2, add = TRUE)
legend("topleft", legend = c("Simulated X", "Theoretical Beta"), 
       col = c("lightblue", "darkred"), lwd = 2, bty = "n")
```
</details>

{::options parse_block_html="false" /}

<img src="assets/img/2025-01-14-beta-distribution-part1/Beta_Gamma_Transform.png" alt="Beta and Gamma Simulation" style="display: block; margin: 0 auto; width: 80%; height: auto;">

In the simulation above, two independent Gamma variables $$ Y_1 \sim \text{Gamma}(3, 1) $$ and $$ Y_2 \sim \text{Gamma}(2, 1) $$ re used to generate $$ X = \frac{Y_1}{Y_1 + Y_2} $$. The histogram of $$ X $$ aligns closely with the theoretical Beta distribution.

### Beta Prime Distribution

The **Beta Prime distribution** is an extension of the Beta distribution that allows the random variable to take values greater than 0, making it suitable for modeling ratios of Gamma-distributed variables:

$$
X = \frac{Y_1}{Y_2}
$$

where

$$ Y_1 \sim \text{Gamma}(\alpha, 1) $$
$$ Y_2 \sim \text{Gamma}(\beta, 1) $$

**Probability Density Function (PDF):**

$$
f(x; \alpha, \beta) = \frac{x^{\alpha - 1} (1 + x)^{-(\alpha + \beta)}}{B(\alpha, \beta)}, \quad x > 0
$$

#### Intuitive Insight

- **$$ x^{\alpha - 1} $$:** Influences the distribution's growth for smaller $$ x $$ values, controlled by $$ \alpha $$.
- **$$ (1 + x)^{-(\alpha + \beta)} $$:** Governs the decay of the distribution as $$ x $$ increases, influenced by $$ \beta $$.
- **$$ B(\alpha, \beta) $$:** Serves as a normalizing constant ensuring the total probability integrates to 1.

Unlike the Beta distribution, which is confined to $$ [0, 1] $$, the Beta Prime distribution extends to $$ x > 0 $$, allowing for modeling ratios that can exceed 1. It is commonly used in F-tests in statistics, modeling the ratio of variances, rates, or proportions.

#### Visualization Example

I simulate the Beta Prime distribution by generating two Gamma-distributed variables and computing their ratio. The resulting histogram closely matches the theoretical Beta Prime curve, demonstrating its right-skewed shape and heavy tails. 

{::options parse_block_html="true" /}

<details><summary markdown="span">R Code for Visualizing Beta Prime Distribution - Click to Expand</summary>

```r
set.seed(123)

alpha <- 3
beta <- 2
n_simulations <- 1000
Y1 <- rgamma(n_simulations, shape = alpha, rate = 1)
Y2 <- rgamma(n_simulations, shape = beta, rate = 1)
X <- Y1 / Y2
dbetaprime <- function(x, alpha, beta) {
  ifelse(x > 0, 
         (x^(alpha - 1) * (1 + x)^(-(alpha + beta))) * 
           (gamma(alpha + beta) / (gamma(alpha) * gamma(beta))), 
         0)
}
hist(X, probability = TRUE, main = "Simulated Beta Prime Distribution",
     xlab = "X", ylab = "Density", col = "lightgreen", breaks = 100)
curve(dbetaprime(x, alpha, beta), col = "darkblue", lwd = 2, add = TRUE, from = 0, to = max(X)*1.1)
legend("topright", legend = c("Simulated X", "Theoretical Beta Prime"), 
       col = c("lightgreen", "darkblue"), lwd = 2, bty = "n")
```
</details>

{::options parse_block_html="false" /}

<img src="assets/img/2025-01-14-beta-distribution-part1/Beta_prime_distribution.png" alt="Beta Prime Simulation" style="display: block; margin: 0 auto; width: 80%; height: auto;">


### Generalized Extensions of the Beta Distribution

There are various ways to extend the Beta distribution to suit different modeling needs. Extensions typically introduce additional parameters or modify the functional form to provide greater flexibility for skewness, kurtosis, or tail behavior. These generalizations can adapt the Beta distribution to fit more complex data patterns. Below are some examples.

#### Generalized Beta Distribution (Type I)

**Form:**

$$
f(x; \alpha, \beta, \gamma, \delta) = \frac{\gamma x^{\alpha - 1} (1 - x^\delta)^{\beta - 1}}{B(\alpha, \beta)}, \quad 0 \leq x \leq 1.
$$

**Why It’s Useful:**

- The $$x^\delta$$ term adds flexibility to adjust skewness and kurtosis.
- Useful for modeling asymmetric data or data with sharp peaks, allowing for fine-tuned control over the shape of the distribution.

#### Kumaraswamy Distribution

**Form:**

$$
f(x; a, b) = a b x^{a - 1} (1 - x^a)^{b - 1}, \quad 0 \leq x \leq 1.
$$

**Why It’s Useful:**

- **No Special Functions:** Unlike the Beta distribution, the Kumaraswamy PDF does not involve the Beta function $$B(a, b)$$, making it computationally efficient for tasks like maximum likelihood estimation or numerical integration.
- **Flexibility:** Despite its simplicity, the $$x^a$$ and $$(1 - x^a)$$ terms allow for a wide range of shapes, similar to the Beta distribution, including skewness and multimodality.

This combination of simplicity and flexibility makes it highly practical for modeling bounded data, such as proportions or probabilities.

#### Mixed Beta Model

The **Mixed Beta Model** combines multiple Beta distributions to capture data with multimodality or heterogeneous subpopulations. Instead of assuming all data points come from a single Beta distribution, a mixture model allows different subsets of data to follow distinct Beta distributions.

**Form:**

$$
f(x) = \sum_{i=1}^k \pi_i \cdot f_\text{Beta}(x; \alpha_i, \beta_i),
$$

where:
- $$\pi_i$$ are the mixing proportions ($$\sum_{i=1}^k \pi_i = 1$$),
- $$f_\text{Beta}(x; \alpha_i, \beta_i)$$ is the Beta PDF for the $$i$$-th component.

**Why It’s Useful:**

- **Multimodal Data:** Captures datasets with multiple peaks that cannot be modeled by a single Beta distribution.
- **Heterogeneous Populations:** Accounts for subpopulations with different behaviors, such as varying success probabilities or response rates.

and there are many more extensions and variations of the Beta distribution that can be tailored to specific data characteristics and modeling requirements.

#### Simulation Example: Mixed Survey Data

Consider a survey measuring customer satisfaction on a scale from 0 to 1, involving two distinct groups:

- **Group 1 (New Customers):** Scores are widely distributed, reflecting varied first experiences.
- **Group 2 (Loyal Customers):** Scores are tightly clustered at high satisfaction levels.

This setup results in multimodal data that cannot be adequately modeled by a single Beta distribution. In this simulation, I mimic such real-world survey data and apply a Mixed Beta Model to accurately capture the distinct patterns of each group while providing an overall fit and compare it with a standard beta model.

The model is fit using [EM-Algoirthm](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm) by maximizing the likelihood of the data iteratively.

{::options parse_block_html="true" /}
<details><summary markdown="span">R Code for Mixed Beta Model Simulation - Click to Expand</summary>

```r
# Load necessary libraries
library(ggplot2)

set.seed(42)
n1 <- 500
n2 <- 500
alpha1 <- 8; beta1 <- 2  # High success rate
alpha2 <- 2; beta2 <- 8  # Low success rate

data1 <- rbeta(n1, alpha1, beta1)
data2 <- rbeta(n2, alpha2, beta2)
data <- c(data1, data2)

# Fit a single Beta model
fit_single_beta <- function(data) {
  mean_x <- mean(data)
  var_x <- var(data)
  alpha <- mean_x * ((mean_x * (1 - mean_x)) / var_x - 1)
  beta <- (1 - mean_x) * ((mean_x * (1 - mean_x)) / var_x - 1)
  return(list(alpha = alpha, beta = beta))
}

single_fit <- fit_single_beta(data)

# Fit a Beta mixture model
fit_beta_mixture <- function(data, k = 2) {
  init_lambda <- rep(1 / k, k)
  init_alpha <- runif(k, 1, 5)
  init_beta <- runif(k, 1, 5)
  init_params <- c(init_lambda, init_alpha, init_beta)
  
  log_likelihood <- function(params, data, k) {
    lambda <- params[1:k]
    alpha <- params[(k + 1):(2 * k)]
    beta <- params[(2 * k + 1):(3 * k)]
    lambda <- lambda / sum(lambda)
    ll <- 0
    for (i in seq_along(data)) {
      ll_i <- sum(lambda * dbeta(data[i], alpha, beta))
      ll <- ll + log(ll_i)
    }
    return(-ll)
  }
  
  opt <- optim(
    par = init_params,
    fn = log_likelihood,
    data = data,
    k = k,
    method = "L-BFGS-B",
    lower = c(rep(0.01, k), rep(0.01, 2 * k)),
    upper = c(rep(1, k), rep(20, 2 * k))
  )
  
  params <- opt$par
  lambda <- params[1:k] / sum(params[1:k])
  alpha <- params[(k + 1):(2 * k)]
  beta <- params[(2 * k + 1):(3 * k)]
  return(list(lambda = lambda, alpha = alpha, beta = beta, logLik = -opt$value))
}

fit_mixed <- fit_beta_mixture(data, k = 2)

# Extract fitted densities
x_vals <- seq(0, 1, length.out = 100)
beta1_fit <- dbeta(x_vals, fit_mixed$alpha[1], fit_mixed$beta[1])
beta2_fit <- dbeta(x_vals, fit_mixed$alpha[2], fit_mixed$beta[2])
overall_fit_mixed <- beta1_fit * fit_mixed$lambda[1] + beta2_fit * fit_mixed$lambda[2]
overall_fit_single <- dbeta(x_vals, single_fit$alpha, single_fit$beta)

fit_df <- data.frame(
  x = x_vals,
  overall_mixed = overall_fit_mixed,
  component1 = beta1_fit * fit_mixed$lambda[1],
  component2 = beta2_fit * fit_mixed$lambda[2],
  overall_single = overall_fit_single
)

ggplot(data = data.frame(x = data), aes(x = x)) +
  geom_histogram(aes(y = ..density..), bins = 30, fill = "#56B4E9", alpha = 0.6,
                 color = "gray70", size = 0.3) +
  geom_line(data = fit_df, aes(x = x, y = overall_mixed, color = "Mixed: Overall Fit"), size = 1.5) +
  geom_line(data = fit_df, aes(x = x, y = component1, color = "Mixed: Component 1"), size = 1.5, linetype = "dashed") +
  geom_line(data = fit_df, aes(x = x, y = component2, color = "Mixed: Component 2"), size = 1.5, linetype = "dotted") +
  geom_line(data = fit_df, aes(x = x, y = overall_single, color = "Single Beta Fit"), size = 1.5) +
  labs(title = "Single Beta vs Mixed Beta Model", x = "x", y = "Density", color = "Fitted Curves") +
  theme_minimal() +
  scale_color_manual(values = c("Mixed: Overall Fit" = "red", 
                                "Mixed: Component 1" = "blue", 
                                "Mixed: Component 2" = "green", 
                                "Single Beta Fit" = "purple")) +
  theme(plot.title = element_text(hjust = 0.5, face = "bold"))
```

</details>

{::options parse_block_html="false" /}

<img src="assets/img/2025-01-14-beta-distribution-part1/Mixed_Beta_Model.png" alt="Mixed Beta Model" style="display: block; margin: 0 auto; width: 80%; height: auto;">

The plot shows the single Beta model fails to capture the distinct peaks in the data, while the Mixed Beta Model accurately separates new and loyal customers. This highlights the mixed model's ability to handle multimodal data.
