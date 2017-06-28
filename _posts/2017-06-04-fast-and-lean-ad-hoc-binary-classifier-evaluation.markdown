---
title: "Fast and Lean Ad Hoc Binary Classifier Evaluation"
date: 2015-03-08 14:05:20 -0700
comments: true
author: "Will High"
categories: statistics, streaming algorithms
---

{% include toc %}

# Motivation

In this post I’m going to see if I can beat [perf](http://osmot.cs.cornell.edu/kddcup/software.html), which is a C++ utility I’ve used that computes precision, recall, AUC, etc.

A scenario where this would come up: you’re a data scientist who’s done binary classification on a fairly large data set in an ad hoc proof of concept analysis, and you want to compute the usual evaluation metrics as fast as possible. Should you use perf, or roll your own? 

I’ll be exploring streaming computation of some useful metrics:

  1. Single-pass precision, recall, accuracy, lift and F1-score.
  1. Single-pass (plus a little) AUC.
  1. Single-pass RMSE.

# Data, procedure and result

I grabbed the biggest data set in Chih-Jen Lin‘s perpetually handy lib-SVM repository. It’s the KDD Cup 2010 Bridge to Algebra data set with the winning team’s best feature set. Big is good because I want an inefficient algorithm to hurt. Some stats about the data:

* 19,264,097 training entities
* 748,401 test entities
* 29,890,095 dimensions

I count 30-ish nonzero features per entity on average, for a part in a million sparsity.

The procedure (laid out in detail in the Procedure section below) is to download the data, unzip it, train the training set, score the scoring set and then compute evaluation metrics. In this post I’m focused on the algorithm performance in evaluation metrics computation in that final stage.

Time-wise, it looks like I can beat perf in computing precision, recall, accuracy, lift, F1-score and RMSE. I can also beat it in AUC computation (a wholly different beast) as long as I can accept just 3 or 4 decimals of accuracy, which is usually more than enough. If I want another digit of AUC accuracy, all else being equal, I can eat just one more algebraic calculation per entity by using the trapezoidal rule rather than a Riemann sum.

I can always beat perf in evaluating 0.5 million entities or more. Without having dug into the perf source code, I do not know offhand why the developer imposed that upper limit.

And since I’m totally beating up on perf, I’ll also reveal that I can do all this with two orders of magnitude fewer lines of code. All my implementations are in gawk.

The full, end-to-end pipeline using the best algos takes 421s. Training is 37% of the total time (it takes more time to unzip than to train) and everything downstream from scoring to post-processing to evaluation takes < 2% of the total time. Here are the timings of each step of the analysis.

| Step | Time |
| ---- | ---- |
| Download training set | 1m 21.510s |
| Download evaluation set | 0m 5.281s |
| Unzip training set | 2m 44.246s |
| Unzip evaluation set | 0m 6.451s |
| Train training set | 2m 35.673s |
| Score evaluation set | 0m 4.764s | 
| Paste and logistic-transform evaluation data | 0m 1.975s |
| Subsample to 500,000 entities | 0m 0.212s |
| Fixed threshold eval (perf, 500,000 entities) | 0m 0.297s |
| Fixed threshold eval (awk, 500,000 entities) | 0m 0.252s |
| Fixed threshold eval (perf, all entities) | – |
| Fixed threshold eval (awk, all entities)	| 0m 0.357s |
| AUC eval (perf, 500,000 entities)	| 0m 0.296s |
| AUC eval (awk, 500,000 entities, 3 decimals) | 0m 0.278s | 
| AUC eval (perf, all entities)	| - |
| AUC eval (awk, all entities, 3 decimals) | 0m 0.424s |
| RMSE eval (perf, 500,000 entities) | 0m 0.263s |
| RMSE eval (awk, 500,000 entities)	 | 0m 0.227s |
| RMSE eval (perf, all entities) | - |
| RMSE eval (awk, all entities)	| 0m 0.347s |

## Procedure

### Download

<!-- <script src="https://gist.github.com/fwhigh/218a1a9582945acbcd2c196ef542f90b.js?file=get_data.sh"></script>
 -->
{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="get_data.sh" data_gist_line="3-100" %}

### Train

Convert and stream to vw format and train.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="train.sh" data_gist_line="3-100" %}

