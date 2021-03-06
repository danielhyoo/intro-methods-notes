
# Weighted Regression

## Weighted Least Squares (WLS)

Ordinary least squares estimates coefficients by finding the coefficients that minimize the sum of squared errors,
$$
\hat{\vec\beta}_{OLS} = \argmin_{\vec{b}} \sum_{i = 1}^N (y_i - \vec{x}\T \vec{b})^2 .
$$

Note that the objective function treats all observations equally; an error in one is as good as any other.

However, there are several situations where we care more about minimizing some errors more than others.
The next situation will discuss the reasons to use WLS, but 

Weighted least squares (WLS) requires only a small change to the OLS objective function.
Each observation is given a weight, $w_i$, and the *weighted* sum of squared errors is minimized,
$$
\begin{aligned}[t]
\hat{\vec\beta}_{WLS} = \argmin_{\vec{b}} \sum_{i = 1}^N w_i (y_i - \vec{x}\T \vec{b})^2
\end{aligned}
\hat{\beta}_{WLS} = \argmin_{\vec{b}} \sum_{i = 1}^N w_i (y_i - \vec{x}\T \vec{b})^2 .
$$
The weights $w_i$ are  provided by the analyst and are not estimated.
Note that OLS is a special case of WLS where $w_i = 1$ for all the observations.
In order to minimize the errors, WLS will have to fit the line closer to observations with higher weights



You can estimate WLS by using the `weights` argument to `rdoc("stats", "lm")`.

## When should you use WLS?

The previous section showed what WLS is, but when should you use weighted regression?

It depends on the purpose of your analysis:

1. If you are estimating population descriptive statistics, then weighting is needed to ensure that the sample is representative of the population.
2. If you are concerned with causal inference, then weighting is more nuanced. You may or may not need to weight, and it will often be unclear which is better.

There are three reasons for weighting in causal inference [@SolonHaiderWooldridge2015a]:

1. To correct standard errors for heteroskedasticity
2. Get consistent estimates by correcting for endogenous sampling
3. Identify average partial effects when there is unmodeled heterogeneity in the effects.

*Heteroskedasticity:* Estimate OLS and WLS. If the model is misspecified or there is endogenous selection, then  OLS and WLS have different probability limits. The contrast between OLS and WLS estimates is a diagnostic for model misspecification or endogenous sampling.  Always use robust standard errors.

*Endogenous sampling:* If the sample weights vary exogenously instead of endogenously, then weighting may be harmful for precision. The OLS still specifies the conditional mean. Sampling is exogenous if the sampling probabilities are independent of the error - e.g. if they are only functions of the explanatory variables. If the probabilities are a function of the dependent variable, then they are endogenous. 

- if sampling rate is endogenous, weight by inverse selection.
- use robust standard errors. 
- if the sampling rate is exogenous, then OLS and WLS are consistent. 
    Use OLS and WLS as test of model misspecification.

*Heterogeneous effects:* Identifying average partial effects. WLS estimates the linear regression of the population, but this is not the same as the average partial effects. But that is because OLS does not estimate the average partial effect, but weights according to the variance in X.

@AngristPischke2009a [p. 92] suggest weighting when "they make it more likely that the regression you are estimaing is close to the population target you are trying to estimate".

- sampling weights: yes
- grouped data (sums, averages): yes
- heteroskedasticity: no (just use robust standard errors)

  - WLS can be more efficient than OLS if variance model is correct
  - If $E(e_i | X)$ is a poor approximation or measurements are noisy, WLS has bad finite sample properties
  - If the CEF is not linear, then OLS and WLS are both wrong, but OLS still interpretable as the minimum mean squared error approximation of the CEF. The WLS is also an approx of CEF, but approx is a function of the weights.
  
Advice of [@CameronTrivedi2010a, p. 113]. There are two approaches to using weights.

- Census parameter: Reweight regression to try to get the population regression estimates.
- Control function approach: Assuming that $\E(\epsilon_i | \vec{x}_i) = 0$, then weights are not needed. WLS is consistent for any weights, and OLS is more efficient. This means if we control for all covariates relevant to sampling probabilities, there is no need to weight. This works as long as sampling probabilities are a function of $x$ and not of $y$.

It seems that @CameronTrivedi2010a "census parameter" approach is what @AngristPischke2009a interprets it as, but it supports the model based approach.
  
Weights should be used for predictions and computing average MEs. [@CameronTrivedi2010a, p. 114-115].

@Fox2016a [p. 461]: inverse probability weights are different than weights in heteroskedasticity, and WLS cannot be used. It will give the wrong SEs but correct point estimates. Seems to suggest using bootstrapping to get standard errors instead [@Fox2016a, p. 661, 666].

## Correcting for Known Heteroskedasticity

Most generally, heteroskedasticity is "unknown" and robust standard errors should be used.

However, there are some cases where heteroskedasticity is "known". For example:

- The outcome variable consists of measurements with a given measurement error - perhaps they are estimates 
    themselves.
- The error of the output depends on input variables in known ways. For example, the sampling error of polls. 

Examples: 

