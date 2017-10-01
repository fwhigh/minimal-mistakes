---
title: "Debiasing Regularized Regression"
date: 2017-07-05 12:00:00 -0700
comments: true
categories: 
  - machine learning
tags:
  - regularization
  - regression
  - bias
---

# tl;dr

To debias Ridge, LASSO, or Elastic Net least-squares regression do this:
1. Run the usual regularized regression with hyperparameter optimization.
2. On a held-out data set, run another ordinary least squares regression with predictions from step 1 as the input.
3. Predict on any new out-of-sample data by predicting on the model from step 1 and inputting the result to the model from step 2.

Mathematically,

$$
\begin{equation}
\hat f_{\mathrm{debiased}}(\pmb{x}) = b_0 + b_1 (\hat w_0 + \pmb{\hat{w}}\cdot\pmb{x})
\end{equation}
$$

where the right-hand parentheses contain the initial regularized regression model that learns coefficients $(\hat w_0 + \pmb{\hat{w}})$ over data $\pmb{x}$, and the downstream debiasing OLS learns $(b_0, b_1)$ to produce the final prediction $\hat f_{\mathrm{debiased}}$.


**This may deliver additional variance reduction beyond just step 1** in addition to the debiasing.

In this post I run this procedure on a simulated, sparse, underdetermined data set (10 times more dimensions than training examples) and demonstrate that, under the simulation settings, Elastic Net model does $7.8\times$ better than OLS but with bias, and the debiased Elastic Net model does another $1.2x$ better than the plain Ealstic Net and with no bias.

{% include toc %}

# Setup and Simulated Data

Regularization reduces the generalization variance of high dimensional regression models at the expense of bias. This is commonly called the bias-variance tradeoff of Ridge, LASSO, and Elastic Net, and it's often a net gain in real machine learning settings. The bias, however, can be modeled and eliminated in such a way that not only doesn't hurt the variance, but even improves it.

I'll demonstrate by simulating a data set $\\{\pmb{x}\\}$ of $N=1000$ points in a $D=10\,000$ dimensional space, and generating a target variable as

$$
\begin{equation}
y = f(\pmb{x}) + \epsilon,
\end{equation}
$$

where 

$$
\begin{equation}
f(\pmb{x}) = w_0 + \pmb{w}\cdot\pmb{x}
\end{equation}
$$

and the error is a random variable,

$$
\begin{equation}
\epsilon \sim \mathrm{Normal}(0,0.01).
\end{equation}
$$

I'll make the data sparse with a random $1\%$ of $D$ being nonzero for each data point. I'll have $100$ of the predictors be predictive of the target variable, i.e., nonzero and equal to $1$, and I'll set the intercept to $1$. 

{% include gist_embed.html data_gist_id="fwhigh/806ed8f8c1737d64efada44bbd6d05c7" data_gist_file="simulate_regression_data.py" %}

<figure>
    <a href="/assets/debiasing-regularized-regression/simulated_data.png"><img src="/assets/debiasing-regularized-regression/simulated_data.png" /></a>
    <figcaption>Summary of the simulated regression data.</figcaption>
</figure>

The true mean and variance of the model errors are respectively $0$ and $0.01$ by construction, and estimates from my realized test data are $0.00035 \pm 0.00316$ and $0.00998 \pm 0.00045$.

# Regularized Regression with Elastic Net

Scikit-learn's Elastic Net fits an objective function

$$
\begin{equation}
\frac{1}{2N}\sum_{i=1}^N(y - \hat f(\pmb{x}))^2 + \alpha\rho\lVert\pmb{w}\rVert_1 + \frac{\alpha(1-\rho)}{2}\lVert\pmb{w}\rVert_2^2
\end{equation}
$$

where $\alpha$ is the strength of regularization and $\rho$ is the L1 ratio controlling the relative amount of L1 and L2 regularization. The linear model is estimated as

$$
\begin{equation}
\hat f(\pmb{x}) = \hat w_0 + \pmb{\hat{w}}\cdot\pmb{x}.
\end{equation}
$$

<figure>
    <a href="/assets/debiasing-regularized-regression/simulated_data_elasticnet_regression_result.png"><img src="/assets/debiasing-regularized-regression/simulated_data_elasticnet_regression_result.png" /></a>
    <figcaption>Elastic Net regression result. The line fit with confidence interval shows clear bias.</figcaption>
</figure>

