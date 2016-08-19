This code allows Dynare to penalize contemporaneous cross-correlation in the shocks.

Code:

If you run the current mod file, it will apply a penalty of 1000 to the norm of the contemporaneous cross-correlation in the vcv matrix. To modify the magnitude of the penalty, simply change this line in the mod file:

options_.custom_penalty = 1000;

The code that performs this penalty is located in my dsge_likelihood.m file, on lines 845 - 878. Here I extract the shocks and calculate the norm. 

For testing purposes, this code is currently very noisy. This should help us see exactly how dynare is using the likelihood function. Every time dsge_likelihood() is run by dynare, it prints some variation of the following:
fval = 2733.9 (fval_original = 2203.3, Loss2 = 0.53054)
Or:
fval = 2203.3 (no penalty applied)

Results:

I have included two log files where I use dynare to estimate the mode of the posterior. In one log file, the penalty is 0, in the other it is 1000. You can replicate both of these logs using the current mod file, just change the penalty appropriately.

As expected, the parameter estimates change when the penalty is applied. To see the parameter estimates, scroll to the bottom of each log file.

But unfortunately I receive a warning from dynare when I estimate the mode of the posterior. See below. However, this warning occurs even without my modifications to dsge_likelihood(). I think this is an issue with estimation using Smets_Wouters_2007.mod.

Posterior Kernel Optimization Problem. 

 (minus) the hessian matrix at the "mode" is not positive definite!
=> posterior variance of the estimated parameters are not positive.
You should try to change the initial values of the parameters using
the estimated_params_init block, or use another optimization routine.

[Warning: The results below are most likely wrong!]

As a result, I think we should do more tests to check that the new code is working.

Questions:

What estimation should I try? 

I noticed that the original Smets_Wouters_2007.mod did not actually estimate anything. It had the following configuration: 

estimation(
optim=('MaxIter',200),
datafile=usmodel_data,
mode_file=usmodel_shock_decomp_mode,
mode_compute=0,
first_obs=1,
presample=4,
lik_init=2,
prefilter=0,
mh_replic=0,
mh_nblocks=2,
mh_jscale=0.20,
mh_drop=0.2, nograph, nodiagnostic, tex);

Since mode_compute=0, dynare does not estimate the mode of the posterior, it just loads it from the mode_file. Also since mh_replic = 0, dynare does not perform mcmc estimation. Indeed, this mod file only runs dsge_likelihood() approximately 3 or 4 times. In the mod file I am currently using, I have changed it to mode_compute=1.