-  $\E(\epsilon)_i^2 \propto z_i^2$ where $a$ is some observated variable. Then $w_i = z_i$.
-  $\E(\epsilon)_^2$ is an average of values. Then $\sigma^2_i = \omega^2 / n_i$. In WLS, $w_i = 1 / \sqrt{n_i}$.
-  $\E(\epsilon)_^2$ is the sum of values. Then $\sigma^2_i = n_i \omega^2 $. In WLS, $w_i = \sqrt{n_i}$.
- If $p_i^{-1}$ is the inverse-sampling probability weight, then weight by $w_i$
    
Suppose that the heteroskedasticity is known up to a *multiplicative* constant,
$$
\Var(\varepsilon_i | \mat{X}) = a_i \sigma^2 ,
$$
where $a_i = a_i \vec{x}_i\T$ is a positive and known function of $\vec{x}_i$.

Define the weighting matrix,
$$
\mat{W} =
\begin{bmatrix}
1 / \sqrt{a_1} & 0 & \cdots & 0 \\
0 & 1 / \sqrt{a_2} & \cdots & 0 \\
\vdots & \vdots & \ddots & 0 \\
0 & 0 & \cdots & 1 / \sqrt{a_N}
\end{bmatrix}, 
$$
and run the regression,
$$
\begin{aligned}[t]
\mat{W} y &= \mat{W} \mat{X} \vec{\beta} + \mat{W} \vec\varepsilon \\
\vec{y}^* &= \mat{X}^* \vec{\beta} + \vec{\varepsilon}^* .
\end{aligned}
$$
Run the regression of $\vec{y}^*$ on $\mat{X}^*$, and the Gauss-Markov assumptions are satisfied.
Then using the usual OLS formula,
$$
\hat{\vec\beta}_{WLS} = ((\mat{X}^*)' \mat{X}^*) (\mat{X}^*)' \vec{y}^* = (\mat{X}' \mat{W}' \mat{W} \mat{X})^{-1} \mat{X}' \mat{W}' \mat{W} \vec{y} .
$$


## Sampling Weights

Sampling weights are the inverse probabilities of selection that are used to weight a sample
to be representative of population (as if were a random draw from the population).

In this situation, whether to use sampling weights depends on whether you are calculating 

If you are calculating a descriptive statistic from the sample as an estimator of a population parameter, you need to use weights

- if sample weights are a function of $X$ only, estimates are unbiased and more efficient without weighting
- if the sample weights are a function of $Y | X$, then use the weights

With fixed $X$, regression does not require random sampling, so the sampling weights of the $X$ are irrelevant.

If the original unweighted data are homoskedastic, then sampling weights induces heteroskedasticity.
Suppose the true model is,
$$
Y_i = \vec{x}\T \vec{\beta} + \varepsilon_i
$$
where $\varepsilon_i \sim N(0, \sigma^2)$.
Then the weighted model is,
$$
\sqrt{w_i} Y_i = \sqrt{w_i} \vec{x}\T \vec{\beta} + \sqrt{w_i} \varepsilon_i
$$
and now $\sqrt{w_i} \varepsilon_i \sim N(0, w_i \sigma^2)$.

If the sampling weights are only a function of the $X$, then controlling for $X$ is sufficient.
In fact, OLS is preferred to WLS, and will produce unbiased and efficient estimates.
The choice between OLS and WLS is a choice between different distributions of $\mat{X}$.
However, if the model is specified correctly the coefficients should be the same, regardless
of the distribution of $\mat{X}$.
Thus, if the estimates of OLS and WLS differ, then it is evidence that the model is misspecified.

@WinshipRadbill1994a suggest using the method of @DumouchelDuncan1983a to test whether the OLS and WLS are difference.

1. Estimate $E(Y) = \mat{X} \beta$
2. Estimate $E(Y) = \mat{X} \beta + \delta \vec{w} + \vec{\gamma} \vec{w} \mat{X}$,
    where all $X$
3. Test regression 1 vs. regression 2 using an F test.
4. If the F-test is significant, then the weights are not simply a function of $X$. Either try to respecify the model or use WLS with robust standard errors. 
    If the F-test is insignificant, then the weights are simply a function of $X$. Use OLS.

Modern survey often use complex multi-stage sampling designs.
Like clustering generally, this will affect the standard errors of these regressions.
Clustering by primary sampling units is a good approximation of the standard errors from multistage sampling.


## References

The WLS derivation can be found in @Fox2016a [p. 304-306, 335-336, 461]. Other textbook discussions: @AngristPischke2009a [p. 91-94], @AngristPischke2014a, [p. 202-203], @DavidsonMacKinnon2004a [p. 261-262], @Wooldridge2012a [p. 409-413].

@SolonHaiderWooldridge2015a is a good (and recent) overview with practical advice of when to weight and when not-to weight linear regressions. Also see the advice from the [World Bank blog](http://blogs.worldbank.org/impactevaluations/tools-of-the-trade-when-to-use-those-sample-weights).
See also @Deaton1997a, @DumouchelDuncan1983a, and @Wissoker1999a.



@Gelman2007a, in the context of post-stratification, proposes controlling for variables related to selection into the sample instead of using survey weights; also see the responses [@BellCohen2007a; @BreidtOpsomer2007a; @Little2007a; @Pfeffermann2007a], and rejoinder [@Gelman2007b] and [blog post](http://andrewgelman.com/2015/07/14/survey-weighting-and-regression-modeling/).
Gelman's approach is similar to that earlier suggested by @WinshipRadbill1994a.

For survey weighting, see the R package **[survey](https://cran.r-project.org/package=survey)**.
