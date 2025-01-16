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