```
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
final_regressor = model.vw
using no cache
Reading datafile = 
num sources = 1
average    since         example     example  current  current  current
loss       last          counter      weight    label  predict features
0.693147   0.693147            1         1.0   1.0000   0.0000       26
0.950634   1.208120            2         2.0  -1.0000   0.8532       23
0.764694   0.578754            4         4.0  -1.0000  -0.5043       25
0.826334   0.887974            8         8.0   1.0000  -0.1111       23
0.736578   0.646821           16        16.0   1.0000   0.7315       41
0.668554   0.600530           32        32.0   1.0000   0.9360       29
0.560276   0.451999           64        64.0   1.0000   2.2459       23
0.470294   0.380312          128       128.0   1.0000   2.1648       25
0.384321   0.298348          256       256.0  -1.0000   1.2838       32
0.338601   0.292881          512       512.0   1.0000   3.7020       35
0.353033   0.367465         1024      1024.0   1.0000  -0.2938       20
0.404202   0.455370         2048      2048.0  -1.0000  -0.0990       59
0.407307   0.410412         4096      4096.0  -1.0000   1.5638       35
0.412617   0.417928         8192      8192.0   1.0000   3.0536       21
0.375746   0.338875        16384     16384.0   1.0000   4.2012       35
0.390718   0.405689        32768     32768.0   1.0000   2.8355       32
0.360178   0.329639        65536     65536.0   1.0000   0.3467       34
0.354580   0.348982       131072    131072.0   1.0000   2.2873       38
0.343257   0.331934       262144    262144.0   1.0000   1.7774       47
0.339802   0.336347       524288    524288.0   1.0000   3.6705       36
0.345376   0.350950      1048576   1048576.0   1.0000   0.2121       26
0.323524   0.301671      2097152   2097152.0   1.0000   2.2170       35
0.315078   0.306633      4194304   4194304.0   1.0000  -0.1576       29
0.307970   0.300861      8388608   8388608.0   1.0000   3.3606       23
0.309194   0.310418     16777216  16777216.0  -1.0000  -0.8738       33

finished run
number of examples per pass = 19264097
passes used = 1
weighted example sum = 1.92641e+07
weighted label sum = 1.38952e+07
average loss = 0.31017
best constant = 0.721302
total feature number = 585609985
```

### Score

Convert to vw format and score.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="score.sh" data_gist_line="3-100" %}

```
Num weight bits = 18
learning rate = 0.5
initial_t = 0
power_t = 0.5
predictions = kddb.t_scores.txt
using no cache
Reading datafile = 
num sources = 1
average    since         example     example  current  current  current
loss       last          counter      weight    label  predict features
0.096423   0.096423            1         1.0   1.0000   2.2904       20
0.062938   0.029453            2         2.0   1.0000   3.5102       21
0.043892   0.024845            4         4.0   1.0000   3.6864       23
0.027345   0.010798            8         8.0   1.0000   4.3615       23
0.469994   0.912643           16        16.0  -1.0000  -0.5099       59
0.382650   0.295306           32        32.0   1.0000   4.6787       35
0.263003   0.143355           64        64.0   1.0000   5.2928       22
0.367205   0.471407          128       128.0   1.0000   3.9544       35
0.397651   0.428097          256       256.0   1.0000   5.3137       33
0.347004   0.296358          512       512.0  -1.0000   3.1846       35
0.324560   0.302116         1024      1024.0   1.0000   2.5141       20
0.329715   0.334869         2048      2048.0   1.0000   3.9218       25
0.341305   0.352895         4096      4096.0   1.0000   9.2050       23
0.318626   0.295948         8192      8192.0   1.0000   0.6735       35
0.302794   0.286961        16384     16384.0   1.0000   0.4491       35
0.306975   0.311156        32768     32768.0   1.0000   2.1338       25
0.310069   0.313162        65536     65536.0   1.0000   7.8721       23
0.297661   0.285254       131072    131072.0   1.0000   3.9228       25
0.291197   0.284732       262144    262144.0  -1.0000   3.4579       35
0.299229   0.307261       524288    524288.0   1.0000   4.9444       29

finished run
number of examples per pass = 748401
passes used = 1
weighted example sum = 748401
weighted label sum = 580347
average loss = 0.30118
best constant = 0.775449
total feature number = 22713476
```

### Paste evaluation data

I’ll paste together the scores and the true classes for the downstream evaluation exercises. This is mainly for API compliance… strictly speaking you can skirt this step in a real production implementation as long as you’re continually associating entities with their true classes and their scores.

I’m going to apply the logistic transformation on the raw scores here so that scores are probabilities in [0,1].

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="logistic_prob.sh" data_gist_line="3-100" %}


### Evaluate fixed-threshold metrics with perf

I’ll compute precision, recall, accuracy, lift and F1-score with perf.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="perf_fixed_thresh_eval.sh" data_gist_line="3-100" %}

```
Aborting.  Exceeded 500000 items.
```

Fail. I need to subsample down to < 500,000.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="subsample.sh" data_gist_line="3-100" %}

```
ACC    0.88585   pred_thresh  0.500000
PRE    0.90768   pred_thresh  0.500000
REC    0.97002   pred_thresh  0.500000
PRF    0.93782   pred_thresh  0.500000
LFT    1.02285   pred_thresh  0.500000
```

### Try to beat it with awk

When I first wrote this I did not think it would beat perf. At minimum I just thought I could evaluate on the full scoring set.

