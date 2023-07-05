---
layout: blog
title: "Sharp ratio."
subtitle: "Calculating sharp ratio with python."
cover_image: images/blog/stock-market.jpg
cover_image_caption: "Stock market."
tags: [ python, finance ]
---

A basic approach to optimizing the risk and return of a portfolio is to maximize its Sharpe ratio, which is the ratio of its expected return to its standard deviation. Standard deviation is a measure of how much you can expect the portfolio to go up or down, so the Sharpe ratio measures how much money you’re expecting to make for each dollar you’re risking. In theory, you can always borrow money at some interest rate, so it’s best to make the investment with the highest Sharpe ratio. That way you can maximize the amount you’re expecting to make without risking more than you’re comfortable with.

Let’s start by calculating the sharp ratio of one equity, and for this example we will use Apple (AAPL).

```python
import scipy
from yahoo_finance import YahooFinance

VALUE_TYPE = {'Date': 0, 'Open': 1, 'High': 2, 'Low': 3, 'Close': 4, 'Volume': 5, 'Adj_Close': 6}
start_date = "20121101"
end_date = "20121201"
yf = YahooFinance("AAPL")

# get historical data from yahoo finance
for (symb, data) in yf.get_historical_prices(start_date, end_date):
    data.reverse()  # reverse the time series
    data.pop()  # get rid of the header

"""
our data look like this :
 ['Date'       'Open'   'High'   'Low'    'Close'  'Volume'   'Adj Clos']
 ['2012-11-01' '598.22' '603.00' '594.17' '596.54' '12903500' '593.8']
 ['2012-11-02' '595.89' '596.95' '574.75' '576.80' '21406200' '574.1']
 ['2012-11-05' '583.52' '587.77' '577.60' '584.62' '18897700' '581.9']
 ['2012-11-06' '590.23' '590.74' '580.09' '582.85' '13389900' '580.2']
 ['2012-11-07' '573.84' '574.54' '555.75' '558.00' '28344600' '558.0']
 ['2012-11-08' '560.63' '562.23' '535.29' '537.75' '37719500' '537.7']
 ['2012-11-09' '540.42' '554.88' '533.72' '547.06' '33211200' '547.0']
 ['2012-11-12' '554.15' '554.50' '538.65' '542.83' '18421500' '542.8']
 ['2012-11-13' '538.91' '550.48' '536.36' '542.90' '19033900' '542.9']
 ['2012-11-14' '545.50' '547.45' '536.18' '536.88' '17041800' '536.8']
 ['2012-11-15' '537.53' '539.50' '522.62' '525.62' '28211100' '525.6']
 ['2012-11-16' '525.20' '530.00' '505.75' '527.68' '45246200' '527.6']
 ['2012-11-19' '540.71' '567.50' '539.88' '565.73' '29404200' '565.7']
 ['2012-11-20' '571.91' '571.95' '554.58' '560.91' '22955500' '560.9']
 ['2012-11-21' '564.25' '567.37' '556.60' '561.70' '13321500' '561.7']
 ['2012-11-23' '567.17' '572.00' '562.60' '571.50' '9743800' '571.5']
 ['2012-11-26' '575.90' '590.00' '573.71' '589.53' '22520700' '589.5']
 ['2012-11-27' '589.55' '590.42' '580.10' '584.78' '19047500' '584.7']
 ['2012-11-28' '577.27' '585.80' '572.26' '582.94' '18602300' '582.9']
 ['2012-11-29' '590.22' '594.25' '585.25' '589.36' '18382100' '589.3']
 ['2012-11-30' '586.79' '588.40' '582.68' '585.28' '13975700' '585.2']
"""
# since we are only interested by the Adjusted Closing Price
adj_prices = scipy.array(map(lambda x: float(x[VALUE_TYPE['Adj_Close']]), data))

>> [593.8  574.1  581.9  580.2  558.   537.7  547.   542.8  542.9  536.8
    525.6  527.6  565.7  560.9  561.7  571.5  589.5  584.7  582.9  589.3
    585.2]

total_return = (adj_prices[-1] / adj_prices[
    0]) - 1  # calculate the total return for apple this month
>> -0.014482990906

daily_return = (adj_prices[1:] / adj_prices[0:-1]) - 1  # calculate the daily_return
>> [-0.03317615  0.01358648 - 0.00292146 - 0.03826267 - 0.03637993  0.01729589
    - 0.00767824  0.00018423 - 0.01123596 - 0.02086438  0.00380518  0.0722138
    - 0.00848506  0.00142628  0.01744704  0.03149606 - 0.00814249 - 0.0030785
    0.01097958 - 0.00695741]

avg_daily_return = scipy.mean(daily_return, 0)  # average daily retuen
>> -0.000437386088059

std_daily_return = scipy.std(daily_return, 0)  # deviation fo daily return
>> 0.0243327696358

sharp_ratio = scipy.sqrt(250) * (avg_daily_return / std_daily_return)  # sharp ratio
>> -0.284212663794
```

