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

I will be writing a series of posts on the beta distribution, a versatile distribution in statistics and machine learning. This first part will provide an intuitive understanding of the beta distribution and explore its properties.

## General Properties of the Beta Distribution

The Beta distribution is defined on the interval $$[0,1]$$, making it ideal for modeling probabilities. It is characterized by two shape parameters, $$ \alpha $$ and $$ \beta $$, which control the shape of the distribution:

$$ \alpha $$: Affects the weight of successes.

$$ \beta $$: Affects the weight of failures.

The probability density function (PDF) of the Beta distribution is:

$$ \text{Beta}(x | \alpha, \beta) = \frac{x^{\alpha - 1}(1 - x)^{\beta - 1}}{B(\alpha, \beta)} $$

, where $$ B(\alpha, \beta) $$ is the Beta function, a normalizing constant that ensures the PDF integrates to 1.

### Shape of the Curve

* $$ \alpha = \beta = 1 $$: Uniform distribution.
* $$ \alpha > \beta $$ : Skewed toward 1 (success).
* $$ \alpha < \beta $$ : Skewed toward 0 (failure).
* $$ \alpha, \beta > 1 $$ : Bell-shaped.
* $$ \alpha, \beta < 1 $$ : U-shaped.

To get intuitions of the shape, think about Beta distribution as parametrizing on the weights of success and failure rates of a Bernoulli trials: $$ p^{\alpha -1} (1-p)^{\beta - 1} $$.

When we have no prior information regarding the success rate, set the weight to 0 resulting constant rate and uniform distribution. When we weight more on success, the distribution will be skewed toward 1, and vice versa. 

![Shape of the Curves of Beta Distribution](assets/img/2025-01-14-beta-distribution-part1/Beta_curve_bell_vs_u.png)

This intuition can be linked to following quantitative facts regarding the expectation of the Reta distribution and further deepen the understanding of the properties of the Beta distribution:

$$ 
\begin{align*}
  \mu = \mathbf{E}[X] &= \int_{0}^{1} x f(x; \alpha, \beta) \, dx \\
      &= \int_{0}^{1} x \frac{x^{\alpha - 1} (1 - x)^{\beta - 1}}{B(\alpha, \beta)} \, dx \\
      &= \frac{\alpha}{\alpha + \beta} \\
      &= \frac{1}{1 + \frac{\beta}{\alpha}}.
\end{align*} 
$$

If we take the limit of expectation with regards to $\frac{\beta}{\alpha}$:

$$
\left\{
\begin{aligned}
\lim_{\frac{\beta}{\alpha} \to 0} \mu &= 1 \\
\lim_{\frac{\beta}{\alpha} \to 1} \mu &= 0
\end{aligned}
\right.
$$

This corresponds to what the plot shows above and the intuition regarding the weights of success and failure rates.

## Beta as a Conjugate Prior in Bayesian Inference

A key property of the Beta distribution is that it is a conjugate prior for the Bernoulli, binomial, and certain other distributions in the exponential family. This means that if we use a Beta distribution as a prior for the parameter of these distributions, the posterior distribution will also be a Beta distribution. This property makes the Beta distribution particularly useful in Bayesian statistics.

A rigorous proof of the conjugacy property can be found [here](https://en.wikipedia.org/wiki/Conjugate_prior#Example). Intuitively, in a Bernoulli trial, assuming we have prior bielief of the success rate, the Beta distribution will update our belief of the success rate after observing the data. For example, if we have a Beta prior with parameters $$ \alpha $$ and $$ \beta $$, and we observe $$ n $$ successes and $$ m $$ failures, the posterior distribution will be a Beta distribution with parameters $$ \alpha + n $$ and $$ \beta + m $$.

A numeric example is shown in the plot below. Under a naive inference scenario, when we observe 5 sucesses and 5 failures, we may think about the success rate as $$ 50 \% $$. But under a Beysian setup, with the convinience of Beta piror and its property of conjugacy, we can easily incorperate historical data to form prior belief and update it.  but wi 

<!--![Prior vs. Posterior](assets/img/2025-01-14-beta-distribution-part1/prior_vs_posterior.png) { width=50% }-->

<!-- Inserting the Image -->
<img src="assets/img/2025-01-14-beta-distribution-part1/prior_vs_posterior.png" alt="Prior vs. Posterior" style="display: block; margin: 0 auto; width: 80%; height: auto;">

Assume we have prior knowledge of $$ 20 \% $$ success rate obtained from historical data containing 2 successes and 8 failures, with 5 successes and 5 failures are actually observed, we can update our belief to a Beta posterior $$ \text{Beta}(2+5, 8+5) $$. The plot shows the shift of the prior distribution to the posterior distribution with added information. Worth to note, the mean of the posterior distribution is $$ \frac{2+5}{2+5+8+5} = \frac{7}{20} = 35 \% $$, which is in between of our prior belief $$ 20 \% $$ and the naive inference soly based on observations $$ 50 \% $$.


