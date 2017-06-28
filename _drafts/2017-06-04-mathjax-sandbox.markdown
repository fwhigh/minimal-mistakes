---
title: "MathJax Sandbox"
date: 2015-03-08 14:05:20 -0700
comments: true
categories: 
---

# [http://haixing-hu.github.io/programming/2013/09/20/how-to-use-mathjax-in-jekyll-generated-github-pages/](http://haixing-hu.github.io/programming/2013/09/20/how-to-use-mathjax-in-jekyll-generated-github-pages/)

\begin{equation}
f=ma
\end{equation}

And $f=ma$.

# [http://gastonsanchez.com/visually-enforced/opinion/2014/02/16/Mathjax-with-jekyll/](http://gastonsanchez.com/visually-enforced/opinion/2014/02/16/Mathjax-with-jekyll/)

Let’s try a first example. Here’s a dummy equation:

$$a^2 + b^2 = c^2$$

Here is an example MathJax inline rendering \\( 1/x^{2} \\), and here is a block rendering: 
\\[ \frac{1}{n^{2}} \\]

$$ \mathbf{X}_{n,p} = \mathbf{A}_{n,k} \mathbf{B}_{k,p} $$

To display inline math use `\\( ... \\)` like this `\\( sin(x^2) \\)` which gets rendered as \\( sin(x^2) \\).

# [https://kramdown.gettalong.org/syntax.html#math-blocks](https://kramdown.gettalong.org/syntax.html#math-blocks)

$$
\begin{align*}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{align*}
$$

The following is a math block:

$$ 5 + 5 $$

But next comes a paragraph with an inline math statement:

\$$ 5 + 5 $$

If you don’t even want the inline math statement, escape the first two dollar signs:

\$\$ 5 + 5 $$

# Gists

<script src="https://gist.github.com/benbalter/5555251.js"></script>
<script src="https://gist.github.com/benbalter/5555251.js?file=gist.md"></script>
