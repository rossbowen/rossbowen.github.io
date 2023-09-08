---
title: "Confidence intervals in the wild"
description: "Some thoughts on the use of confidence intervals with non-technical audiences."
date: "2023-09-06"
---

Confidence intervals have always come up during interviews for analytical roles. I'm usually asked what a confidence interval is, and I'll respond along the lines of the [definition given by the Office for National Statistics](https://www.ons.gov.uk/methodology/methodologytopicsandstatisticalconcepts/uncertaintyandhowwemeasureit#confidence-interval).

> **Using a 95% confidence interval**
>
> A 95% confidence level is frequently used. This means that if we drew 20 random samples and calculated a 95% confidence interval for each sample using the data in that sample, we would expect that, on average, 19 out of the 20 (95%) resulting confidence intervals would contain the true population value and 1 in 20 (5%) would not. If we increased the confidence level to 99%, wider intervals would be obtained.

This definition aligns with that I was taught -- if we took many samples and created many 95% confidence intervals, we'd expect 95% of those intervals to contain the true value for the parameter of interest. For many X% confidence intervals, we'd expect X% of the intervals to contain the true value.

I've then been asked how I would help a non-technical audience interpret a specific confidence interval. The thing is, I've never felt that confidence intervals are a good thing to use with that kind of audience. Many [statisticians misinterpret them](https://statmodeling.stat.columbia.edu/2017/12/28/stupid-ass-statisticians-dont-know-goddam-confidence-interval/), so how we could expect a non-technical audience to use them correctly?

I've gotten into some awkward debates over this -- so here's me getting my thoughts together.

## What does a confidence interval *actually* tell you?

I'd see a confidence interval in the wild and feel a bit unsure of how I could interpret it. If the definition of a confidence interval describes a behavior across *many* intervals, what does it mean when we're presented with a *single* interval?

A quick google and we see some common interpretations of confidence intervals:

-   There's a 95% chance that the parameter of interest lies within a 95% confidence interval.
-   Confidence intervals are a range in which we think the true value is likely to lie.

I couldn't see how either of these followed from the definition we used earlier, about creating many intervals and on average X% of them containing the true value. I went searching and found many blog posts on the subject from [Andrew Gelman](https://statmodeling.stat.columbia.edu/), one of which introduced [this paper from Richard Morey et al.](https://statmodeling.stat.columbia.edu/wp-content/uploads/2014/09/fundamentalError.pdf) which has shaped my thoughts over the years.

------------------------------------------------------------------------

Take this example from some of the [ONS's COVID-19 reporting](https://www.ons.gov.uk/peoplepopulationandcommunity/healthandsocialcare/conditionsanddiseases/bulletins/coronaviruscovid19infectionsurveycharacteristicsofpeopletestingpositiveforcovid19uk/16november2022):

> The latest estimated rate of coronavirus (COVID-19) reinfections on 28 October 2022 was 42.8 per 100,000 participant days at risk (95% confidence interval: 42.0 to 43.6).

Here we have a single 95% interval reported alongside an estimate of the COVID-19 reinfection rate. Maybe this is one of the lucky intervals which contains the true reinfection rate, but maybe it's not. We don't know.

The idea behind the 95% confidence interval is to perform some procedure which, on average, returns intervals $(L, U)$ which contain the true parameter 95% of the time.

$$P(L(X) \leq \theta \leq U(X)) = 0.95$$

We write $L(X)$ and $U(X)$ to emphasise these are random variables which in practice will often depend on the data. The problem occurs when we try to replace $(L(X), U(X))$ with an actual interval we've obtained.

If $\theta$ was the true COVID-19 reinfection rate, $\theta$ is just some fixed number we don't know the value of. We can't say anything probabilistic about the value of $\theta$ (unless we start using Bayesian techniques).

But, if we replace $L(X)$ and $U(X)$ above with the obtained interval, the probability changes.

$$
  P(42.0 \leq \theta \leq 43.6) = \begin{cases}
    1, & \text{if } 42.0 \leq \theta \leq 43.6, \\
    0, & \text{otherwise}.
  \end{cases}
$$

Andrew Gelman provides an analogy in [a blog post](https://statmodeling.stat.columbia.edu/2016/11/23/abraham-lincoln-confidence-intervals/) to explain why the probability changes once you substitute actual values into the interval. To provide a similar analogy -- let's say 170cm is a fairly average male height -- we could imagine that half of men are taller and half are shorter than 170cm. Bob is 168cm tall.

It seems reasonable to say, if $X$ was the height of some arbitrary man, that the probability of that man being taller than 170cm is a half.

$$P(X > 170\text{cm}) = 0.5.$$

But we can't just replace $X$, which represents the height of some arbitrary man. with Bob's height and expect the probability to remain the same. If we condition the above probability to talk specifically about Bob, rather than some arbitrary man, then the probability becomes 0, as Bob is shorter than 170cm.

$$P(X > 170\text{cm} \mid X = \text{Bob's height}) = P(168\text{cm} > 170\text{cm}) = 0.$$

Jerzy Neyman ([who introduced confidence intervals in this 1937 paper](https://royalsocietypublishing.org/doi/epdf/10.1098/rsta.1937.0005)) was so clear about their interpretation:

> It will be noticed that in the above description the probability statements refer to the problems of estimation with which the statistician will be concerned in the future. In fact, I have repeatedly stated that the frequency of correct results will tend to $\alpha$. Consider now the case when a sample $E'$, is already drawn, and the calculations have given $\underset{\bar{}}{\theta}(E') = 1$ and $\bar{\theta}(E') = 2$. **Can we say that in this particular case the probability of the true value falling between 1 and 2 is equal to** $\alpha$?
>
> **The answer is obviously in the negative.** The parameter $\theta_1$ is an unknown constant, and no probability statement concerning its value may be made, that is except for hypothetical and trivial ones
>
> $$
> \begin{equation} \tag{21}
>   P(1 \leq \theta_1 \leq 2) = \begin{cases}
>     1, & \text{if } 1 \leq \theta_1 \leq 2, \\
>     0, & \text{if either } \theta_1 \leq 2 \text{ or } 2 \geq \theta_1.
>   \end{cases}
> \end{equation}
> $$

------------------------------------------------------------------------

To sum up -- we can't say that *all* confidence intervals give a range of likely values for some parameter, at least not from the definition of a confidence interval alone. *Some* confidence intervals may be sensible to interpret as a Bayesian credible intervals, at which point we could make statements about likely values.

Or as Morey et al. put it:

> In the case where data are normally distributed, for instance, there is a particular prior that will lead to a confidence interval that is numerically identical to Bayesian credible intervals computed using the Bayesian posterior (Jeffreys, 1961; Lindley, 1965). This might lead one to suspect that it does not matter whether one uses confidence procedures or Bayesian procedures. We showed, however, that confidence intervals and credible intervals can disagree markedly. The only way to know that a confidence interval is numerically identical to some credible interval is to prove it. The correspondence cannot -- and should not -- be assumed.

## Closing thoughts

I think that confidence intervals have a use, particularly when the procedures to create them provide intervals which help describe the uncertainty in an estimate. But I'd be nervous to include them when writing for non-technical audiences, mainly due to the risk of misinterpretation:

-   A specific 95% confidence interval doesn't cover the true value with 95% probability, but many readers will see an interval and assume that.
-   A confidence interval doesn't give a range of likely values for some parameter, unless it is appropriate to interpret it as a Bayesian credible interval -- which a non-technical audience wouldn't be familiar with. A reader may assume all intervals provide likely values, regardless of the underlying statistical model being applied.
-   The Morey et al. paper also covers how smaller confidence intervals don't necessarily mean more precise estimates, which is another common misinterpretation (with some more discussion [here](https://stats.stackexchange.com/questions/204530/what-do-confidence-intervals-say-about-precision-if-anything)). It's true for *some* intervals.
-   Also, readers may not appreciate the difference between precision and accuracy, and think an estimate is better and assign more trust to it just because it's precise, even if it's completely inaccurate.

Other gotchas I think make confidence intervals difficult to interpret:

-   What does it mean to be confident anyway? What is a confidence level? Do most people interpret being confident in a confidence interval as meaning the interval is likely to contain the true value?
-   There's some real mental gymnastics to remember that a 99% confidence interval is wider than a 95% confidence interval. A 99% interval is wider, so there's more uncertainty, but we're now 99% confident instead of 95% confident. 🤷 To me, being more confident would meaning being able to provide a narrower interval with less uncertainty.
-   When two confidence intervals do not overlap, it means that the difference between the two parameters is statistically significant. But when two confidence intervals do overlap, they may or may not be significantly different. Overlapping intervals are often interpreted as meaning that the difference is not statistically significant, but this is not always the case.

If I were to include confidence intervals in my writing then I'd expect the audience to have some level of technical knowledge, and I think it's important to include what procedure has been used to compute the interval. If we think it is reasonable to interpret the interval as a Bayesian credible interval, then we can tell the audience that's what we're doing, and also include information about the prior we've used as part of our methodology.

## More reading

Some posts from Andrew Gelman's blog:

-   [Problematic interpretations of confidence intervals](https://statmodeling.stat.columbia.edu/2014/03/15/problematic-interpretations-confidence-intervals/)
-   [Abraham Lincoln and confidence intervals](https://statmodeling.stat.columbia.edu/2016/11/23/abraham-lincoln-confidence-intervals/)
-   [How to interpret confidence intervals?](https://statmodeling.stat.columbia.edu/2017/03/04/interpret-confidence-intervals/)
-   [Confidence intervals, compatability intervals, uncertainty intervals](https://statmodeling.stat.columbia.edu/2022/04/05/confidence-intervals-compatability-intervals-uncertainty-intervals/)

Two papers from Richard Morey et al.

-   [Robust misinterpretation of confidence intervals](https://www.ejwagenmakers.com/inpress/HoekstraEtAlPBR.pdf)
-   [The fallacy of placing confidence in confidence intervals](https://statmodeling.stat.columbia.edu/wp-content/uploads/2014/09/fundamentalError.pdf)