---
layout: blog
title: "Options Pricing Greeks delta, gamme, vega, theta, rho."
subtitle: "Greeks show the sensitivity of an option price depending on the  change of  a single parameter."
cover_image: images/blog/random.jpg
cover_image_caption: "Random"
tags: [finance, math, probability, python]
---

**Option Delta**:

By definition, delta of an option is the amount of underlying asset increase/decrease causing the certain amount change in option price. In mathematical term it’s the first derivative with respect to the underlying asset’s price.

The delta of a call option is: $$CDelta = \frac{\partial C}{\partial S} = N(d1)$$

With the put-cal parity, the delta of put option is: $$PDelta = N(d1) - 1$$

**Option Gamma**:

When the underlying moves so does the Delta value on any option. This rate of change of Delta resulting from movement of the underlying is known as Gamma.

The gamma of an option is: $$Gamma = \frac{\partial^2 C}{\partial S ^2} = \frac{N'(d1)}{\sigma S\sqrt(T)}$$

**Option Vega**:

When any position is taken in options, not only is there risk from changes in the underlying but there is risk from changes in volatility. Vega is the measure of that risk. When the underlying changes, or even if it does not in some cases, implied volatility levels may change.

The vega of an option is: $$Vega = \frac{\partial C}{\partial \sigma } = SN'(d1)\sqrt(T)$$

**Option Theta**:

Change in OPTION price with respect to change in Term (T). This is known as time decay. This is always negative. As time passes, the option and gets closer to expiration, it becomes less valueable. Time erodes the value of the option.

The theta of a call option is: $$CTheta = \frac{\partial C}{\partial t} = - \frac{SN'(d1)\sigma }{2\sqrt(T-t)} - rKe^{-r(T-t)}N(d2)$$

The theta of a put option is: $$PTheta = - \frac{SN'(d1)\sigma }{2\sqrt(T-t)} + rKe^{-r(T-t)}N(-d2)$$

**Option Rho**:

Rho is a risk measure related to changes in interest rates.

The rho of a call option is: $$CRho = \frac{\partial C}{\partial r} = K(T-t)e^{-r(T-t)}N(d2)$$

The rho of a put option is: $$PRho = -K(T-t)e^{-r(T-t)}N(-d2)$$


N.B: We will keep the same notations from the previous post!

```python
import scipy
import math
from scipy import stats

#delta
cDelta = lambda s,k,v,r,t,div : math.exp(-div*t) * stats.norm.cdf(d1(s,k,v,r,t,div))
pDelta = lambda s,k,v,r,t,div : math.exp(-div*t) * (stats.norm.cdf(d1(s,k,v,r,t,div)) - 1)

#gamma
gamma = lambda s,k,v,r,t,div : math.exp(-div*t) * stats.norm.pdf(d1(s,k,v,r,t,div))/(s * v * scipy.sqrt(t))

#vega
vega = lambda s,k,v,r,t,div : s * math.exp(-div*t) * stats.norm.pdf(d1(s,k,v,r,t,div)) * scipy.sqrt(t)

#theta
cTheta = lambda s,k,v,r,t,div : -(s * v * stats.norm.pdf(d1(s,k,v,r,t,div)) / (2 * scipy.sqrt(t))) + \
        (div * s * math.exp(-div*t) * stats.norm.cdf(d1(s,k,v,r,t,div))) - \
        (r * k * math.exp(-r*t) * stats.norm.cdf(d2(s,k,v,r,t,div)))
pTheta = lambda s,k,v,r,t,div : -(s * v * stats.norm.pdf(d1(s,k,v,r,t,div)) / (2 * scipy.sqrt(t))) - \
        (div * s * math.exp(-div*t) * stats.norm.cdf(d1(s,k,v,r,t,div))) + \
        (r * K * math.exp(-r*t) * stats.norm.cdf(-d2(s,k,v,r,t,div)))

#rho
cRho =  lambda s,k,v,r,t,div : k * t * math.exp(-r*t) * stats.norm.cdf(d2(s,k,v,r,t,div))
pRho =  lambda s,k,v,r,t,div : -k * t * math.exp(-r*t) * stats.norm.cdf(-d2(s,k,v,r,t,div))
```
