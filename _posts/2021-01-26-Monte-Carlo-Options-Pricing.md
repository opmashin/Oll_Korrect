---
layout: post
---

This post Discusses How you can use Monte Carlo Sampling to derive the Implied Volatily of an Options Contract.

## Monte Carlo Method.

Monte Carlo is method of computation that relies on repeated random sampling to derive a probabilistically accurate numerical result.
This method is useful for simulating a complex environment that would otherwise require complicated math.
This method favors clarity, ease of implementation and calculation over precision.
For usecases like IV calculation for Options, since we already take in assumptions with Black Scholes models, Monte Carlo can actually not be that bad.

## Deriving Options Premium.

The premium price of any options contract embodies the risk that the option writer is willing to take so that the buyer has the right to either excercise the contract or not.
With the Monte Carlo method, we simulate the possible outcomes for the contract on the expiry day based on 100s of random underlying price movement.
Calculate the possible premium value at expiry based on the 100s of the random Prices at expiry, take the mean of them and voila you have an approximate premium value

generating random Price movement given, days till expiry, current price and Standard deviation:

```
def generateRandomPriceMovement( init_price, days, sigma=1):
    price_list = [ init_price ]
    rand_vals = np.random.normal(0, sigma, days)
    for i in range(days):
        rand_val = rand_vals[i]
        price_list.append(price_list[-1]+rand_val)
    return price_list
```

Calculating Premium based on 100 random price moves

```
def premiumCalculator( price, days, sigma):
    expiry_premium = []
    init_price = price
    for i in range(100):
        price_list = generateRandomPriceMovement(init_price, 10)
        expiry_premium.append(price_list[-1]-init_price if init_price < price_list[-1] else 0)
    return np.mean(expiry_premium)
```

Now we have the premium, we now iterate through the possible sigma values to find the the one that provides a premium value closest to the current premium in the market.
and use that sigma to  calculate the IV.  
IV can be derived from the following formula

![std to iv](https://raw.githubusercontent.com/opmashin/Oll_Korrect/master/images/std_to_iv.png)

```python
def ivCalculator( price, days, act_premium):
    expiry_prices = []
    implied_sigma = 0
    implied_premium = 0
    sigmas = range(500)
    for sigma in sigmas:
        premium = premiumCalculator(price, days, sigma)
        if abs(premium-act_premium) < abs(implied_premium - act_premium):
            implied_premium = premium
            implied_sigma = sigma
    return (implied_sigma*np.sqrt(365/days)*100/price)
```
