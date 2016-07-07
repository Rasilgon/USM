Statistical concepts, at the example of the coin flip model
===





Assume I'm trying to guess if a coin comes up heads or tails. I have had 10 trials, and 7x success.


```r
trials = 10
success = 8
```

What we want to know now is what my properties are regarding correctly guessing the outcome of the coin flip experiment. 

### The model

We will try to examine this data with a simple model that makes the following assumption: every time I throw the coin, I will have a probability of p to guess correctly. 

For a single trial, this model is called the bernoulli-model; for multiple trials, it's calle the binomial model. It's easy to calculate the probabilities by hand, but you can also get it in r via the dbinom function. 

### Overview

In the rest of this script, I will look at this data with the binomial model, applying the three most common inferential methods 

1. Maximum likelihood estimation
2. Null-hypothesis significance testing
3. Bayesian inference 

Each of these methods is consistent and calculates a sensible thing. But each of these three things are different. I'll highlight differences at the end of each application. 

## THE MLE ESTIMATE

The idea of maximum likelihood estimation (MLE) is to look for the set of parameters that would, under the given model assumption, lead to the highest probability to obtain the observed data. In our case we have only one parameter, the probability of success per flip. Let's plot this for different values and look for the maximum.  


```r
# parameters to check
parametervalues <- seq(0,1,0.001)
# get the probability for all parametervalues
likelihood <- dbinom(success,trials,parametervalues)

# plot results
plot(parametervalues, likelihood, type = "l")
legend("topleft", legend = c("Likelihood", "maximum"), col = c("black", "red"), lwd = 1)
MLEEstimate <- parametervalues[which.max(likelihood)]
abline(v=MLEEstimate, col = "red")
```

![](InferenceMethods_files/figure-html/unnamed-chunk-3-1.png)

### Confidence intervals

OK, the MLE is the best value, but what is often m

MLE confidence intervals are constructed by asking yourself:

If I would do the experiment many times, how would the estimate scatter, and how wide would I have to take the interval so that the true value is contained in the interval x% (typically 95%) under repreatedly performing the experiment?

The test statistics that can be used to do this are discussed, e.g., in https://onlinecourses.science.psu.edu/stat504/node/39. The result for a 1-parameter model is that the CI is at a log likelihood difference of 1.92


```r
plot(parametervalues, likelihood, type = "l")
legend("topleft", legend = c("Likelihood", "maximum", "CI"), col = c("black", "red", "green"), lwd = 1)
MLEEstimate <- parametervalues[which.max(likelihood)]
abline(v=MLEEstimate, col = "red")

confidence.level <- log(max(likelihood)) -1.92
leftCI <- parametervalues[which.min(abs(log(likelihood[1:which.max(likelihood)]) - confidence.level))]
abline(v=leftCI, col = "green")
rightCI <- parametervalues[which.min(abs(log(likelihood[which.max(likelihood):length(likelihood)]) - confidence.level)) + which.max(likelihood) -1]
abline(v=rightCI, col = "green")
```

![](InferenceMethods_files/figure-html/unnamed-chunk-4-1.png)

