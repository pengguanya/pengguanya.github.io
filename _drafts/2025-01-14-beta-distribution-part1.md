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

This corresponds to what the plot shows above and explains the skrewness of the curve under the different senarios when $\alpha$ is greater or smaller than $\beta$.

## Beta as a Conjugate Prior in Bayesian Inference

A key property of the Beta distribution is that it is a conjugate prior for the Bernoulli, binomial, and certain other distributions in the exponential family. This means that if we use a Beta distribution as a prior for the parameter of these distributions, the posterior distribution will also be a Beta distribution. This property makes the Beta distribution particularly useful in Bayesian statistics.

A rigorous proof of the conjugacy property can be found [here](https://en.wikipedia.org/wiki/Conjugate_prior#Example). Intuitively, in a Bernoulli trial where number of successe$$ k $$ follows a Binomial distribution - the **likelihood** in the Bayesian setup:

$$ k \mid p \sim \text{Binomial}(n, p) $$

, assuming the probability of success $$ p $$ has a prior:

$$ p \sim \text{Beta}(\alpha, \beta) $$

, and we observe $$ k $$ successes out of $$ n $$ trials, the posterior distribution for $$ p $$ is:

$$ p \mid k \sim \text{Beta}(\alpha + k, \beta + n - k) $$

, which is again a Beta distribution and thus we call Beta a conjugate prior for the Bernoulli likelihood.

A numeric example is shown in the plot below. Under a naive inference scenario, when we observe 5 sucesses and 5 failures, we may think about the success rate as $$ 50 \% $$. But under a Beysian setup, with the convinience of Beta piror and its property of conjugacy, we can easily incorperate historical data to form prior belief and update it.  but wi 

<!--![Prior vs. Posterior](assets/img/2025-01-14-beta-distribution-part1/prior_vs_posterior.png) { width=50% }-->

<!-- Inserting the Image -->
<img src="assets/img/2025-01-14-beta-distribution-part1/prior_vs_posterior.png" alt="Prior vs. Posterior" style="display: block; margin: 0 auto; width: 80%; height: auto;">

Assume we have prior knowledge of $$ 20 \% $$ success rate obtained from historical data containing 2 successes and 8 failures, with 5 successes and 5 failures are actually observed, we can update our belief to a Beta posterior $$ \text{Beta}(2+5, 8+5) $$. The plot shows the shift of the prior distribution to the posterior distribution with added information. Worth to note, the mean of the posterior distribution is $$ \frac{2+5}{2+5+8+5} = \frac{7}{20} = 35 \% $$, which is in between of our prior belief $$ 20 \% $$ and the naive inference soly based on observations $$ 50 \% $$.

## Iterative Updating
One of the most powerful aspects of Bayesian methods is **iterative (or online) updating**. In practice, data often arrives in small batches over time rather than all at once. If we are modeling a Bernoulli process with a Beta prior, each new batch of successes and failures can be used to update the current posterior distribution—which then becomes the new prior for the next batch of data. Because of the Beta–Bernoulli conjugacy, the posterior distribution remains in the Beta family, making these updates both simple and efficient.

**How It Works**  
1. Begin with a $$ \text{Beta}(\alpha_0, \beta_0) $$ prior.  
2. Observe $$ n $$ Bernoulli trials with $$ k $$ successes and $$ n - k $$ failures.  
3. Update your prior to obtain a posterior:
   
   $$
   \mathrm{Beta}\bigl(\alpha_0 + k,\;\beta_0 + (n - k)\bigr).
   $$

4. Repeat the process when new data arrive, treating the current posterior as the next round’s prior.

Over time, the posterior distribution will converge to the true success rate as more data are observed. 

**R Example of Iterative Updating**

Below is the simulation result of iterative Bayesian updating for a Bernoulli process with a true success probability, $$ p_{\text{true}} = 0.7 $$. We repeatedly observe small batches of data, update our Beta prior, and plot how the posterior shifts closer to the true value.

<img src="assets/img/2025-01-14-beta-distribution-part1/Iterative_Bayesian.png" alt="Iterative Bayesian" style="display: block; margin: 0 auto; width: 80%; height: auto;">

**Interpretation**:

 - The black curve is the initial $$ \text{Beta}(\alpha_0, \beta_0) $$prior.
 - Each new colored curve shows the updated posterior after seeing 10 more Bernoulli trials.
 - The dashed vertical line at $p_{\text{true}} = 0.7$ helps visualize the convergence of the posterior around the true parameter.

With each iteration, the posterior "learns" more from incoming data, shifting and narrowing around the true success rate. Beta as conjugate prior shows its power in both computational efficiency and intuitive interpretation in this process.


## Appendix

### Proof of the Differential Identity of Binomial PMF

Recall that for a Binomial random variable 
$$ \mathrm{Bin}(n,p) $$ 
with parameters \(n\) and \(p\), its pmf is

$$
P[\mathrm{Bin}(n,p) = i]
=
\binom{n}{i} \, p^i \, (1-p)^{\,n-i}.
$$

We want to take the derivative of this probability with respect to \(p\). Note that 
$$ \binom{n}{i} $$ 
is a constant (with respect to \(p\)), so we apply the product rule to 
$$ p^i (1-p)^{\,n-i}. $$

Hence,

$$
\frac{d}{dp} \Bigl( P[\mathrm{Bin}(n,p) = i] \Bigr)
=
\frac{d}{dp} \Bigl[\binom{n}{i} \, p^i \, (1-p)^{\,n-i}\Bigr].
$$

Using the product rule:

$$
\frac{d}{dp}\Bigl[p^i (1-p)^{n-i}\Bigr]
=
i \, p^{\,i-1} (1-p)^{n-i} 
\;-\; 
(n-i) \, p^i (1-p)^{\,n-i-1}.
$$

So,

$$
\frac{d}{dp}\Bigl[\binom{n}{i} \, p^i (1-p)^{n-i}\Bigr]
=
\binom{n}{i}
\Bigl[
  i \, p^{\,i-1} (1-p)^{\,n-i}
  \;-\;
  (n-i) \, p^i (1-p)^{\,n-i-1}
\Bigr].
$$

Factor out 
$$ p^{\,i-1} (1-p)^{\,n-i-1} $$ 
to get:

$$
\frac{d}{dp} \Bigl[\binom{n}{i} \, p^i (1-p)^{n-i}\Bigr]
=
\binom{n}{i} \, p^{\,i-1} (1-p)^{\,n-i-1}
\Bigl[
  i \,(1-p) \;-\; (n-i)\,p
\Bigr].
$$

On the other hand, observe that

$$
P[\mathrm{Bin}(n-1,p) = i-1]
=
\binom{n-1}{i-1} \, p^{\,i-1} (1-p)^{\,n-i},
$$

and

$$
P[\mathrm{Bin}(n-1,p) = i]
=
\binom{n-1}{i} \, p^{\,i} (1-p)^{\,n-1-i}.
$$

Using the identities

$$
\binom{n-1}{i-1}
=
\frac{i}{n} \binom{n}{i},
\quad
\binom{n-1}{i}
=
\frac{n-i}{n} \binom{n}{i},
$$

we have

$$
n\,P[\mathrm{Bin}(n-1,p) = i-1]
=
n \,\binom{n-1}{i-1} \, p^{\,i-1} (1-p)^{\,n-i}
=
i \,\binom{n}{i} \, p^{\,i-1} (1-p)^{\,n-i},
$$

and

$$
n\,P[\mathrm{Bin}(n-1,p) = i]
=
n \,\binom{n-1}{i} \, p^{\,i} (1-p)^{\,n-1-i}
=
(n-i)\,\binom{n}{i}\, p^i \,(1-p)^{\,n-1-i}.
$$

Hence,

$$
n \Bigl[P[\mathrm{Bin}(n-1,p) = i-1] - P[\mathrm{Bin}(n-1,p) = i]\Bigr]
=
\binom{n}{i} \, p^{\,i-1} (1-p)^{\,n-i-1}
\Bigl[
  i \,(1-p) \;-\; (n-i) \, p
\Bigr].
$$

This is precisely the same expression we got by direct differentiation. Therefore,

$$
\frac{d}{dp} P[\mathrm{Bin}(n,p) = i]
=
n \Bigl[P[\mathrm{Bin}(n-1,p) = i-1] - P[\mathrm{Bin}(n-1,p) = i]\Bigr].
$$

