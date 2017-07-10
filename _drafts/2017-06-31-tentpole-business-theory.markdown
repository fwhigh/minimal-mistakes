---
title: "Tentpole Business Theory"
date: 2017-06-29 12:00:00 -0700
comments: true
categories: 
  - statistics
tags:
  - business models
  - Monte Carlo
---

{% include toc %}

# Hollywood is a Tentpole Industry

Hollywood is famously a "tentpole" industry. Paramount's recent tentpoles are Star Trek, Transformers and Mission Impossible. Warner Brothers' are DC Comics movies like Batman and most recently Wonder Woman---though, importantly, some of them have been unexpectedly flopping. Sony has been floundering landing a tentpole. Disney can't be stopped.

Closely tied with the notion of tentpoles in Hollywood is the franchise or the "universe." The Marvel Universe of Thor and Iron Man and Captain America, which all reference one another and recycle the same talent, consistently generates tentpoles. Disney, Marvel Studios' owner, is an absolute tentpole factory. But not all universes generate tentpoles so consistently, as in the Warner Brothers example. Dark Knight is quite possibly the greatest superhero movie of all time, and, tragically, no subsequent Batman movie has remotely lived up to it. 

So is the universe the tentpole, or does the universe sometimes generate a tentpole movie? I subscribe to the latter thinking. I think universes that contain a tentpole have a higher likelihood of generating another one, as long as there is a skilled studio team executing on it. Owning a universe---James Bond, Jason Bourne, Harry Potter---and paying top dollar to execute well are ways to increase your odds.

# Generalizing the Concept

Ok, this is Hollywood, but there are other tentpole industries. Take early stage venture capital. A single company can carry the entire portfolio by offsetting the losses of all other companies in the portfolio. A tentpole can make a VC firm.

This leads me to a more general definition:

> A tentpole business is one that invests in a portfolio of assets, and only a small fraction of the assets generate a profit, offsetting the losses of the rest.

Some businesses do not survive in a tentpole industry because they never land a big enough tentpole.

I have a hypothesis about tentpole industries, which is that they are a fundamental consequence of efficient market dynamics in a marketplace that contains assets whose returns have right-tailed distributions.

<figure>
    <a href="/assets/tentpole-business-theory/distro_of_returns.png"><img width="70%" src="/assets/tentpole-business-theory/distro_of_returns.png" /></a>
    <figcaption width="70%">The hypothesized distribution of returns of all possible assets for a tentpole industry, scaled roughly to Hollywood movies.</figcaption>
</figure>

Here's the R code:

```R
library(ggplot2)
library(data.table)

thm <- theme_bw()  
thm <- thm + theme(axis.line = element_line(colour = "black"),
                   panel.grid.major = element_blank(),
                   panel.grid.minor = element_blank(),
                   panel.border = element_blank(),
                   panel.background = element_blank())

x <- seq(0, 2e9, length.out=1e3)
meanlog <- log(1e8)
sdlog <- 1
meanreal <- exp(meanlog + sdlog^2/2)
sdreal <- sqrt(exp(sdlog^2) - 1)*meanreal
df <- data.table(x=x,
                 y=dlnorm(x, meanlog = meanlog, sdlog = sdlog))
p <- ggplot(df, aes(x/1e6, y)) + geom_line()
p <- p + xlab("Million USD")
p <- p + ylab("Probability density")
p <- p + thm
print(p)
ggsave(p, file="distro_of_returns.png")
```

# Theoretical Fundamentals of a Tentpole Industry