The best regularization hyperparameters came out as $\alpha=0.000177$ and $\rho=1.0$, which is pure L1 regularization. 

The fitted model coefficient is $w_0 = 0.992$, $860$ of the $10\,000$ coefficients came back as nonzero (96 are actually nonzero in my realization), and the histogram of nonzero coefficients is

<figure>
    <a href="/assets/debiasing-regularized-regression/simulated_data_elasticnet_regression_coef_hist.png"><img src="/assets/debiasing-regularized-regression/simulated_data_elasticnet_regression_coef_hist.png" /></a>
    <figcaption>Histogram of nonzero Elastic Net regression coefficients.</figcaption>
</figure>

The mean-squared test error is $0.110$. 

# Ordinary Least Squares and Dummy Estimators

To baseline the Elastic Net result, I'll run ordinary least-squares like this.

<figure>
    <a href="/assets/debiasing-regularized-regression/simulated_data_ordinary_regression_result.png"><img src="/assets/debiasing-regularized-regression/simulated_data_ordinary_regression_result.png" /></a>
    <figcaption>OLS regression result.</figcaption>
</figure>


which gives me a mean-squared test error of $0.861$. Perfect coefficient estimation would have given MSE of $0.010$, which was the variance input in my error model. A simple model that is the mean of the training data gives MSE $0.984$, and a simple median model would give $0.986$.

So the OLS predictions are better than a very naive model but regularized regression is substantially better. Regularization did indeed give me improved variance at the expense of bias in the coefficients and the out-of-sample predictions.

# Regularization Debiasing

The debiasing procedure is

$$
\begin{equation}
\hat f_{\mathrm{debiased}}(\pmb{x}) = b_0 + b_1 \hat f(\pmb{x})
\end{equation}
$$

The ultimate, unbiased prediction is $\hat f_{\mathrm{debiased}}(\pmb{x})$.

<figure>
    <a href="/assets/debiasing-regularized-regression/simulated_data_debiased_reg_regression_result.png"><img src="/assets/debiasing-regularized-regression/simulated_data_debiased_reg_regression_result.png" /></a>
    <figcaption>Debiased regularized regression result.</figcaption>
</figure>

This gives me an MSE of $0.092$, which is better than the Elastic Net model alone.

The bias values in this realization are $b_0 = -0.152$ and $b_1 = 1.146$.

# Repeat

Here's how I ran this simulation.

```bash
mkdir -p data
rm data/simulated_data*
python simulate_regression_data.py
python debiased_reg.py --experiment_name simulated_data --train_data_filename simulated_data_train --test_data_filename simulated_data_test
```

The `debiased_reg.py` script follows.

{% include gist_embed.html data_gist_id="fwhigh/806ed8f8c1737d64efada44bbd6d05c7" data_gist_file="debiased_reg.py" %}

I can't be sure a single iteration of the experiment is not subject to strong sampling uncertainty under my parameter settings, so I want to run many simulations.


# Public Machine Learning Data Sets

[http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/)



# Discussion

This was a simple Monte Carlo sketch of the debiasing idea. The concept is intuitive and the simulation panned out essentially as expected, but it remains to be proven rigorously why this works. An interesting wrinkle is that this procedure should debias generalization *predictions* but not the original, regularized linear *coefficients*. So while it is obvious to me that $(\hat w_0, \pmb{\hat{w}})$ are biased estimates of $(w_0, \pmb{w})$ in the presence of L1 and/or L2 regularization terms, it's not obvious to me that $b_1(\hat w_0, \pmb{\hat{w}})$ are unbiased estimates, even if the ultimate predictions are unbiased.


Regularization also induces bias in logistic regression, where it is partially responsible for RLR's famous uncalibrated probabilities problem. My gut tells me Platt-scaling may reduce some or all bias and may also give variance improvements, like it did in this regression setting. 

Full disclosure: I do not understand why I do not see this debiasing procedure in common usage. I do not have an encyclopedic knowledge of the machine learning literature, so if anybody would like to point to papers that do what I've described and show improvement in generalization error, I'd greatly appreciate it. 

# Full code

{% include gist_embed.html data_gist_id="fwhigh/806ed8f8c1737d64efada44bbd6d05c7" data_gist_file="debiased_reg.py" %}

{% include gist_embed.html data_gist_id="fwhigh/806ed8f8c1737d64efada44bbd6d05c7" data_gist_file="debiased_reg.sh" %}