Now we can move to a more complicated example, where we will deal 4 equities, let’s consider Apple (AAPL), Google (GOOG), Gold (GLD), US oil (USO).

```python
import scipy
from yahoo_finance import YahooFinance


VALUE_TYPE = {'Date': 0, 'Open': 1, 'High': 2, 'Low': 3, 'Close': 4, 'Volume': 5, 'Adj_Close': 6}
start_date = "20121101"
end_date = "20121201"

yf = YahooFinance("AAPL,GOOG,GLD,USO")
percent = scipy.array([0.25, 0.15, 0.4, 0.2]) #investment distribution portfolio

all_adj_prices = []
all_total_return = []
all_daily_return = []
all_avg_daily_return = []
all_std_daily_return = []
all_sharp_ratio = []

ticker = 0
for (symb, data) in yf.get_historical_prices(start_date, end_date):
    data.reverse() #reverse the time series
    data.pop() #get rid of the header
    adj_prices = scipy.array(map(lambda x : float(x[VALUE_TYPE['Adj_Close']]), data))
    all_adj_prices.append(adj_prices)
    total_return = (adj_prices[-1]/adj_prices[0]) - 1 #calculate the total return for apple this month
    all_total_return.append(total_return)
    daily_return = (adj_prices[1:]/adj_prices[0:-1]) - 1 #calculate the daily_return
    all_daily_return.append(daily_return)
    avg_daily_return = scipy.mean(daily_return, 0) #average daily retuen
    all_avg_daily_return.append(avg_daily_return)
    std_daily_return = scipy.std(daily_return, 0) #deviation fo daily return
    all_std_daily_return.append(std_daily_return)
    sharp_ratio = scipy.sqrt(250) * (avg_daily_return / std_daily_return) # sharp ratio
    all_sharp_ratio.append(sharp_ratio)

  """
  our prices
  GOOG : 687.5,  687.9,  682.9,  681.7,  667.1,  652.2,  663. ,  665.9,
        659. ,  652.5,  647.2,  647.1,  668.2,  669.9,  665.8,  667.9,
        661.1,  670.7,  683.6,  691.8,  698.3

  USO : 32. ,  31.3,  31.6,  32.5,  31.2,  31.3,  31.7,  31.5,  31.4,
        31.7,  31.4,  31.9,  32.6,  31.9,  32.1,  32.3,  32.1,  31.9,
        31.7,  32.1,  32.5

  AAPL : 593.8,  574.1,  581.9,  580.2,  558. ,  537.7,  547. ,  542.8,
        542.9,  536.8,  525.6,  527.6,  565.7,  560.9,  561.7,  571.5,
        589.5,  584.7,  582.9,  589.3,  585.2

  GLD : 166. ,  162.6,  163.2,  166.3,  166.4,  167.9,  167.8,  167.4,
        167.1,  167.1,  166. ,  165.8,  167.8,  167.3,  167.5,  169.6,
        169.4,  168.7,  166.5,  167.1,  166.

  total return for each stock : [0.015709090909090895, 0.015625, -0.014482990906028781, 0.0]
  daily return :
  GOOG : 0.00058182, -0.0072685 , -0.00175721, -0.02141705, -0.02233548,
        0.01655934,  0.00437406, -0.01036192, -0.00986343, -0.00812261,
       -0.00015451,  0.03260702,  0.00254415, -0.00612032,  0.0031541 ,
       -0.01018116,  0.01452125,  0.01923364,  0.01199532,  0.00939578

  USO : -0.021875  ,  0.00958466,  0.02848101, -0.04      ,  0.00320513,
        0.01277955, -0.00630915, -0.0031746 ,  0.00955414, -0.00946372,
        0.01592357,  0.02194357, -0.02147239,  0.00626959,  0.00623053,
       -0.00619195, -0.00623053, -0.00626959,  0.0126183 ,  0.01246106

  AAPL : -0.03317615,  0.01358648, -0.00292146, -0.03826267, -0.03637993,
        0.01729589, -0.00767824,  0.00018423, -0.01123596, -0.02086438,
        0.00380518,  0.0722138 , -0.00848506,  0.00142628,  0.01744704,
        0.03149606, -0.00814249, -0.0030785 ,  0.01097958, -0.00695741

  GLD : -0.02048193,  0.00369004,  0.0189951 ,  0.00060132,  0.00901442,
       -0.00059559, -0.00238379, -0.00179211,  0.        , -0.00658288,
       -0.00120482,  0.01206273, -0.00297974,  0.00119546,  0.01253731,
       -0.00117925, -0.00413223, -0.0130409 ,  0.0036036 , -0.00658288

  average daily return for each stock : [0.00086921414889110646, 0.00090320887851679248, -0.00043738608805856115, 3.7192619905374034e-05]
  std deviation for each stock : [0.01340975818314491, 0.015929201121363604, 0.024332769635802963, 0.008623865082371179]
  sharp ratio for each stock : [1.0248866711092259, 0.8965287201907356, -0.28421266379411581, 0.068190648813791305]
  """

  corr = scipy.corrcoef(all_adj_prices)
  """
  [[ GOOG        USO         AAPL        GLD]
   [ 1.          0.35796266  0.83675929 -0.38083073]
   [ 0.35796266  1.          0.49457339  0.33637871]
   [ 0.83675929  0.49457339  1.         -0.04349294]
   [-0.38083073  0.33637871 -0.04349294  1.        ]]
  """

portfolio_total_return = None
portfolio_daily_return = None

for ticker in range(4):
    if ticker == 0:
        portfolio_daily_return = all_daily_return[ticker] * percent[ticker]
        portfolio_total_return = all_total_return[ticker] * percent[ticker]
    else:
        portfolio_daily_return += all_daily_return[ticker] * percent[ticker]
        portfolio_total_return += all_total_return[ticker] * percent[ticker]

portfolio_avg_daily_return = scipy.mean(portfolio_daily_return, 0)
portfolio_std_daily_return = scipy.std(portfolio_daily_return, 0)
portfolio_sharp_ratio = scipy.sqrt(250) * (portfolio_avg_daily_return / portfolio_std_daily_return) # sharp ratio

"""
portfolio daily return :
[ -2.05026424e-02   5.79317575e-03   6.46328287e-03  -2.65390640e-02
  -1.78521880e-02   1.28560048e-02  -3.40091395e-03  -3.35140050e-03
  -5.52711836e-03  -1.31125392e-02   3.63101326e-03   4.27413546e-02
  -6.57479444e-03   2.19962885e-04   1.12093815e-02   8.88849236e-03
  -1.38771006e-03   2.83893335e-05   1.00041289e-02   1.18563923e-04]

portfolio total return : 0.000477826364861
portfolio average daily return : 0.000185268957758
portfolio std deviation daily return : 0.0143152971765
portfolio sharp ratio: 0.20463140898

"""
```
