---
title: "The Bayesian Interpretation of Regularization"
date: 2017-06-28 14:05:20 -0700
comments: true
categories: 
  - statistics
  - machine learning
tags:
  - Bayes theorem
  - LASSO
  - Ridge
  - regularization
htmlwidgets: TRUE
---

{% include toc %}

It's well known that Ridge and LASSO regularization have Bayesian interpretations. It is also known they can give improved generalization variance at the expense of bias---this is nicely outlined in one of Andrew Ng's [Coursera videos](https://www.coursera.org/learn/machine-learning/lecture/4VDlf/regularization-and-bias-variance). It's easy to demonstrate the bias-variance tradeoff empircally by doing regularized regression on fake data drawn from a known distribution, but it's also possible to realize these facts purely theoretically by considering the Bayesian interpretation in depth. 

When you do this you realize that regularization is equivalent to a poorly posed assertion about prior belief of your model parameters. LASSO and Ridge regression are akin to asserting prior belief that coefficients of a linear model should *all be zero*. In the eyes of a Bayesian purist, if you think all your coefficients should be zero then modeling is an absurd task to begin with. Pragmatists know better: in many real applications bias is not as important as some of us were taught, and more to the point, *if you're not aggressively reducing variance even at the expense of bias, you're leaving a lot of value on the table*.

In this post I want to derive the Bayesian posterior probability distribution that corresponds to standard regularized regression in its full gory detail. 

Let's see what it looks like.

# Ridge regression objective function

Ridge regression has two terms: the ordinary least-squares term and the L2 penalty.

There are a number of equivalent ways to write down the ordinary least squares objective function. Arguably the most useful one averages the loss over the data points because it's the one that's amenable to stochastic gradient descent in mini-batches. [Scikit-learn's SGD least-squares with L2 regularization optimizes](http://scikit-learn.org/stable/modules/sgd.html#mathematical-formulation)

\begin{equation}
U(\pmb{\theta}) = \frac{1}{N} \sum_{i=1}^N (y_i - f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{\lambda}{2}\lVert\pmb{\theta}\rVert_2^2
\end{equation}

There are $N$ data points in the data set, or more accurately, in the mini-batch, which can be as small as $N=1$. 

The linear model is

\begin{equation}
f(\pmb{\theta}; \pmb{x}) = \pmb{\theta}\cdot\pmb{x}
\end{equation}

with the coefficients $\pmb{\theta}$ serving as the free parameters. The first dimension of $\pmb{x}$ is always set to $1$ as a trick to account for the $y$-intercept. Ridge regression is a penalty on the square of the square- or L2-norm $\lVert\cdot\rVert_2^2$ of the parameters. The Lp norm is defined as

\begin{equation}
\lVert\pmb{x}\rVert_p = \left(\sum_{i=1}^Dx_i^p\right)^{1/p}
\end{equation}

with $p=0$ norm defined to be the number of nonzero entries of $\pmb{x}$. 

# Bayes' Theorem

Now I'm going to connect the dots to Bayes' Theorem. In scientific applications Bayes' Theorem is framed as

\begin{equation}
P(H|D) = \frac{P(D|H)P(H)}{P(D)}.
\end{equation}

You can begin to become friends with Bayes Theorem by first remembering the names of its kids.

| Term | Name | What it is |
| ---- | ------ | ---------- |
| $H$ | Hypothesis | Your statistical model or set of models |
| $D$ | Data | Your data set |
| $P(H\\|D)$ | Posterior | The probability a process that's approximated by a model produced the data that you observed |
| $P(D\\|H)$ | Likehood | The probability that the model produced the data | 
| $P(H)$ | Prior | Your weighted belief in the different values the model parameters can take |
| $P(D)$ | Evidence or marginal likelihood. | Hard to derive, and often ignored because it's model independent |

It's useful and common practice to immediately take the negative of the log and begin to ignore the evidence term.

$$
\begin{multline}
- \log{P(H|D)} =  \\
- \log{P(D|H)} - \log{P(H)} + \textrm{terms independent of }H.
\end{multline}
$$

The parameters that maximize the posterior also minimize the negative log-posterior. The reason this is useful is that it's significantly easier to take the first and second derivative of a sum than of a product, which is needed for many optimization algorithms, including stochastic gradient descent.

# A vanilla Bayesian model with sprinkles

Regularized least-squares is equivalent to asserting a specific Normal distribution of errors over a specific prior of coefficient values. I'll go through that piece by piece.

## The likelihood function

The first piece is a likelihood function proposed out of thin air:

\begin{equation}
P(D|H) = \mathscr{L}(\pmb{\theta}; \{y, \pmb{x}\}) = 
\prod_{i=1}^N A\exp\left(\frac{(y_i-f(\pmb{\theta}; \pmb{x}_i))^2}{2\sigma^2}\right)
\end{equation}

This is the probability that your data was drawn from a process that's well represented by your linear model, but with errors that are distributed according to a single Gaussian function. The errors are also asserted to be uncorrelated from data point to data point.

$A$ is a constant that you can look up, but it doesn't depend on $\pmb{\theta}$ so it doesn't matter what it is for what I'm trying to do.

## The prior

Ridge regression is akin to asserting this likelihood plus a normal prior on the coefficients,

\begin{equation}
P(H) = P(\pmb{\theta}) = \prod_{j=1}^M B \exp\left(\frac{\theta_j^2}{2\sigma_{2}^2} \right).
\end{equation}

This says, my prior belief is that all coeffients are most probably zero but other values near zero are possible according to the specific shape of the Gaussian function, and they're all uncorrelated and have equal variance $\sigma_{2}^2$. 

Putting the normal error distribution together with the normal prior on coefficients, and skipping over the simple algebra, the log-posterior is

$$
\begin{multline}
- \log{P(f(\pmb{\theta}; \pmb{x})|\{y, \pmb{x}\})} = \\
\frac{1}{2\sigma^2}\sum_i^N (y_i-f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{1}{2\sigma_{2}^2} \lVert\pmb{\theta}\rVert_2^2 + \textrm{terms independent of }\pmb{\theta}.
\end{multline}
$$

I've absorbed the probability distribution coeffients into the $\pmb{\theta}$-independent term. Compare this to the Scikit-learn objective function.

# Interpreting the Bayesian formulation

It's pretty obvious the negative log-posterior looks a lot like the Scikit-learn objective function. If you multiply the entire negative log-posterior by $2\sigma^2/N$ you get $U$ back exactly as

$$
\begin{multline}
U(\pmb{\theta}) = - \frac{2\sigma^2}{N} \log{P(f(\pmb{\theta}; \pmb{x})|\{y, \pmb{x}\})} = \\
\frac{1}{N}\sum_i (y_i-f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{1}{N}\frac{\sigma^2}{\sigma_{2}^2} \lVert\pmb{\theta}\rVert_2^2 + \textrm{terms independent of }\pmb{\theta}.
\end{multline}
$$

so long as you define the regularization parameter as

$$
\begin{equation}
\lambda = \frac{2}{N}\frac{\sigma^2}{\sigma_{2}^2}.
\end{equation}
$$

This form leads to the Bayesian interpretation of the Ridge penalty. The L2 hyperparameter is proportional to the inverse variance of the width of your prior over coefficients. 

**Choosing a small $\lambda$ is like asserting a weak prior belief on the range of possible coefficient values, centered on zero.** In the extreme case of $\lambda\to0,$ the width of the Gaussian prior grows to become a flat prior and you end up doing ordinary least squares.

**Choosing a large $\lambda$ is like asserting a strong prior belief on the range of possible coefficient values, centered on zero.** In the extreme case of $\lambda\to\infty,$ the width of the Gaussian prior shrinks to become a Dirac delta function, and you're effectively saying you know exactly what the coefficients should be---all zero---and you might as well not be doing an additional regression.

Ridge regression is therefore the same as a Bayesian model with a specific normal likelihood and a specific normal prior, and the prior makes little intuitive sense unless it is very weak, in which case you're essentially doing ordinary least-squares regression.

# LASSO's prior

LASSO, or L1 regularized regression, is extremely closely related to Ridge. The only difference is a Laplace prior instead of Gaussian.

$$
\begin{equation}
P(H) = P(\pmb{\theta}) = C \exp\left(\frac{\lVert\pmb{\theta}\rVert_1}{\sigma_{1}} \right).
\end{equation}
$$

The Laplace distribution is peakier at 0, but otherwise behaves somewhat like the Gaussian prior of Ridge regression.

# The tragedy of L0 regularization 

Since we're counting down in L-numbers, let's talk about L0 regularization. The L0 norm is the number of nonzero values of a vector. You might be tempted to write down a prior as 

$$
\begin{equation}
P(H) = P(\pmb{\theta}) = \prod_{j=1}^M C \exp\left(\frac{\lVert\pmb{\theta}\rVert_0^0}{\sigma_{0}} \right).
\end{equation}
$$

And for completeness, here's the L0 objective function.

\begin{equation}
U(\pmb{\theta}) = \frac{1}{N} \sum_{i=1}^N (y_i - f(\pmb{\theta}; \pmb{x}_i))^2 + \lambda\lVert\pmb{\theta}\rVert_0^0
\end{equation}

This is a nonsensical prior, and the regression itself is ill defined as well.

First, the coefficients are continuous variables and are therefore never really exactly 0 valued. The L0 norm is the total number of nonzero coefficients, but because the coefficients will take on values of 0 with vanishingly small probability during training, this will always be equal to $M$, the total number of coefficients. So maybe you get a little tricky and initialize your coefficients to 0---now your subgradient is broken because it is not defined at 0, so SGD has no idea where to push your solution for the next iteration. So you get tricky again and initialize your coefficients according to some random distribution. Now your subgradients are well defined but your coefficients will *never be exactly 0*, which renders the entire exercise meaningless.

Second, this prior form is actually flat, except for the infinitessimal discontinuity at 0. And yet we mysteriously gave it an exponential functional form---with a width parameter, even. If the number of nonzero coefficients $m$ is forced to be fixed during a single training run, then this functional form reduces to a simple flat prior, which is fine. But if we are to try different numbers of nonzero coefficients ($m$ variable) as L0 regularization implies, then the prior as written does not integrate to $1$, rendering it nonsense.

Taking a step back, what the L0 norm is perhaps attempting to say is "some number of coefficients are zero and some are not, I just don't know which ones. Of those that are nonzero, I am entirely ignorant as to what value they should take". This is more like a random selection of some number of coefficients, which serve as candidates for an ordinary least-squares regression, plus a strategy that tries this over all possible groupings of any number of candidate coefficients. When you think of it this way it's obvious you have a combinatorial explosion on your hands. If you have $M$ total coefficients, you have to choose $m$ as regression candidates and you have to try all possible $m$ values. This is the sum of the Binomial coefficients, which is equal to $2^M$. Not possible. (I am told that [this paper](https://pdfs.semanticscholar.org/f629/5fd69d76d606f66cc15f58767a8161d60335.pdf) proves that L0 regularization is NP hard, but I do not claim to understand the proof there.)

Long story short, L0 regularization has its heart in the right place but it just doesn't fit into the same regression or Bayesian inference framework as Ridge and LASSO.

# Conclusion

Regularization is bad Bayesianism but good practice. I haven't demonstrated its usefulness---I'll leave that for a future post.