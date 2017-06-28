---
title: "The Bayesian Interpretation of Regularization"
date: 2017-06-05 14:05:20 -0700
comments: true
categories: 
htmlwidgets: TRUE
---

One really smart data scientist I worked with once said, "I've never heard anyone use the word 'prior' correctly."

Actually what I think she said was, every time someone says prior she stops believing whatever they're saying. You could say she had a strong prior.

Bayesian inference is a power tool that's hard to handle. It's how humanity puts forth its best estimates of the fundamental physical constants of the universe, and in those applications scientists may use priors. The wrong choice of priors lead to biased estimates, and it's easy to understand why this is not acceptable when you're, say, wanting to tell the world whether the Higgs boson or dark energy exists. They cannot afford to be fast and loose about it.

Bias in other settings is acceptable, and if you are willing to accept some bias you often can get radical reductions in variance. Regularization is one way to do this. Andrew Ng does a great job briefly describing the bias-variance tradeoff here in his [Coursera video](https://www.coursera.org/learn/machine-learning/lecture/4VDlf/regularization-and-bias-variance). 

Regularization is supremely useful, and biased. 

Regression, and regularized regression, have Bayesian interpretations -- and if you carry regularized linear regression all the way back to its Bayesian formulation it is immediately obvious where the bias is coming from. **Regularization is equivalent to a poorly posed assertion about prior belief of your model parameters.** LASSO and Ridge regression are akin to asserting prior belief that coefficients of a linear model should **all be zero**. In the eyes of a Bayesian purist, if you think all your coefficients should be zero then the modeling task is absurd to begin with.

But in many real applications bias is not that important, and more to the point, **if you're not aggressively reducing variance even at the expense of bias, you're leaving a lot of value on the table**.

I'd like to step through some fundamentals of the Bayesian interpretation of regularization. The broader goal is to build intuition and provide a platform for working data scientists to have this discussion. 

## Ridge regression objective function

Ridge regression has two terms: the ordinary least-squares term and the L2 penalty.

There are a number of equivalent ways to write down the ordinary least squares objective function. Arguably the most useful one averages the loss over the data points because it's the one that's amenable to stochastic gradient descent in mini-batches. [Scikit-learn's SGD least-squares with L2 regularization optimizes](http://scikit-learn.org/stable/modules/sgd.html#mathematical-formulation)

$$
U(\pmb{\theta}) = \frac{1}{N} \sum_{i=1}^N (y_i - f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{\lambda}{2}\lVert\pmb{\theta}\rVert_2^2
$$

The linear model is

$$
f(\pmb{\theta}; \pmb{x}) = \pmb{\theta}\cdot\pmb{x}
$$

with the first dimension of \\(\pmb{x}\\) always set to \\(1\\) as a trick to account for the \\(y\\)-intercept. Ridge regression is a penalty on the square of the square- or L2-norm \\(\lVert\cdot\rVert_2^2\\) of the parameters. The Lp norm is defined as

$$
\lVert\pmb{x}\rVert_p = \left(\sum_{i=1}^Dx_i^p\right)^{1/p}
$$

with \\(p=0\\) norm defined to be the number of nonzero entries of \\(\pmb{x}\\).	

## Bayes' Theorem

I'm going to connect the dots to Bayes' Theorem. In scientific applications Bayes' Theorem can be framed as

$$
P(H|E) = \frac{P(E|H)P(H)}{P(E)}.
$$

You first become friends with Bayes Theorem by remembering the names of its kids.

| Term | Name | What it is |
| ---- | ------ | ---------- |
| \\(H\\) | Hypothesis | Your statistical model or set of models |
| \\(E\\) | Evidence | Your data set |
| \\(P(H\\|E)\\) | Posterior | The probability a process that's approximated by a model produced the data that you observed |
| \\(P(E\\|H)\\) | Likehood | How well your model explains the data over different model parameter values | 
| \\(P(H)\\) | Prior | Your weighted belief in the different values the model parameters can take |
| \\(P(E)\\) | Also "evidence" | Often ignored because it's model independent |

It's useful to immediately take the negative of the log and begin to ignore the evidence term.

$$
- \log{P(H|E)} = - \log{P(E|H)} - \log{P(H)} + \textrm{terms independent of }H.
$$

The parameters that maximize the posterior also minimize the negative log-posterior.

## A specific Bayesian model

Regularized least-squares is equivalent to asserting a specific Normal distribution of errors over a specific prior of coefficient values. I'll go through that piece by piece.

### The likelihood function

The first piece is a likelihood function proposed out of thin air:

$$
P(E|H) = \mathscr{L}(\pmb{\theta}; \{y, \pmb{x}\}) = \prod_{i=1}^N A\exp\left(\frac{(y_i-f(\pmb{\theta}; \pmb{x}_i))^2}{2\sigma^2}\right)
$$

This says that errors are normally distributed, that the predictors are uncorrelated and that the predictors all have the same variance. \\(A\\) is a constant that you can look up, but it doesn't depend on \\(\pmb{\theta}\\) so it doesn't matter what it is for what I'm trying to do.

### The prior

Ridge regression is akin to asserting this likelihood plus a normal prior on the coefficients,

$$
P(H) = P(\pmb{\theta}) = \prod_{j=1}^M B \exp\left(\frac{\theta_j^2}{2\sigma_2^2} \right).
$$

This says, my prior belief is that all coeffients are most probably zero but other values near zero are possible according to the specific shape of the Gaussian function, and they're all uncorrelated and have equal variance \\(\sigma_2^2\\). 

Putting the normal error distribution together with the normal prior on coefficients, and skipping over the simple algebra, the log-posterior is

$$
- \log{P(f(\pmb{\theta}; \pmb{x})|\{y, \pmb{x}\})} = \frac{1}{2\sigma^2}\sum_i (y_i-f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{1}{2\sigma_2^2} \lVert\pmb{\theta}\rVert_2^2 + \textrm{terms independent of }\pmb{\theta}.
$$

I've absorbed the probability distribution coeffients into the \\(\pmb{\theta}\\)-independent term. Compare this to the Scikit-learn objective function.

## Interpreting the Bayesian formulation

It's pretty obvious the negative log-posterior looks a lot like the Scikit-learn objective function. If you multiply the entire negative log-posterior by \\(2\sigma^2/N\\) you get \\(U\\) back exactly as

$$
U(\pmb{\theta}) = - \frac{2\sigma^2}{N} \log{P(f(\pmb{\theta}; \pmb{x})|\{y, \pmb{x}\})} = \frac{1}{N}\sum_i (y_i-f(\pmb{\theta}; \pmb{x}_i))^2 + \frac{1}{N}\frac{\sigma^2}{\sigma_2^2} \lVert\pmb{\theta}\rVert_2^2 + \textrm{terms independent of }\pmb{\theta}.
$$

so long as you define the regularization parameter as

$$
\lambda = \frac{2}{N}\frac{\sigma^2}{\sigma_2^2}.
$$

This form leads to the Bayesian interpretation of the Ridge penalty. If you standardize your data and fix the sample size then \\(\sigma^2=1\\) and \\(N\\) is a fixed constant. Then the L2 hyperparameter is proportional to the inverse variance of the width of your prior over coefficients. 

**Choosing a small \\(\lambda\\) is like asserting a weak prior belief on the range of possible coefficient values, centered on zero.** In the extreme case of \\(\lambda\to0\\), the width of the Gaussian prior grows to become a flat prior and you end up doing ordinary least squares.

**Choosing a large \\(\lambda\\) is like asserting a strong prior belief on the range of possible coefficient values, centered on zero.** In the extreme case of \\(\lambda\to\infty\\), the width of the Gaussian prior shrinks to become a Dirac delta function, and you're effectively saying you know exactly what the coefficients should be and you might as well not be doing an additional regression.

Ridge regression is therefore the same as a Bayesian model with a specific normal likelihood and a specific normal prior, and the prior makes little intuitive sense unless it is very weak, in which case you're essentially doing ordinary least-squares regression.

## LASSO's prior

LASSO is extremely closely related to Ridge. The only difference is a Laplace prior instead of Gaussian.

$$
P(H) = P(\pmb{\theta}) = C \exp\left(\frac{\lVert\pmb{\theta}\rVert_1}{\sigma_1} \right).
$$

The Laplace distribution is peakier at 0, but otherwise behaves somewhat like the Gaussian prior of Ridge regression.

## The tragedy of L0 regularization 

Since we're counting down in L-numbers, let's talk about L0 regularization. The L0 norm is the number of nonzero values of a vector. You might naively want to write down a prior as 

$$
P(H) = P(\pmb{\theta}) = \prod_{j=1}^M C \exp\left(\frac{\lVert\pmb{\theta}\rVert_0^0}{\sigma_1} \right).
$$

It's not obvious to me the is right, and here's why. To me the statement "some number of items of a fixed length vector are nonzero" sounds a lot like a multinomial trial. A multinomial trial is like rolling a possibly biased \\(k\\)-sided dice \\(n\\) times, and the number of times a given side comes up is multinomially distributed, and a multinomial distribution is a function of factorials, not of \\(\exp(\cdot)\\). Here's the multinomial L0 prior:

$$
P(H) = P(\pmb{\theta}) =  
$$

Mapping this analogy to the L0 problem, the dice is the vector itself, \\(k\\) is the number of entries in the vector and \\(n=k\\).



It's not computationally feasible.

## Conclusion

Regularization is bad Bayesianism but good practice. I haven't demonstrated its usefulness -- I'll leave that for a future post.