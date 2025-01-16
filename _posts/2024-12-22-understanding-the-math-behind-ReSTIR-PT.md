---
layout: single
title: "Understanding The Math Behind ReSTIR PT"
date: 2024-12-22 16:41
category: 
author: Patrick Attimont
use_math: true
toc: true
toc_sticky: true
words_per_minute: 75
header:
    teaser: "/assets/images/teasers/restir-pt.jpg"
---

**Note:** this post is work in progress, I'm updating it whenever I find the time.
{: .notice--danger .text-center}

When I first tried to delve into the math of the original ReSTIR paper, I stumbled upon [this excellent article](https://agraphicsguynotes.com/posts/understanding_the_math_behind_restir_di/) that helped me a lot to understand all the theory underlying the algorithm - I would highly recommend starting with that article first as an introduction.
Since there is more to ReSTIR than ReSTIR DI, I thought I could share my own notes on GRIS and ReSTIR PT, hoping it will help others grasp the mathematical concepts behind unbiased and consistent spatiotemporal resampling.
These notes focus exclusively on the mathematical aspects, as the course [A Gentle Introduction to ReSTIR:
Path Reuse in Real-time](https://intro-to-restir.cwyman.org/) already provides a strong and intuitive understanding of the algorithm.

## Definitions
- For $f: \Omega \rightarrow \mathbb{R}$, $$\text{supp } f = \{x \in \Omega: f(x) > 0\}$$
- For  a random variable $X \in \Omega$ with a pdf $p_X$, $$\text{supp } X = \text{supp }p_X$$
- $\Omega:$ Integration domain of $f$
- $\Omega_i:$ Integration domain of sample $X_i$
- $X_i:$ input sample for RIS
- $M:$ number of input samples
- $Y:$ output sample for RIS
- $f: \Omega \rightarrow \mathbb{R}:$   function we want to integrate
- $I:$  integral we want to estimate. Essentially, $I = \int_{\Omega}f(x)dx$
- $\hat{p}:$   unnormalized target pdf
- $\bar{p}:$   normalized target pdf $(\bar{p} = \hat{p} / \|\|\hat{p}\|\|)$
- $W_i:$ unbiased contribution weight
- $w_i:$ resampling weight
- $T_i:$ shift mapping from $\Omega_i$ to $\Omega$

## Motivations

ReSTIR relies on resampling from previous frames and neighboring pixels with RIS.
The main problem is that these samples already come from RIS applied to different integration domains and they can be correlated due to spatiotemporal resampling in previous frames.

Generalized resampled importance sampling (GRIS) is a generalization of RIS that aims at solving these issues. It applies RIS on correlated samples $(X_i)_{i=1}^M$ from different integration domains $\Omega_i$. It selects sample $X_s$ and maps it to $f$'s domain $\Omega$ using a shift mapping: $Y = T(X_s)$ so that $Y$'s pdf approaches $\hat{p}$.

Another important problem that GRIS solves is that samples drawn from previous RIS passes have intractable pdfs, which means that we cannot use $p_i(x_i)$ to compute our RIS weights in the next RIS steps. GRIS relies instead on unbiased contribution weights $W_i$ which are random variables approximating $1/p_i(X_i)$ (unbiased estimator).

## Unbiased contribution weight

{% capture UCW-definition %}
More formally, an **unbiased contribution weight** (UCW) for a random variable $X \in \Omega$ is a random variable $W \in \mathbb{R}$ for which:

$$
\begin{equation}
\mathbb{E}[f(X)W] = \int_{\text{supp}(X)} f(x)dx
\label{eq:sample}
\end{equation}
$$

for any $f: \Omega \rightarrow \mathbb{R}$ integrable.
{% endcapture %}

<div class="notice--warning">{{ UCW-definition | markdownify }}</div>

$W$ acts as a replacement for the factor $1/p(X)$ in the Monte Carlo estimator. It is in fact an unbiased estimator of $1/p(X)$:

$W$ is an unbiased contribution weight for $X \iff \mathbb{E}[W\|X] = \dfrac{1}{p(X)}$
{: .notice--info .text-center}

{% capture UCW-proof %}
**Proof**

$\Leftarrow$: Assuming $ \mathbb{E}[W\|X] = \dfrac{1}{p(X)} $

$$
\begin{align*}
\mathbb{E}[f(X)W] &= \mathbb{E}[\mathbb{E}[W|X]f(X)] \\
                &= \mathbb{E}\left[\dfrac{f(X)}{p(X)}\right] \\
                &= \int_{\text{supp}(X)} f(x)dx

\end{align*}
$$

$\Rightarrow$: from GRIS supplemental

For any $f: \Omega \rightarrow \mathbb{R}$ integrable, assuming

$$
\begin{equation*}
\mathbb{E}[f(X)W] = \int_{\text{supp}(X)} f(x)dx
\end{equation*}
$$

we want to prove that

$$
\begin{equation*}
\forall x \in \text{supp}(X)\text{, } \mathbb{E}[W|X=x] = \dfrac{1}{p(x)}
\end{equation*}
$$

We can resonate on any measurable subset $A \subset \text{supp}(X)$ to prove both functions are equal:

$$
\begin{align*}
\int_A \dfrac{1}{p(x)}dx &= \int_{\text{supp}(X)} \dfrac{\mathbb{1}_A (x)}{p(x)}dx \\
                        &= \mathbb{E}\left[W \dfrac{\mathbb{1}_A (X)}{p(X)}\right] \\
                        &= \mathbb{E}\left[\mathbb{E}[W|X]\dfrac{\mathbb{1}_A (X)}{p(X)}\right] \\
                        &= \int_{\text{supp}(X)} \mathbb{E}[W|X = x] \mathbb{1}_A (x)dx \\
                        &= \int_A \mathbb{E}[W|X = x]dx
\end{align*}
$$

Since this holds for every measurable subset $A \subset \text{supp}(X)$, we have:

$$
\begin{equation*}
\forall x \in \text{supp}(X)\text{, } \mathbb{E}[W|X=x] = \dfrac{1}{p(x)}
\end{equation*}
$$
{% endcapture %}

<div class="notice">{{ UCW-proof | markdownify }}</div>

## Resampling with unbiased contribution weights

These UCW allow us to still be able to resample from previous RIS steps by using new resampling weights $w_i$. For now, we assume:

$$
\begin{equation}
w_i = m_i(X_i) \hat{p}_i(X_i) W_i
\end{equation}
$$
{: .notice--warning .text-center}

The only difference with the previous RIS weights formulation lies in the replacement of $1/p_i(X_i)$ with $W_i$: if $X_i$ comes from a previous RIS process, we must replace it with its UCW $W_i$. However, in the case that we can express $X_i$'s pdf (e.g., a BSDF or light sample), we should obviously use $1/p_i(X_i)$.

Now, assuming we have these weights $w_i$ for each $X_i \in \mathbb{R}$, we want to unbiasedly estimate $f$'s integral, and compute a new UCW $W_Y$ for the sample $Y = X_s$ that we are going to draw with RIS. $W_Y$ will then be used in following RIS passes (spatial or temporal resampling) to draw a new sample $Y'$ and so on.

GRIS paper introduces functions $g_i: \Omega_i \rightarrow \mathbb{R}$ and computes the expectation of the random variable

$$
\begin{equation*}
Z = \dfrac{g_s(X_s)W_s}{p_s(s)}
\end{equation*}
$$

with $s$ the index of the selected sample $Y = X_s$, and $p_s$ the PMF of the selection index: 

$$
\begin{equation*}
p_s(i) = \dfrac{w_i}{\sum_{j = 1}^{M} w_j}
\end{equation*}
$$

We obtain:

$$
\begin{align}
\mathbb{E}[Z] &= \mathbb{E}\left[\mathbb{E}\left[\dfrac{g_s(X_s)W_s}{p_s(s)}| X_1, .., X_M \right]\right] \nonumber \\
            &= \mathbb{E}\left[\sum_{i = 1}^{M} \dfrac{g_i(X_i)W_i}{p_s(i)}p_s(i) \right] \nonumber \\
            &= \sum_{i = 1}^{M}\mathbb{E}\left[g_i(X_i)W_i\right] \nonumber \\
            &= \sum_{i = 1}^{M}\int_{\text{supp}(X_i)} g_i(x)dx \quad (W_i \text{ is an UCW for } X_i) \label{eq:ez}
\end{align}
$$

We now have an expression that looks close to $f$'s integral: if we can express every $g_i$ so that this sum is equal to $I$, we have solved GRIS! However, there is still one problem: $X_i$ have different sampling domains $\Omega_i$, so we cannot apply $f$ on $X_i$ without some caution.

### Special case

In the special case that all $X_i$ have the same domain $\Omega$, we can choose:

$$
\begin{equation*}
\forall i \in [1, M], g_i(X_i) = \dfrac{1}{M}f
\end{equation*}
$$

Which gives

$$
\begin{equation*}
\mathbb{E}[Z] = \int_{\text{supp}(Y)} f(x)dx
\end{equation*}
$$

Thus we get an unbiased estimator for $I$:

$$
\begin{equation}
\hat{I}_{GRIS} = \dfrac{1}{M}f(Y)\dfrac{\sum_{j = 1}^{M}w_j}{w_s}W_s
\end{equation}
$$
{: .notice--warning}

Note that if we replace $W_i$ with $1/p_i(X_i)$, we obtain the original RIS estimator which is expected since all the $X_i$ are in $\Omega$:

$$
\begin{equation*}
\hat{I}_{RIS} = \dfrac{f(Y)}{\hat{p}(Y)}\sum_{j = 1}^{M}w_j
\end{equation*}
$$

Additionally, this equality gives us an UCW for $Y$: since we have

$$
\begin{equation*}
\mathbb{E}[f(Y)W_Y] = \int_{\text{supp}(Y)} f(x)dx
\end{equation*}
$$

by definition, we have

$$
W_Y = \dfrac{1}{M}\dfrac{\sum_{j = 1}^{M}w_j}{w_s}W_s
$$
is an unbiased contribution weight for Y.
{: .notice--warning .text-center}


## Shift mappings

In the general case, each $X_i$ has a different sampling domain $\Omega_i$.

This time, we don't assume any expression for $w_i$ and we start again from equation $\eqref{eq:ez}$:

$$
\begin{equation*}
\mathbb{E}[Z] = \sum_{i = 1}^{M}\int_{\text{supp}(X_i)} g_i(x)dx
\end{equation*}
$$

We want to find $\mathbb{E}[Z] = I$.

We will have to map all the samples $X_i$ from $\Omega_i$ to $\Omega$, and account for this mapping in our integral. For this, we will use shift mappings for each $X_i$.

A **shift mapping** $T_i$ from $\Omega_i$ to $\Omega$ is a bijective function from a subset $\mathcal{D}(T_i) \subset \Omega_i$ to its image $\mathcal{I}(T_i) \subset \Omega$.
{: .notice--warning .text-center}

Note that the bijectivity is an important property since $T_i$ will change the variable of integration in our equality.

We can then choose:

$$
g(x) =
\begin{cases} 
c_i(y_i)f(y_i) \left| \dfrac{\partial T_i}{\partial x} \right| & \text{if } x \in \mathcal{D}(T_i), \\
0 & \text{otherwise}
\end{cases}
$$

with $y_i = T_i(x)$.

$c_i: \Omega \rightarrow \mathbb{R}$ are called contribution MIS weights. They are a partition of unity:
$\forall x \in \Omega, \sum_{i = 1}^{M} c_i(x) = 1$, and their expression will be developed later.

We also assume the following:

$$
\begin{equation}
X_i \in \mathcal{D}(T_i) \text{ and } \hat{p}(Y_i) > 0 \iff w_i > 0
\label{eq:eqwi}
\end{equation}
$$
{: .notice--warning .text-center}

with $Y_i = T_i(X_i)$.

In other words, $X_i$ must be samplable iff $T_i(X_i)$ is defined and lies in $\hat{p}$'s support.

Assuming $\mathcal{D}_i = \text{supp} X_i \cap \mathcal{D}(T_i)$, this gives:

$$
\begin{align}
\mathbb{E}[Z] &= \sum_{i = 1}^{M} \int_{\mathcal{D}_i} c_i(T_i(x))f(T_i(x))\left| \dfrac{\partial T_i}{\partial x} \right| dx \nonumber \\
            &= \sum_{i = 1}^{M} \int_{T_i(\mathcal{D}_i)} c_i(x) f(x) dx \nonumber \\
            &= \int_{\bigcup_{i = 1}^{M} T_i(\mathcal{D}_i)} \sum_{i = 1}^{M} c_i(x) f(x) dx \nonumber \\
            &= \int_{\bigcup_{i = 1}^{M} T_i(\mathcal{D}_i)} f(x) dx
\end{align}
$$

It should now be clearer why we assumed $\eqref{eq:eqwi}$: our equality holds only if we can sample $X_i$ when $T_i(X_i)$ is defined, ie $X_i \in \mathcal{D}_i$, and $\hat{p}(Y_i) > 0$. If $\hat{p}(Y_i) > 0$ and $w_i = 0$, it means that a part of this integral will never be evaluated since some $Y_i \in \text{supp } \hat{p}$ are never sampled. On the other hand, if $T_i(X_i)$ is not defined or $\hat{p}(Y_i) = 0$, we don't want to choose $X_i$ since in either case it will not contribute to the integral.

Additionally, $\eqref{eq:eqwi}$ gives:

$$
\begin{align}
\text{supp }Y &= \{ y_i: w_i > 0 \} \nonumber \\
            &= \bigcup_{i = 1}^{M} \text{supp } \hat{p} \cap T_i(\mathcal{D}_i) \nonumber \\
            &= \text{supp } \hat{p} \cap \bigcup_{i = 1}^{M} T_i(\mathcal{D}_i)
\end{align}
$$

We can rewrite our equation since $Y_i \notin \text{supp } \hat{p}$ have zero RIS weights and thus don't contribute to the integral:

$$
\begin{equation}
\mathbb{E}[Z] = \int_{\text{supp}(Y)} f(x)dx
\end{equation}
$$

We now have an unbiased estimator for $I$:

$$
\begin{equation}
\hat{I}_{GRIS} = c_s(Y)f(Y) \left|  \dfrac{\partial T_s}{\partial X_s} \right| \dfrac{\sum_{j = 1}^{M}w_j}{w_s} W_s
\end{equation}
$$
{: .notice--warning .text-center}

still with the assumption that $g_i(x) = 0$ if $x \notin \mathcal{D}(T_i)$.

And, by definition, we get a new unbiased contribution weight for $Y$:

$$
\begin{equation}
W_Y = c_s(Y) \left| \dfrac{\partial T_s}{\partial X_s} \right| \dfrac{\sum_{j = 1}^{M} w_j}{w_s} W_s
\end{equation}
\label{eq:W_Yfirstdef}
$$
{: .notice--warning .text-center}

Theoretically, we now have a way of integrating $f$ on $Y$'s support. If we want to obtain this integral on $\Omega$, we must have $\text{supp }Y = \Omega$, which translates into:

$$
\bigcup_{i = 1}^{M} T_i(\mathcal{D}_i) = \Omega
$$

We have a very nice way of solving this: choosing $\Omega_1 = \Omega$ as one of the sampling domains for $X_1$ with a pdf $p_1$ such that $\text{supp } p_1 = \text{supp }f$. We can then use the identity shift $T_1(X_1) = X_1$ and see that $T_1(\mathcal{D}_1) = \Omega$ and thus $\text{supp }Y = \Omega$. Such samples are called **canonical samples** (eg., a BSDF sample).

For now, we have an unbiased estimator of $I$, which means that if the GRIS algorithm is repeated many times, the average of the results will converge to $I$.
However, ReSTIR doesn't average over time, it just increases the sample count $M$. Thus we would like to obtain convergence as $M$ increases: $\hat{I}_{GRIS}$ should be **consistent**.
In other words, we want the output sample's pdf $p_Y$ to converge towards $\bar{p}$ as the number of input samples $M$ goes to infinity.
Therefore we have to find an expression for $w_i$, $m_i$ and $c_i$ that satisfies this asymptotic convergence.

{% capture wi-def %}
Assuming $\text{supp }\hat{p} \subset \text{supp } Y$, and

$$
\begin{equation}
\text{Var}\left[\displaystyle\sum_{i = 1}^{M} w_i\right] \xrightarrow{M \to \infty} 0
\label{eq:wi-asump}
\end{equation}
$$

then choosing

$$
\begin{equation}
w_i = 
\begin{cases}
m_i(T_i(X_i))\hat{p}(T_i(X_i))W_i \left| \dfrac{\partial T_i}{\partial X_i} \right| & \text{if } X_i \in \mathcal{D}(T_i), \\
0 & \text{otherwise}
\end{cases}
\label{eq:wi-def}
\end{equation}
$$

guarantees this asymptotic convergence.
{% endcapture %}

<div class="notice--warning">{{ wi-def | markdownify }}</div>


{% capture wi-proof %}
**Proof**

From GRIS supplemental. Since we have

$$
\begin{equation*}
\sum_{i = 1}^{M} w_i = \hat{p}(Y)W_Y
\end{equation*}
$$

then \eqref{eq:wi-asump} becomes: $\text{Var}[\hat{p}(Y)W_Y] \xrightarrow{M \to \infty}0$. We want to prove that, for $w_i$ defined in \eqref{eq:wi-def}, $p$ converges towards $\bar{p}$ in probability, which means:

$$
\begin{equation*}
\forall \epsilon \in \mathbb{R}, \text{Pr}\left[|\bar{p}(Y) - p(Y)| > \epsilon\right] \xrightarrow{M \to \infty} 0
\end{equation*}
$$

We first show that 

$$
\begin{equation*}
\mathbb{E} \left[ \left| \frac{\bar{p}(Y)}{p(Y)} - 1 \right|^2 \right] \xrightarrow{M \to \infty} 0
\end{equation*}
$$

$$
\begin{align*}
\mathbb{E} \left[ \left| \frac{\bar{p}(Y)}{p(Y)} - 1 \right|^2 \right]
&= \frac{1}{\|\widehat{p}\|^2} \mathbb{E} \left[ \left| \frac{\widehat{p}(Y)}{p(Y)} - \|\widehat{p}\| \right|^2 \right] \\
&\leq \frac{1}{\|\widehat{p}\|^2} \operatorname{Var} \left[ \widehat{p}(Y) W_Y \right]
\xrightarrow{M \to \infty} 0
\end{align*}
$$

{% endcapture %}

<div class="notice">{{ wi-proof | markdownify }}</div>

We can now replace $w_s$ in \eqref{eq:W_Yfirstdef} using \eqref{eq:wi-def}. This gives:

$$
\begin{equation}
W_Y = \dfrac{c_s(Y)}{m_s(Y)} \dfrac{1}{\hat{p}(Y)} \sum_{j = 1}^{M} w_j
\end{equation}
$$
{: .notice--warning .text-center}

## Additional insights

These are the main questions I had when reading the paper and trying to understand the algorithm:
- Why does the offline version of ReSTIR PT turn off temporal reuse?

    This confused me but it's really important to understand it. What is the main goal of temporal resampling? To reuse good samples from previous frames. Why is it useful? Because it allows to feed the RIS algorithm with information gathered temporally, ie a reservoir that has gathered many samples into one with its unbiased contribution weight.
    In the case of offline rendering, we are averaging over time, which means that we don't loose temporal information as we did in the real time version. Reusing samples from previous frames means that we would reuse information that is already present in our average. This makes us reuse several times the same sample which will just increase correlation and make convergence slower.



## References
- [Importance Resampling for Global Illumination](https://doi.org/10.2312/EGWR/EGSR05/139-146)
- [Spatiotemporal reservoir resampling for real-time ray tracing with dynamic direct lighting](https://dl.acm.org/doi/10.1145/3386569.3392481)
- [ReSTIR GI: Path Resampling for Real‐Time Path Tracing](https://diglib.eg.org/items/ae55c04f-4832-48af-b60a-95fecd62d0ce)
- [Generalized resampled importance sampling: foundations of ReSTIR](https://dl.acm.org/doi/10.1145/3528223.3530158)
- [Area ReSTIR: Resampling for Real-Time Defocus and Antialiasing](https://dl.acm.org/doi/10.1145/3658210)
- [Decorrelating ReSTIR Samplers via MCMC Mutations](https://dl.acm.org/doi/10.1145/3629166)
- [A Gentle Introduction to ReSTIR Path Reuse in Real-Time](https://dl.acm.org/doi/10.1145/3587423.3595511)
- [Understanding The Math Behind ReStir DI](https://agraphicsguynotes.com/posts/understanding_the_math_behind_restir_di/)