

# regife (Bai 2009)

The command `regife` estimates models with interactive fixed effects (Bai 2009). 


### Syntax

`regife` requires a formula and the option `factors`, composed of an id variable, a time variable, and the dimension.

```
insheet "data/cigar.csv", clear
regife sales price, f(state year, 3)
```




### Absorb
Impose id or time fixed effect with the option `absorb`. 

```
regife sales price, f(state year, 2)  a(state year)
```
In my experience, the convergence is much faster when id or time fixed effects are specified.




### Unbalanced Panel
The command handles unbalanced panels (ie missing observation for a given id, time) as described in the appendix of Bai 2009. In this case,  standard errors should be estimated by bootstrap.



### Standard errors
The `vce` option allows to compute robust standard errors 

```
regife sales price, f(state year, 2) a(state year) vce(cluster state) 
```

Except for bootstrap, the `vce` option is simply passed to the command regressing y on x and covariates of the form `i.id#c.year` and `i.year#c.id` (as discussed in section 6 of of Bai 2009).

In presence of correlation, the estimate for beta isbiased (See Theorem 3 in Bai 2009). Instead of computing robust standard errors, you may want to add enough factors until residuals are i.i.d.



In my experience, it is wiser to bootstrap the standard errors, [especially for small T](monte-carlo/result.png)
```
regife sales price, f(state year, 2)  vce(bootstrap, reps(100))
```

To obtain standard errors by block bootstrap:
```
regife sales price, f(state year, 2)  vce(bootstrap, cluster(state))
```


### Weights
Weights are supported but should be constant within id


### Convergence
The iteration algorithm can be modified using the option `tolerance` (default to 1e-9) or `maxiteration` (default to 10000).



### Save factors
Save loadings and/or factors by specifying new variable names using `=`

```
regife sales price, a(fe = state) f(loading_state=state factor_year=year, 2) 
```

To obtain residuals, directly use the option `residuals`


```
regife sales price, f(state year, 2) residuals(newres)
```


### Speed
`regife` can be quite slow since typically, a high number of iterations is required until convergence. 

- You can start the convergence at a given `beta` using `bstart`
- id fixed effects or time fixed effects tend to speed up the convergence
- The [same algorithm](https://github.com/matthieugomez/FixedEffectModels.jl) is available in Julia, and is 10x faster



# ife
The command `ife` estimates a factor model for some variable

- Contrary to Stata `pca` command, 
 - `ife` handles dataset in long forms (ie `ife` works on datasets where each row represents an id and a time, rather than datasets where each row represents an id and each variable a time).
 - `ife` handles unbalanced panel data : in the first step, missing observations are set to zero and a factor model is estimated.  In a second step, missing observations are replaced by the predicted value of the factor model, etc until convergence. This corresponds to the algorithm described in Stock and Watson (1998).


- To save loadings and/or the factors, use the lhs of `=`
 ```
 ife sale, f(loading_state=state factor_year=year)  d(2)
 ```

 If you want to obtain residuals, you can directly use the `residuals` option

 ```
 ife sale, f(state year, 2) residuals(p30_res)
 ```

- By default, `ife` does not demean the variable. If you want to estimate a PCA, you probably want to demean the variable with respect to id and/or time. To do so, use the option `absorb`. 


 ```
 ife sale, a(fe_state = state fe_year = year) f(factors = state loading = year, 2) 
 ife sale, a(state year) f(state year, 2) residuals(p30_res)
 ```





# cce (Pesaran 2006)

The command `ccemg` and `ccep` correspond respectively to Pesaran (2006) Common Correlated Effects Mean Group estimator (CCEMG) and Common Correlated Effects Pooled estimator (CCEP). 

Like with year fixed effect, these commands generate the mean value of regressors at each time accross groups. and add them as regressor. After this step,
- `ccemg` runs the new model within each id and averages the betas accross all ids. 
- `ccep` runs the new model on the pooled sample, interacting the newly created variables with group dummies. 

`ccep` relies on `reghdfe` and is generally faster than `ccemg`.




# Installation

### regife
`regife` is now available on ssc
```
ssc install regife
```


To install the latest version  on Github with Stata13+
```
net install regife, from(https://github.com/matthieugomez/stata-regife/raw/master/)
```

With Stata 12 or older, download the zipfiles of the repositories and run in Stata the following commands:
```
net install regife, from("SomeFolderRegife")
```

### requirement

`regife` requires [`reghdfe` and `hdfe`](https://github.com/sergiocorreia/reghdfe) with version 3.0+

```
ssc install reghdfe
ssc install hdfe
```