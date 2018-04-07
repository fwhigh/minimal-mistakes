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

Hollywood is a tentpole industry. Paramount's recent tentpoles are Star Trek, Transformers and Mission Impossible. Warner Brothers' are DC Comics movies like Batman and more recently Wonder Woman---though, importantly, some have been unexpectedly flopping. Sony has struggled landing a tentpole. Disney can't be stopped. Now I'm just guessing that these are each studios' tentpoles. To confirm I'd have to know not just the box office but the full return-on-investment each of these movies is generating.

> Tentpoles are those few assets in a portfolio (movies or TV shows in the case of Hollywood) that generate a return on investment that is so great that it makes up for the losses of many other items in the portfolio.

Closely tied with the notion of tentpoles in Hollywood is the franchise or the **universe**. The Marvel Universe of Thor and Iron Man and Captain America, which all reference one another and recycle the same talent, consistently generates tentpoles. Disney, Marvel Studios' owner, is a tentpole juggernaut. But not all universes generate tentpoles so consistently, as in the Warner Brothers example. Dark Knight is quite possibly the greatest superhero movie of all time, and, tragically, no subsequent Batman movie has lived up to it. 

So is the universe the tentpole, or does the universe sometimes generate a tentpole movie? I subscribe to the latter thinking. I think universes that contain a tentpole have a higher likelihood of generating another one, as long as there is a skilled studio team executing on it and budgets are huge. Owning a universe---James Bond, Jason Bourne, Harry Potter---and paying top dollar to execute well probably increase your odds of creating a tentpole movie.

# Generalizing the Concept

That's Hollywood, but there are other tentpole industries. Early stage venture capital is one. A single company can carry the entire portfolio by offsetting the losses of all other companies in the portfolio. A tentpole startup can make a VC firm. These days I believe we're calling these unicorns.

Some businesses do not survive in a tentpole industry because they never land a big enough tentpole. I might argue this happens when they don't make enough bets and/or the bets are big enough.

This hypothesis is grounded in a model I carry in my head, which is that tentpole industries are any that operate in a marketplace that contains assets whose returns have right-tailed distributions. Let's dig into that thinking.

http://www.boxofficemojo.com/alltime/world/

https://stephenfollows.com/how-many-films-are-released-each-year/

http://www.the-numbers.com/market/

<figure>
    <a href="/assets/tentpole-business-theory/distro_of_returns.png"><img width="70%" src="/assets/tentpole-business-theory/distro_of_returns.png" /></a>
    <figcaption width="70%">The hypothesized distribution of returns of all possible assets for a tentpole industry, scaled roughly to Hollywood movies.</figcaption>
</figure>

Here's the R code:

```R
library(ggplot2)
library(data.table)
library(fitdistrplus)

thm <- theme_bw()
thm <- thm + theme(axis.line = element_line(colour = "black"),
                   panel.grid.major = element_blank(),
                   panel.grid.minor = element_blank(),
                   panel.border = element_blank(),
                   panel.background = element_blank())

box_office_df <- fread('tmp/box_office_mojo_all_time_box_office.tsv')
box_office_df$Worldwide <- as.numeric(box_office_df[, gsub("[\\$,]","",Worldwide)])
box_office_df$Year <- as.numeric(box_office_df[, gsub("\\^","",Year)])

n_movies_since_1975 <- 400*40
df_cens <- data.table(left=c(box_office_df$Worldwide, rep(0,n_movies_since_1975)),
                      right=c(box_office_df$Worldwide, rep(min(box_office_df$Worldwide),n_movies_since_1975)))
#plotdistcens(df_cens,Turnbull = FALSE)
f_w <- fitdistcens(df_cens, "weibull")
f_ln <- fitdistcens(df_cens, "lnorm")
f_g <- fitdistcens(df_cens, "gamma")
cdfcompcens(list(f_w,f_ln,f_g), 
            legendtext=c("Weibull","log-normal","gamma"), 
            ylim=c(0.9,1), 
            xlogscale=F)

ggplot(df_cens, aes(left)) +
  geom_histogram(aes(y = ..density..)) +
  stat_function(fun = dlnorm, 
                args = list(meanlog = coef(f_ln)[[1]], sdlog = coef(f_ln)[[2]]), 
                col = 'red') +
  stat_function(fun = dweibull, 
                args = list(shape = coef(f_w)[[1]], scale = coef(f_w)[[2]]), 
                col = 'blue') +
  stat_function(fun = dgamma, 
                args = list(shape = coef(f_g)[[1]], rate = coef(f_g)[[2]]), 
                col = 'green')




p <- ggplot(df, aes(x/1e6, y)) + geom_line()
p <- p + geom_histogram(data=box_office_df, aes(x=Worldwide,y=..density..), alpha=0.2)
p <- p + xlab("Million USD")
p <- p + ylab("Probability density")
p <- p + thm
print(p)
ggsave(p, file="distro_of_returns.png")
```

# Theoretical Fundamentals of a Tentpole Industry