Here I run on on the subsampled data.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="awk_fixed_thresh_eval.sh" data_gist_line="3-100" %}

```
ACC	0.885848
PRE	0.907684
REC	0.97002
PRF	0.937817
LFT	1.02285
```

Running on the full evaluation set works fine.

```
ACC	0.88651
PRE	0.908369
REC	0.970005
PRF	0.938176
LFT	1.02326
```

### Evaluate AUC, a threshold-sweep metric, with perf

AUC, the area under the receiver operating characteristic curve, requires sweeping the threshold across the range of scores. A sweep is also needed in exercises where you want to look at any of the other metrics as a function of reach, for example, when maximizing some notion of return on investment.

Perf calls AUC “ROC,” and here it is on the subsampled data.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="perf_auc_eval.sh" data_gist_line="3-100" %}

```
ROC    0.79828
```

Again, it doesn’t want to compute AUC on the full evaluation set.

### Try to beat it in awk

Naively, the sweep screams “sort,” which is annoying. Sorting the subsampled data takes more than 1s, so I can’t do that and expect to beat perf.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="sort.sh" data_gist_line="3-100" %}

Without looking into the perf source code, I’m just going to try to beat it another way.

I’ll bin the scores into 10^{decimals} decimal places, and count positive and negative instances per bin. This is effectively an empirical probability distribution, or a histogram. I’ll assume the scores are in [0,1], which is generically a strong assumption, but I know my data conforms to this in this case.

Here are the two passes:

1. First I’ll pass over the full data set of size N, counting positive and negative instances by bin. For the benchmark set, N=500,000.
1. The second pass will be over the 10^{\text{decimals}} bins, where I’ll compute
  A. the cumulative true-positives and false-positives, which is proportional to the respective cumulative distribution functions, and
  A. a streaming AUC as a discrete Riemann sum (imagine the ROC curve parameterized by score, not by the false-positive rate).

Here it is:

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="awk_auc_eval.sh" data_gist_line="3-100" %}

```
ROC	0.799652
```

I’ll benchmark using the average of 10 runs after a few warm starts (this is how I do all the evaluation metric benchmarks).

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="auc_benchmark.sh" data_gist_line="3-100" %}

Perf baseline is 270ms with AUC 0.79828.

| Decimals | AUC | Error | Time | Speedup |
| -------- | --- | ----- | ---- | ------- |
| 1 | 0.877566 | 0.079286 | 247ms | +23ms |
| 2 | 0.810426 | 0.012146 | 252ms | +18ms |
| 3 | 0.799652 | 0.001372 | 248ms | +22ms |
| 4 | 0.798422 | 0.000142 | 280ms | +10ms |
| 5 | 0.798296 | 0.000016 | 376ms | +106ms |

So I can get 3 or 4 digits of accuracy in essentially the same time as perf. I cannot get close to perf at 5 digits of accuracy, which suggests to me I didn’t come up with the most efficient AUC algo, or perf is also doing an approximate AUC computation, or …?

In any case, here’s AUC on the full data set.

```
ROC 0.801368
```

I could deal with far more entities, but I would eventually run into integer overflows in the cardinality computations, in which case I would move toward parallel computation and/or efficient probabilistic data structures.

I tested the trapezoidal rule, which gave me another order of magnitude in agreement with perf with only a small increase of compute time. There’s one additional sum per entity.

### Evaluate RMSE with perf

Another fixed-threshold metric. I’m pursuing this because this was the KDD 2010 eval metric, but normally I would not use RMSE for a classification problem.

Here’s perf.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="perf_rmse.sh" data_gist_line="3-100" %}

```
RMS    0.29650
```

And rolling my own.

{% include gist_embed.html data_gist_id="fwhigh/218a1a9582945acbcd2c196ef542f90b" data_gist_file="auc_rmse.sh" data_gist_line="3-100" %}

```
RMS	0.296497
```

And RMSE on the full data set.


```
RMS	0.295575
```

# What did we learn?

Ok, a disclaimer: I admit it’s total BS to claim not to look under the hood in an exercise like this when there’s no mechanism to enforce that, but whatever, this was fun and I really didn’t so let’s go with that. Peaking at the header of the perf source code now, I see a MAX_BINS global variable that is set to 1000, which is about the level my timing equals that of perf, so maybe it is in fact doing an AUC approximation like mine.

But we learned how to stream binary evaluation metrics efficiently, and even one continuous metric. In this particular case I saved very little real time in terms of human productivity, however, for a much larger data set I now know how I might approach these computations in a parallel compute environment. In this case the time saved might be palpable. I can speak from experience that really does come up, and any time saved waiting for a cluster to finish a job I submitted is hugely valuable.

That said, this is not quite how you would want to implement these evaluation metrics in a true Big Data setting, but more on that in a future post.

