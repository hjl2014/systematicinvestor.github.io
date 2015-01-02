---
layout: page
title: Monthly End-of-the-Month Strategy
---


To install [Systematic Investor Toolbox (SIT)](https://github.com/systematicinvestor/SIT) please visit [About](/about) page.





The [Monthly End-of-the-Month Strategy (MEOM) by Quanting Dutchman](http://quantingdutchman.wordpress.com/2010/06/30/strategy-2-monthly-end-of-the-month-meom/)
backtest and live signal.

The [MEOM Strategy](http://quantingdutchman.wordpress.com/2010/06/30/strategy-2-monthly-end-of-the-month-meom/)
invests into the top 2 ETFs that are trading above a medium term moving avarage (WMA89) 
from the universe of 26 ETFs: 


* [DIA](http://finance.yahoo.com/q/hl?s=DIA)
* [EEM](http://finance.yahoo.com/q/hl?s=EEM)
* [EFA](http://finance.yahoo.com/q/hl?s=EFA)
* [EWH](http://finance.yahoo.com/q/hl?s=EWH)
* [EWJ](http://finance.yahoo.com/q/hl?s=EWJ)
* [EWT](http://finance.yahoo.com/q/hl?s=EWT)
* [EWZ](http://finance.yahoo.com/q/hl?s=EWZ)
* [FXI](http://finance.yahoo.com/q/hl?s=FXI)
* [GLD](http://finance.yahoo.com/q/hl?s=GLD)
* [GSG](http://finance.yahoo.com/q/hl?s=GSG)
* [IEF](http://finance.yahoo.com/q/hl?s=IEF)
* [ILF](http://finance.yahoo.com/q/hl?s=ILF)
* [IWM](http://finance.yahoo.com/q/hl?s=IWM)
* [IYR](http://finance.yahoo.com/q/hl?s=IYR)
* [QQQ](http://finance.yahoo.com/q/hl?s=QQQ)
* [SPY](http://finance.yahoo.com/q/hl?s=SPY)
* [VNQ](http://finance.yahoo.com/q/hl?s=VNQ)
* [XLB](http://finance.yahoo.com/q/hl?s=XLB)
* [XLE](http://finance.yahoo.com/q/hl?s=XLE)
* [XLF](http://finance.yahoo.com/q/hl?s=XLF)
* [XLI](http://finance.yahoo.com/q/hl?s=XLI)
* [XLP](http://finance.yahoo.com/q/hl?s=XLP)
* [XLU](http://finance.yahoo.com/q/hl?s=XLU)
* [XLV](http://finance.yahoo.com/q/hl?s=XLV)
* [XLY](http://finance.yahoo.com/q/hl?s=XLY)
* [XLK](http://finance.yahoo.com/q/hl?s=XLK)

The best asset is selected using following ranking criteria:

> Rank = MA( C/Ref(C,-2), 5 ) * Ref( MA( C/Ref(C,-2), 10 ), -5 )


Following report is based on Monthly re-balancing.
Signal is generated at the month end,
and execution is done at close at the month end.

The strategy enters positions in the top 2 ranked ETFs
on the last day of the month at the close, and
exits positions two days later at the close.





Load historical data from Yahoo Finance:



{% highlight r %}
#*****************************************************************
# Load historical data
#*****************************************************************
library(SIT)
load.packages('quantmod')

tickers = 'DIA,EEM,EFA,EWH,EWJ,EWT,EWZ,FXI,GLD,GSG,IEF,ILF,IWM,IYR,QQQ,SPY,VNQ,XLB,XLE,XLF,XLI,XLP,XLU,XLV,XLY,XLK'

data <- new.env()

getSymbols.extra(tickers, src = 'yahoo', from = '1970-01-01', env = data, auto.assign = T, set.symbolnames = T, getSymbols.fn = getSymbols.fn, calendar=calendar)
  for(i in data$symbolnames) data[[i]] = adjustOHLC(data[[i]], use.Adjusted=T)
bt.prep(data, align='keep.all', dates='1995::', fill.gaps=T)

print(last(data$prices))
{% endhighlight %}



|           |    DIA|   EEM|   EFA|   EWH|   EWJ|   EWT|   EWZ|   FXI|    GLD|   GSG|    IEF|   ILF|    IWM|   IYR|    QQQ|    SPY| VNQ|   XLB|   XLE|   XLF|   XLI|   XLP|   XLU|   XLV|   XLY|   XLK|
|:----------|------:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|------:|-----:|------:|-----:|------:|-----:|------:|------:|---:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
|2014-12-31 | 177.88| 39.29| 60.84| 20.54| 11.24| 15.11| 36.57| 41.62| 113.58| 21.58| 105.99| 31.81| 119.62| 76.84| 103.25| 205.54|  81| 48.58| 79.16| 24.73| 56.58| 48.49| 47.22| 68.38| 72.15| 41.35|
    




{% highlight r %}
#*****************************************************************
# Setup
#*****************************************************************
data$universe = data$prices > 0
prices = data$prices * data$universe
	n = ncol(prices)
  nperiods = nrow(prices)
{% endhighlight %}





Code Strategy Rules:



{% highlight r %}
#*****************************************************************
# Code Strategy
#******************************************************************
# BuyRule = C > WMA(C, 89)
buy.rule = prices > bt.apply.matrix(prices, function(x) { WMA(x, 89) } )		
  buy.rule = ifna(buy.rule, F)

# 2-day returns
ret2 = ifna(prices / mlag(prices, 2), 0)

# Rank2 = MA( C/Ref(C,-2), 5 ) * Ref( MA( C/Ref(C,-2), 10 ), -5 )
position.score = bt.apply.matrix(ret2, SMA, 5) * mlag( bt.apply.matrix(ret2, SMA, 10), 5)
	position.score[!buy.rule] = NA

period.ends2 = iif(period.ends + 2 > nperiods, nperiods, period.ends + 2)


models = list()

data$weight[] = NA
	data$weight[period.ends,] = ntop(position.score[period.ends,], 2)		
	data$weight[period.ends2,] = 0		
models$strategy = bt.run.share(data, clean.signal=F, trade.summary=T, silent=T)		
{% endhighlight %}


![plot of chunk plot-7](/public/images/Strategy-MEOM/plot-7-1.png) 

#Strategy Performance:
    




|           |strategy          |
|:----------|:-----------------|
|Period     |Jan1995 - Dec2014 |
|Cagr       |7.52              |
|Sharpe     |1.07              |
|DVR        |1.02              |
|Volatility |6.98              |
|MaxDD      |-9.37             |
|AvgDD      |-2.04             |
|VaR        |0                 |
|CVaR       |-1.01             |
|Exposure   |8.85              |
    


![plot of chunk plot-7](/public/images/Strategy-MEOM/plot-7-2.png) 

#Monthly Results for strategy :
    




|     |Jan  |Feb  |Mar  |Apr  |May  |Jun  |Jul  |Aug  |Sep  |Oct  |Nov  |Dec  |Year |MaxDD |
|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:-----|
|1995 |     | 0.0 | 0.0 | 0.0 | 0.0 |-0.2 | 0.7 |-0.4 | 1.4 |-0.4 | 1.4 | 1.3 | 4.0 |-0.5  |
|1996 | 1.3 | 0.0 | 2.2 | 1.3 |-1.5 | 1.0 | 0.5 | 0.0 | 0.0 | 1.5 | 0.2 |-1.2 | 5.2 |-1.7  |
|1997 | 1.1 | 0.5 | 0.1 | 0.0 | 1.6 | 1.7 | 0.3 |-0.1 | 0.0 | 1.9 | 0.0 | 2.0 | 9.4 |-1.3  |
|1998 | 0.7 | 2.1 | 1.0 |-1.0 | 0.9 | 0.0 | 0.7 | 0.0 | 0.0 | 0.0 | 2.2 | 0.5 | 7.1 |-5.9  |
|1999 | 1.4 |-0.3 |-0.1 | 2.9 | 2.2 |-0.6 | 2.5 | 0.2 | 0.3 | 1.8 |-0.9 | 4.9 |14.9 |-1.4  |
|2000 |-4.7 | 2.3 | 0.2 | 1.2 |-1.2 | 1.6 |-0.6 | 1.9 |-2.2 |-0.4 |-0.7 |-0.2 |-2.9 |-5.8  |
|2001 |-1.2 |-3.0 | 0.0 |-0.4 | 2.7 | 0.1 |-1.0 |-0.1 | 1.5 | 1.5 | 0.0 | 2.2 | 2.3 |-5.2  |
|2002 | 1.6 |-2.0 | 4.2 | 0.9 | 1.3 |-0.1 |-2.1 | 0.0 |-1.0 | 0.0 | 2.8 |-3.1 | 2.3 |-5.0  |
|2003 | 3.9 | 0.3 | 0.1 | 3.6 |-0.2 | 1.3 | 5.1 |-0.4 | 3.1 | 2.9 | 1.0 | 3.0 |26.2 |-0.8  |
|2004 | 6.9 | 1.5 | 2.5 | 2.2 | 1.3 | 0.1 |-2.4 | 0.2 | 1.2 | 4.3 |-0.2 | 0.3 |18.9 |-2.4  |
|2005 |-5.6 | 2.1 |-0.9 | 0.1 | 0.2 | 1.5 |-0.7 | 3.4 | 2.0 |-1.5 | 1.0 | 2.0 | 3.2 |-5.8  |
|2006 | 4.1 |-3.0 | 0.4 | 1.1 | 2.0 | 1.3 |-0.9 | 0.0 | 1.0 | 0.1 | 1.1 | 1.1 | 8.5 |-4.2  |
|2007 |-0.2 | 0.6 | 0.0 | 1.3 | 1.1 | 2.6 | 3.2 |-0.6 |-0.1 | 4.8 |-4.4 |-0.7 | 7.6 |-6.0  |
|2008 | 3.7 |-1.7 |-0.2 | 4.9 | 1.9 |-2.7 | 1.9 |-0.4 | 2.7 |-1.0 | 0.0 |-1.1 | 8.1 |-4.0  |
|2009 |-2.1 |-1.1 |-2.8 | 6.6 | 4.8 | 2.5 |-2.0 | 7.1 |-4.4 |-1.0 | 1.9 | 2.3 |11.7 |-6.8  |
|2010 | 3.2 | 1.2 | 1.0 | 1.3 | 0.8 | 0.3 |-1.4 | 2.4 |-0.5 | 0.4 | 1.1 | 3.9 |14.5 |-3.9  |
|2011 | 0.7 | 1.2 | 1.0 | 0.7 |-1.2 |-2.2 | 1.8 | 1.7 |-2.0 |-1.2 | 1.4 |-0.8 | 1.0 |-5.3  |
|2012 |-1.6 | 2.2 | 0.2 | 0.3 | 0.8 | 0.1 | 1.3 |-1.0 | 0.2 | 0.8 | 1.9 | 0.1 | 5.4 |-1.6  |
|2013 | 2.6 |-0.2 | 1.4 | 1.4 |-0.4 |-0.5 | 1.4 | 1.4 | 0.2 |-0.1 | 0.4 |-1.6 | 6.1 |-2.7  |
|2014 |-1.4 |-0.9 | 1.7 | 2.5 |-0.8 | 0.4 |-1.6 | 0.9 | 0.9 | 0.0 |-0.4 |-0.5 | 0.8 |-2.6  |
|Avg  | 0.8 | 0.1 | 0.6 | 1.5 | 0.8 | 0.4 | 0.3 | 0.8 | 0.2 | 0.7 | 0.5 | 0.7 | 7.7 |-3.7  |
    


![plot of chunk plot-7](/public/images/Strategy-MEOM/plot-7-3.png) 

#Trades for strategy :
    




|symbol |weight |entry.date |exit.date  |entry.price |exit.price |return |
|:------|:------|:----------|:----------|:-----------|:----------|:------|
|IWM    | 50    |2014-02-28 |2014-03-04 |116.45      |118.74     | 0.98  |
|XLB    | 50    |2014-02-28 |2014-03-04 | 46.17      | 46.79     | 0.67  |
|EWZ    | 50    |2014-03-31 |2014-04-02 | 43.61      | 44.77     | 1.33  |
|ILF    | 50    |2014-03-31 |2014-04-02 | 35.81      | 36.65     | 1.17  |
|XLE    | 50    |2014-04-30 |2014-05-02 | 92.25      | 92.24     |-0.01  |
|XLU    | 50    |2014-04-30 |2014-05-02 | 42.09      | 41.39     |-0.83  |
|EWJ    | 50    |2014-05-30 |2014-06-03 | 11.43      | 11.55     | 0.52  |
|QQQ    | 50    |2014-05-30 |2014-06-03 | 90.51      | 90.38     |-0.07  |
|GLD    | 50    |2014-06-30 |2014-07-02 |128.04      |127.70     |-0.13  |
|XLU    | 50    |2014-06-30 |2014-07-02 | 43.48      | 42.21     |-1.46  |
|EWH    | 50    |2014-07-31 |2014-08-04 | 21.80      | 21.85     | 0.11  |
|FXI    | 50    |2014-07-31 |2014-08-04 | 39.96      | 40.56     | 0.75  |
|EWZ    | 50    |2014-08-29 |2014-09-03 | 53.04      | 53.33     | 0.27  |
|ILF    | 50    |2014-08-29 |2014-09-03 | 42.14      | 42.71     | 0.68  |
|IEF    | 50    |2014-09-30 |2014-10-02 |103.08      |103.63     | 0.27  |
|XLP    | 50    |2014-09-30 |2014-10-02 | 44.79      | 44.59     |-0.22  |
|IWM    | 50    |2014-10-31 |2014-11-04 |116.56      |115.82     |-0.32  |
|XLI    | 50    |2014-10-31 |2014-11-04 | 54.90      | 54.80     |-0.09  |
|FXI    | 50    |2014-11-28 |2014-12-02 | 40.09      | 40.00     |-0.11  |
|XLY    | 50    |2014-11-28 |2014-12-02 | 71.51      | 70.99     |-0.36  |
    




#Signals for strategy :
    




|           | DIA| EEM| EFA| EWH| EWJ| EWT| EWZ| FXI| GLD| GSG| IEF| ILF| IWM| IYR| QQQ| SPY| VNQ| XLB| XLE| XLF| XLI| XLP| XLU| XLV| XLY| XLK|
|:----------|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|2013-05-31 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|
|2013-06-28 |   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|
|2013-07-31 |   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|
|2013-08-30 |   0|   0|   0|   0|   0|   0|   0|   0|  50|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2013-09-30 |   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2013-10-31 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|
|2013-11-29 |   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2013-12-31 |   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-01-31 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|  50|   0|   0|   0|
|2014-02-28 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-03-31 |   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-04-30 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|  50|   0|   0|   0|
|2014-05-30 |   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-06-30 |   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|
|2014-07-31 |   0|   0|   0|  50|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-08-29 |   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|
|2014-09-30 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|
|2014-10-31 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|
|2014-11-28 |   0|   0|   0|   0|   0|   0|   0|  50|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|
|2014-12-31 |   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|   0|  50|   0|  50|   0|
    






For your convenience, the 
[Strategy-MEOM](/public/images/Strategy-MEOM/Strategy-MEOM.pdf)
report can also be downloaded and viewed the pdf format.


For more details please check the
[MEOM strategy code at bt.meom.test() function in bt.test.r at github](https://github.com/systematicinvestor/SIT/blob/master/R/bt.test.r)



*(this report was produced on: 2015-01-02)*