Note: there are also other methods to look at uncertainty with likelihoods, e.g. the profile likelihood, see discussion [here](http://stats.stackexchange.com/questions/77528/what-is-the-relationship-between-profile-likelihood-and-confidence-intervals)


### Outcome of a MLE 

* Best estimate (MLE)
* 95% CI --> if we would do the experiment over and over again, 95% of the CIs would contain the true value. NOTE: this is != saying: for a given dataset, the true value is in the CI with 95% probability!


## GETTING THE P-VALUE FOR FAIR COIN

want to get p-value for a smaller or equal result (1-tailed) given a fair coin p(k<=kobs|H0:p=0.5). Basically, we want the sum over the red bars


```r
barplot(dbinom(0:10, 10, 0.5), col = c(rep("grey", success ), rep("red", 11-success)))
```

![](InferenceMethods_files/figure-html/unnamed-chunk-5-1.png)

```r
line(pbinom(0:10,trials,prob = 0.5, lower.tail = F))
```

```
## 
## Call:
## line(pbinom(0:10, trials, prob = 0.5, lower.tail = F))
## 
## Coefficients:
## [1]   1.2640  -0.1373
```


We can get this with the cummulative distribution function in R


```r
pValue <- pbinom(success,trials,prob = 0.5, lower.tail = F)
```

but it is a bit tricky, because depeding on which side one wants to test, you have to add a -1 to the successes becaues of the discrete nature of teh data and the definition of the cummulative in R. You can try, but it's safer in practice to use the binom.test, which calculates the same values 


```r
binom.test(7,trials,0.5) # two sided 
```

```
## 
## 	Exact binomial test
## 
## data:  7 and trials
## number of successes = 7, number of trials = 10, p-value = 0.3438
## alternative hypothesis: true probability of success is not equal to 0.5
## 95 percent confidence interval:
##  0.3475471 0.9332605
## sample estimates:
## probability of success 
##                    0.7
```

Alternatively:


```r
binom.test(7,trials,0.5, alternative="greater") # testing for greater
binom.test(7,trials,0.5, alternative="less") # testing for less
```

### Side-note: multiple testing

Imagine there is no effect, but we keep on repeating the test 100 times. How often do you think will we find a significant effect?


```r
data= rbinom(100,10,0.5)
pValue <- pbinom(data,trials,prob = 0.5, lower.tail = F)
sum(pValue < 0.05)
```

```
## [1] 5
```

Yes, 5 is what you expect. To be exact, in the case of disrecte random distributions, the value doesn't have to be exactly 5%, but that is a side topic, and here it works. 

The message here is: if you do repeated tests, and you want to maintain a fixed overall type I error rate, you need to adjust the p-values, e.g. by 


```r
pValueAdjusted <- p.adjust(pValue, method = "hochberg")
sum(pValueAdjusted < 0.05)
```

```
## [1] 0
```

Remember: in general, if you choose an alpha level of 5%, and you have absolutely random data, you should get 5% false positives (type I error) assymptotically, and the distribution of p-values in repeated experiments will be flat. 

Also note: we are free to choose the null-hypothesis as we want. What would you do if you null hypothesis is that a coint should have an 0.8 proability of head?



### Outcome of a NHST 

* p-value --> probability to see the observed or more extreme data given the null hypothesis
* rejection H0 if p < alpha. If p > alpha, the test ist inconclusive
* if you do multiple tests, you may want to adjust the p-values

## The BAYESIAN ESTIMATE

Remember for Bayes p(M|D) = p(D|M) * p(M) / P(D), and we can show that p(D) is just the integral over p(D|M) * p(M)

We had already calculated p(D|M), so we just need to define p(M), the prior

for flat prior p(M) = flat = 1
See http://en.wikipedia.org/wiki/Jeffreys_prior , section on Bernoulli trial, to understand that this is not neccesarily the best uninforative choice, but it is simple at any rate


```r
prior <- rep(1,1001)
posterior <- likelihood * prior / sum(likelihood * prior) * length(parametervalues)

plot(parametervalues, posterior, col = "darkgreen", type = "l")
lines(parametervalues, likelihood)
lines(parametervalues, prior, col = "red" )
legend("topright", c("likelihood", "prior", "posterior"), col = c("black", "red", "green"), lwd = 1 )
```

![](InferenceMethods_files/figure-html/unnamed-chunk-11-1.png)

you see that likelihood and posterior have the same shape. However, this is only because I chose a flat prior. There is still a difference, however, namely that the posterior is normalized, i.e. will integrate to one. It has to be, because we want to interpret it as a pdf, while the likelihood is not a pdf. Let's look at the same example for an informative prior



```r
prior <- dnorm(parametervalues, mean = 0.5, sd = 0.1)
posterior <- likelihood * prior / sum(likelihood * prior) * length(parametervalues)

plot(parametervalues, posterior, col = "darkgreen", type = "l")
lines(parametervalues, likelihood)
lines(parametervalues, prior, col = "red" )
legend("topright", c("likelihood", "prior", "posterior"), col = c("black", "red", "green"), lwd = 1 )
```

![](InferenceMethods_files/figure-html/unnamed-chunk-12-1.png)

you can see that the likelihood moves the posterior away from the prior, but not by much. try the same think with more data, but the same ratio, i.e. change to 30 trials, 9 success


### Bayesian CI

calculated via central 95% probability


```r
plot(parametervalues, posterior, col = "darkgreen", type = "l")
lines(parametervalues, likelihood)
lines(parametervalues, prior, col = "red" )
legend("topright", c("likelihood", "prior", "posterior"), col = c("black", "red", "green"), lwd = 1 )

cumPost <- cumsum(posterior) / (length(posterior) -1)

leftCI <- parametervalues[which.min(abs(cumPost - 0.025))]
abline(v=leftCI, col = "darkgreen", lty = 2, lwd = 2)
leftCI <- parametervalues[which.min(abs(cumPost - 0.975))]
abline(v=leftCI, col = "darkgreen", lty = 2, lwd = 2)
```

![](InferenceMethods_files/figure-html/unnamed-chunk-13-1.png)


---
**Copyright, reuse and updates**: By Florian Hartig. Updates will be posted at https://github.com/florianhartig/LearningBayes. Reuse permitted under Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License

