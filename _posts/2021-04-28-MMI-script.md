---
layout: post
title: Modified Momentum Indicator - MMI
subtitle: A substitute for an ageing indicator that has arguably influenced the entire trading comunity.
categories: indicator
tags: [RSI, momentum]
---

Hi kids. Today, we'll see an indicator script, and the stpry behind. This is quite an interesting one.

## Introduction

This script is the solution to the need of an improved version of the RSI, which in itself would be good enough to take trades, and gives a better sense of confirmation for trade signals given by any on-chart indicator, adjusted for volatility.

### The RSI

For well over the better part of a quarter century, everyone in the trading community, unless one was living under a rock, was aware of RSI, and everyone learnt about RSI probably in the first few trading sessions, and how it moved.

The best part about RSI was its' versatility, and it's ability to be adaptable virtually to any chart you put it in.

The calculation methodology was also in its' favour. Simple and straight forward calculation aided young traders in understanding and using the versatile indicator.

### Beyond RSI

The RSI lacked in three distinct aspects:

- RSI wasn't fractal. A chart could have an overbought signal on one timeframe, and an extremely oversold signal on another. This isn't particularly bad, but, more often than not, this leads to more temporary drawdowns.
- RSI's calculation is based on delta, and delta being delta, doesn't take into account the trades that happened outside the range of open-close.
- As RSI doesn't account for volatility, The two - overbought and oversold zones were places form where the rsi would not generally revert, but, would stay and linger for longer periods of time, forcing Technical Analysts and followers of the religion of TA, to resort to finding divergences. Though this isn't bad, this defeats the purpose of having overbought and oversold zones.

### Goals for the System

The goal for the script was simple. Try to fix the places where RSI has lacked. After sometime of forethought, any reader would be able to tell that the way to fix the above is to do these - Fractal - have a moving average, for ranges outside open-close, and for the volatility part, it can be solved using Average True Range.

### Beinaheleidenschaftgegenstand

ATR is itself an EMA of Tr, so, it would seem that atr is what we need. But, it is far from it.

One reason for it is that the computation of ATR does a dual injustice. It takes the ema of the max of H-L, P.close - Low, and P.close - High. The issue with taking the max of these is that, when there is a gap, these equations get thrown out of proportions. When there is a gap down, after a large red candle, and a small candle closed near the previous day's close, the true range immeadiately thinks that the worst is over, and starts spiking. This makes it prone to give fakes at the end of every leg of a long term trend.

<em>The mere fact that the long term trend is in one direction, and the mini trends were against it, is all easy to identify now that we see a fully formed chart in the past.</em>

## lebenslangerschicksalsschatz

The solution is the simplest indicator script I've written in terms of the number of lines of code, and yet works remarkably well. For the sake of simplicity, i'll split into three sections.

### 1 |  Trading Range

~~~
A_range = (close[1]+low[1]-high[1])
~~~

We define current candle's trading range as the close, as reduced by low, and added by the high of the previous candle removing look-ahead bias, in case we develop any strategy using this indicator in the future. Let's call it A_range.

The most commonly asked question here is why reduce high, and add low. The answer is that unidirectional excess volatilty is detrimental to that side. For instance, when a candle makes a large high, and fails to sustain, falling, it is as good as a submission to "r/therewasanattempt". That drives a negative sentiment towards the next candle.

This trading range, even when plotted just like that, gives quite a unique output, siding with the side (up or down) with the lesser relative volatilty. But, that's not why we are here.

### 2 |  Relational Computation

~~~
unirange = iff((A_range > A_range[1]),(high+(tr*2)),abs((tr*2)-low))
~~~

Here is the brain of the computation formula. We define the variable "unirange" as high + twice the true range, or  low as reduced from twice the true range dependent on whether yesterday's "A_range" is lesser or greater than today's "A_range".

If your eyebrows aren't raised now, they better be shaved. I was the guy who wrote saying true range isn't good enough, and here I am using it. WHY?

It is true that True Range may not be the right fit for our purpose of measuring the larger range. But, it is a good computational quant, that is versatile with different markets. The 2 here, is the multiplier of the strategy. In the final script, it will be replaced with input function "mult", with a default value of 2.

### 3 |  Plot Function

~~~
p0 = plot(rsi(ema(unirange,len),len))
p1 = plot(70)
p2 = plot(30)
p3 = plot(50)
p4 = plot(100)
p5 = plot(0)
fill(p1,p4,color=color.red)
fill(p1, p2, color=color.yellow)
fill(p2, p0, color=color.green)
~~~

The plot function is again quite different, and infact goes back to take what we intended to replace, taking an rsi or the ema of the variable "var" over the same length (Defaulting to 20)

This gives us two things, one being the usability of the indicator, as anyone knowing RSI, will be able to use MMI, and the second being the smoothening of the function "unirange", with a lower kurtosis.

The resulting output is a changed RSI, that has two levels and 4 zones, all the way from overbought, to oversold.

p1, p2, p3, p4, p5 and the fill function should not require any further discussion, them being elimentary.
In addition, I went ahead and added a function of delta of the MMI, and the acceleration of MMI to get more meaningful insight.

## Indicator on TV
This looks like this - 
<!-- TradingView Chart BEGIN -->
<script type="text/javascript" src="https://s3.tradingview.com/tv.js"></script>
<script type="text/javascript">
var tradingview_embed_options = {};
tradingview_embed_options.width = '720';
tradingview_embed_options.height = '480';
tradingview_embed_options.chart = 'jNajT4Ub';
new TradingView.chart(tradingview_embed_options);
</script>
<p><a href="https://www.tradingview.com/script/jNajT4Ub-Modified-Momentum-Index-MMI/">Modified Momentum Index - MMI</a> by <a href="https://www.tradingview.com/u/BaldUncle/">BaldUncle</a> on <a href="https://www.tradingview.com/">TradingView.com</a></p>
<!-- TradingView Chart END -->

Click on the above line and add this to your fav to use on your charts.


## Complete Code - MMI

{: .box-warning}
**Note:** Tradingview link is given above. Use it to directly apply it on your charts. The below code is given for understanding alone.



~~~
// © balduncle
/////////////////////////////////////////////////////////////////////
//Version Details
//@version=4
/////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////

//input criteria//

study("Modified Momentum Index - MMI", overlay=false)
len = input(title="Length", type=input.integer,defval=20, minval=1)
//res = input("Resolution of Shading",type=input.resolution ) [res is depreciated in the current version]

//range & Calculation//
A_range = (close[1]+low[1]-high[1])
dayb4 = (close[2]+low[2]-high[2])
a = iff((A_range > A_range[1]),(high+(tr*2)),abs((tr*2)-low))
MMI = (rsi(ema(a,len),len))

//plot functions//
p0 = plot(rsi(ema(a,len),len), title="MMI")
p1 = plot(70, title = "70")
p2 = plot(30, title="30")
p3 = plot(50, title="50")
fill(p1,p2,color=color.white)

//Delta and Acceleration Plots//
deltaMMI = MMI-MMI[1]
accMMI = (deltaMMI/MMI[1])*100
plot(accMMI, title = "Acceleration of MMI", color=color.orange, style=plot.style_area)
plot(deltaMMI, title = "Change in MMI", style=plot.style_area)

/////////////////////////////////////////////////////////////////////
//Feel free to use this, just keep uncle in credit.//
/////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////
~~~

This can be paired up with a classy on-chart indicator that Uncle loves. That is called Baka Mitai. Let's call it BM.


Cheers Kids. Trade wise.

Bald Uncle