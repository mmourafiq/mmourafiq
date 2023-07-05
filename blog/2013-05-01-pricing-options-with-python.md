---
layout: blog
title: "Pricing options with python."
subtitle: "Vanilla European puts/calls"
cover_image: images/blog/random.jpg
cover_image_caption: "Random"
tags: [finance, math, probability, python]
---

The black-scholes formula is a must-know  differential equation in financial mathematics. It is used for various purposes to price financial derivatives. In this post we will price vanilla European puts/calls.

The differential equation is given by:

$$\frac{\partial V}{\partial t} + rS \frac{\partial V}{\partial S} + 1/2 \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} - rV = 0$$

where:

 * V –> price of the derivative
 * S –> price of the underlying asset
 * r –> risk-free interest rate
 * σ –> asset volatility of the underlying S

**The pricing formula**:

Depending on the derivative being priced, different conditions will apply, and consequently the equation will have different solutions.

The price of a European Vanilla Call, C(S,t) is given below:

$$C(S,t) = SN(d1) - k e^{-rT}N(d2)$$


With d1 and d2 defined as follows:

$$d1 = \frac{log(S/K) + (r + \sigma^2/2)T}{\sigma \sqrt{T}}$$

$$d2 = d1 - \sigma \sqrt{T}$$

And thanks to put-call parity, we are able to price a European Vanilla Put P(S,T), as well, with the following formula:

$$P(S,T) = Ke^{-rT} - S + C(S,T)$$

The only thing we need to describe is N, which is the [cumulative distribution function of the standard normal distribution](http://en.wikipedia.org/wiki/Normal_distribution#Cumulative_distribution_function), N is given by:

$$N(x) = 1/\sqrt{2\pi}\int_{-\inf}^x e^{-t^2/2}dt$$

Since we will like to have a closed form solutions for the “Greeks”, i.e. the option price sensitivities to the underlying variables and parameters. For this reason we also need the formula for the [probability density function of the standard normal distribution](http://en.wikipedia.org/wiki/Normal_distribution#Probability_density_function) which is given below:

$$f(x) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2}$$

**Python implementation**:

```python
import scipy
import math
from scipy import stats

""" Price an option using the Black-Scholes model.
s: initial stock price
k: strike price
t: expiration time
v: volatility
r: risk-free rate
div: dividend
cp: +1/-1 for call/put
"""

d1 = lambda s,k,v,r,t,div : (scipy.log(s/k)+(r-div+0.5*math.pow(v,2))*t)/(v*scipy.sqrt(t))
d2 = lambda s,k,v,r,t,div : d1(s,k,v,r,t,div) - v*scipy.sqrt(t)
call = lambda s,k,v,r,t,div : (s*math.exp(-div*t)*stats.norm.cdf(d1(s,k,v,r,t,div))) - (k*scipy.exp(-r*t)*stats.norm.cdf(d2(s,k,v,r,t,div)))
put =  lambda s,k,v,r,t,div : (k*math.exp(-r*t)*stats.norm.cdf(-d2(s,k,v,r,t,div))) - (s*math.exp(-div*t)*stats.norm.cdf(-d1(s,k,v,r,t,div)))

#for checking purpose this two functions will assert true if the pricing is correct in case of vanilla options
check_call = lambda s,k,r,t,call_price : k * math.exp( -r * t ) + call_price - s
check_put = lambda s,k,r,t,put_price : s + put_price - k * math.exp( -r * t )
```

In a next post we will see how to calculate the sensitivity of the price to the underlying variables using the “Greecs”.
