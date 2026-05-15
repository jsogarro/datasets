# Implementing Trend Indicators in kdb+

## Author Information
**James Galligan** is a kdb+ consultant who has designed and developed data-capture and data-analytics platforms for trading and analytics across multiple asset classes in multiple leading financial institutions.

## Introduction & Overview

The paper demonstrates how kdb+ and the q language enable efficient implementation of financial trend indicators through native built-in functions rather than extensive libraries. The focus is on domain-specific algorithms commonly used in finance.

"The compactness of kdb+ and the terseness of q focus code on a small number of high-performing native built-in functions rather than extensive libraries."

Cryptocurrency data for Bitcoin and Ethereum from multiple exchanges (Bitfinex, HitBtc, Kraken, Coinbase) spanning May-July 2019 illustrates the concepts. Code is available at the kxcontrib/trend-indicators repository, developed on kdb+ version 3.6.

## Data Extraction

Data was captured via Python scripts connecting to exchange feeds, published to a kdb+ tickerplant, routed to a real-time database (RDB), then written to a historical database (HDB) for analysis.

Sample data includes daily OHLCV (high/low/open/close/volume) for Bitcoin on Kraken:

```q
q)bitcoinKraken:get `:bitcoinKraken
q)\l cryptoFuncs.q
"loading in cryptoFuncs"
q)10#bitcoinKraken
date       sym     exch   high   low    open   close  vol
--------------------------------------------------------------
2019.05.09 BTC_USD KRAKEN 6174   6037.9 6042   6151.4 1808.803
2019.05.10 BTC_USD KRAKEN 6430   6110.1 6151.4 6337.9 9872.36
2019.05.11 BTC_USD KRAKEN 7450   6338   6339.5 7209.9 18569.93
2019.05.12 BTC_USD KRAKEN 7588   6724.1 7207.9 6973.9 18620.15
2019.05.13 BTC_USD KRAKEN 8169.3 6870   6970.1 7816.3 19668.6
2019.05.14 BTC_USD KRAKEN 8339.9 7620   7817.1 7993.7 18118.61
2019.05.15 BTC_USD KRAKEN 8296.9 5414.5 7988.9 8203   11599.71
2019.05.16 BTC_USD KRAKEN 8370   7650   8201.5 7880.7 13419.86
2019.05.17 BTC_USD KRAKEN 7946.2 6636   7883.6 7350   21017.35
2019.05.18 BTC_USD KRAKEN 7494.2 7205   7353.9 7266.8 6258.585
```

## Technical Analysis

Technical analysis identifies trading opportunities based on past price movements. Traders use patterns and indicators from price charts to make financial decisions. Common tools include candlestick charts, MACD, and RSI. These indicators create buy/sell signals but do not predict future prices.

### Pattern Recognition

Candlestick charts illustrate open/high/low/close of securities, enabling traders to identify patterns based on historical movements.

```q
candlestick : {
  fillscale : .gg.scale.colour.cat 01b!(.gg.colour.Red; .gg.colour.Green);

  .qp.theme[enlist[`legend_use]!enlist 0b]
  .qp.stack (
    // open/close
    .qp.interval[x; `date; `open; `close]
      .qp.s.aes[`fill; `gain]
        ,.qp.s.scale[`fill; fillscale]
        ,.qp.s.labels[`x`y!("Date";"Price")]
        ,.qp.s.geom[`gap`colour!(0; .gg.colour.White)];
      // low/high
      .qp.segment[x; `date; `high; `date; `low]
        .qp.s.aes[`fill; `gain]
        ,.qp.s.scale[`fill; fillscale]
        ,.qp.s.labels[`x`y!("Date";"Price")]
        ,.qp.s.geom[enlist [`size]!enlist 1]) }

.qp.go[700;300]
  .qp.theme[.gg.theme.clean]
  .qp.title["Candlestick chart BTC"]
  candlestick[update gain: close > open
    from select from wpData where sym=`BTC_USD,exch=`KRAKEN]
```

Each candle shows high/open/close/low and whether the security closed higher than open, useful for predicting short-term price movements.

### Simple Moving Averages

Moving averages smooth price data by creating a flowing line representing average price over time. Two commonly used types are Simple Moving Average (SMA) and Exponential Moving Average (EMA), with EMA giving larger weighting to recent prices.

"Traders analyze where the current trade price lies in relation to the moving averages. If the current trade price is above the moving-average (MA) line this would indicate over-bought (decline in price expected), trade price below MA would indicate over-sold (increase in price may be seen)."

```q
q)10#update sma2:mavg[2;close],sma5:mavg[5;close] from bitcoinKraken
date       sym     exch   high   low    open   close  vol      sma2    sma5
-------------------------------------------------------------------------------
2019.05.09 BTC_USD KRAKEN 6174   6037.9 6042   6151.4 1808.803 6151.4  6151.4
2019.05.10 BTC_USD KRAKEN 6430   6110.1 6151.4 6337.9 9872.36  6244.65 6244.65
2019.05.11 BTC_USD KRAKEN 7450   6338   6339.5 7209.9 18569.93 6773.9  6566.4
2019.05.12 BTC_USD KRAKEN 7588   6724.1 7207.9 6973.9 18620.15 7091.9  6668.275
2019.05.13 BTC_USD KRAKEN 8169.3 6870   6970.1 7816.3 19668.6  7395.1  6897.88
2019.05.14 BTC_USD KRAKEN 8339.9 7620   7817.1 7993.7 18118.61 7905    7266.34
2019.05.15 BTC_USD KRAKEN 8296.9 5414.5 7988.9 8203   11599.71 8098.35 7639.36
2019.05.16 BTC_USD KRAKEN 8370   7650   8201.5 7880.7 13419.86 8041.85 7773.52
2019.05.17 BTC_USD KRAKEN 7946.2 6636   7883.6 7350   21017.35 7615.35 7848.74
2019.05.18 BTC_USD KRAKEN 7494.2 7205   7353.9 7266.8 6258.585 7308.4  7738.84
```

Short-term traders use relatively short time periods while long-term investors compare larger periods like 100-200 days.

```q
sma:{[x]
  .qp.go[700;300]
    .qp.title["SMA BTC Kraken"]
    .qp.theme[.gg.theme.clean]
      .qp.stack(
        .qp.line[x; `date; `sma10]
          .qp.s.geom[enlist[`fill]!enlist .gg.colour.Blue]
          ,.qp.s.scale [`y; .gg.scale.limits[6000 0N] .gg.scale.linear]
          ,.qp.s.legend["";
            `sma10`sma20`close!(.gg.colour.Blue;.gg.colour.Red;.gg.colour.Green)]
          ,.qp.s.labels[`x`y!("Date";"Price")];
        .qp.line[x; `date; `sma20]
          .qp.s.geom[enlist[`fill]!enlist .gg.colour.Red]
            ,.qp.s.scale [`y; .gg.scale.limits[6000 0N] .gg.scale.linear]
            ,.qp.s.labels[`x`y!("Date";"Price")];
        .qp.line[x; `date; `close]
          .qp.s.geom[enlist[`fill]!enlist .gg.colour.Green]
          ,.qp.s.scale [`y; .gg.scale.limits[6000 0N] .gg.scale.linear]
          ,.qp.s.labels[`x`y!("Date";"Price")])}

q)sma[update sma10:mavg[10;close], sma20:mavg[20;close]
    from select from wpData where sym=`BTC_USD,exch=`KRAKEN]
```

### Moving Average Convergence Divergence

MACD is a trend indicator showing the relationship between two moving averages of a security's price, calculated by subtracting the long-term EMA (26 periods) from the short-term EMA (12 periods). The 9-day moving average of MACD (signal line) identifies buy/sell signals.

```q
macd:{[tab;id;ex]
  macd:{[x] ema[2%13;x]-ema[2%27;x]}; /macd line
  signal:{ema[2%10;x]}; /signal line
  res:select 
      sym, date, exch, close, 
      ema12:ema[2%13;close],
      ema26:ema[2%27;close],
      macd:macd[close] 
    from tab where sym=id, exch=ex;
  update signal:signal[macd] from res }

q)10#macd[bitcoinKraken;`BTC_USD;`KRAKEN]
sym     date       exch   close  ema12    ema26    macd     signal
--------------------------------------------------------------------
BTC_USD 2019.05.09 KRAKEN 6151.4 6151.4   6151.4   0        0
BTC_USD 2019.05.10 KRAKEN 6337.9 6180.092 6165.215 14.87749 2.975499
BTC_USD 2019.05.11 KRAKEN 7209.9 6338.524 6242.599 95.92536 21.56547
BTC_USD 2019.05.12 KRAKEN 6973.9 6436.274 6296.769 139.505  45.15338
BTC_USD 2019.05.13 KRAKEN 7816.3 6648.586 6409.327 239.2588 83.97447
BTC_USD 2019.05.14 KRAKEN 7993.7 6855.527 6526.688 328.8385 132.9473
BTC_USD 2019.05.15 KRAKEN 8203   7062.83  6650.859 411.9708 188.752
BTC_USD 2019.05.16 KRAKEN 7880.7 7188.656 6741.959 446.6977 240.3411
BTC_USD 2019.05.17 KRAKEN 7350   7213.478 6786.999 426.4797 277.5688
BTC_USD 2019.05.18 KRAKEN 7266.8 7221.682 6822.54  399.1421 301.8835
```

"There is a buy signal when the MACD line crosses over the signal line and there is a short signal when the MACD line crosses below the signal line."

### Relative Strength Index

RSI is a momentum oscillator measuring speed and change of price movements, oscillating between 0-100. A security is overbought when above 70 and oversold when below 30. The default period is 14 days.

Formula:

RSI = 100 - (100/(1+RS))

RS = Average Gain / Average Loss

The first calculation of average gain/loss are simple 14-day averages:
- First Average Gain: sum of Gains over the past 14 days/14
- First Average Loss: sum of Losses over the past 14 days/14

Subsequent calculations are based on prior averages and current gain/loss:

Average Gain = ((previous Average Gain) × 13 + current Gain) / 14

Average Loss = ((previous Average Loss) × 13 + current Loss) / 14

```q
relativeStrength:{[num;y]
  begin:num#0Nf;
  start:avg((num+1)#y);
  begin,start,{(y+x*(z-1))%z}\[start;(num+1)_y;num] }

rsiMain:{[close;n]
  diff:-[close;prev close];
  rs:relativeStrength[n;diff*diff>0]%relativeStrength[n;abs diff*diff<0];
  rsi:100*rs%(1+rs);
  rsi }

q)update rsi:rsiMain[close;14] by sym,exch from wpData
```

"It is shrewd to use both RSI and MACD together as both measure momentum in a market, but, because they measure different factors, they sometimes give contrary indications."

### Money Flow Index

MFI is a technical oscillator similar to RSI but uses price and volume to identify overbought/oversold conditions. MFI is known as the "volume-weighted RSI" because it weighs in on volume in addition to price.

"A low volume with a large price movement will have less impact on the relative score compared to a high volume move with a lower price move."

```q
mfiMain:{[h;l;c;n;v]
  TP:avg(h;l;c);                    / typical price
  rmf:TP*v;                         / real money flow
  diff:deltas[0n;TP];               / diffs
  /money-flow leveraging func for RSI
  mf:relativeStrength[n;rmf*diff*diff>0]%relativeStrength[n;abs rmf*diff*diff<0];
  mfi:100*mf%(1+mf);                /money flow as a percentage
  mfi }

q)update mfi:mfiMain[high;low;close;14;vol] by sym,exch from wpData
```

Sample output with 6-day period:

```q
q)10#update rsi:rsiMain[close;6],mfi:mfiMain[high;low;close;6;vol] from bitcoinKraken
date       sym     exch   high   low    open   close  vol      rsi      mfi
--------------------------------------------------------------------------------
2019.05.09 BTC_USD KRAKEN 6174   6037.9 6042   6151.4 1808.803
2019.05.10 BTC_USD KRAKEN 6430   6110.1 6151.4 6337.9 9872.36
2019.05.11 BTC_USD KRAKEN 7450   6338   6339.5 7209.9 18569.93
2019.05.12 BTC_USD KRAKEN 7588   6724.1 7207.9 6973.9 18620.15
2019.05.13 BTC_USD KRAKEN 8169.3 6870   6970.1 7816.3 19668.6
2019.05.14 BTC_USD KRAKEN 8339.9 7620   7817.1 7993.7 18118.61
2019.05.15 BTC_USD KRAKEN 8296.9 5414.5 7988.9 8203   11599.71 90.64828 81.06234
2019.05.16 BTC_USD KRAKEN 8370   7650   8201.5 7880.7 13419.86 78.60196 85.19688
2019.05.17 BTC_USD KRAKEN 7946.2 6636   7883.6 7350   21017.35 62.25494 62.04519
2019.05.18 BTC_USD KRAKEN 7494.2 7205   7353.9 7266.8 6258.585 59.91089 62.10847
```

"Analysts use both RSI and MFI together to see whether a price move has volume behind it."

### Commodity Channel Index

CCI measures the current price level relative to an average price level over time. It is used to spot new trends and identify overbought/oversold levels. Positive CCI indicates prices above historical average; negative indicates below. Moving from negative to high positive signals uptrends; reverse signals downtrends.

Formula:

CCI = (Typical Price - Moving Average) / (0.015 × Mean Deviation)

Typical Price = (high + low + close) / 3

```q
maDev:{[tp;ma;n]
  ((n-1)#0Nf),
    {[x;y;z;num] reciprocal[num]*sum abs z _y#x}'
    [(n-1)_tp-/:ma; n+l; l:til count[tp]-n-1; n] }

CCI:{[high;low;close;ndays]
  TP:avg(high;low;close);
  sma:mavg[ndays;TP];
  mad:maDev[TP;sma;n];
  reciprocal[0.015*mad]*TP-sma }

q)update cci:CCI[high;low;close;14] by sym,exch from wpData
```

### Bollinger Bands

Bollinger Bands consist of two lines plotted two standard deviations from the simple moving-average price (one positive, one negative). Standard deviation measures volatility; more volatile markets widen bands while less volatility contracts them.

"If the prices move towards the upper band the security is seen to be overbought and as the prices get close to the lower bound the security is considered oversold."

"90% of price action occurs between the bands. A breakout from this would be seen as a major event."

```q
bollB:{[tab;n;ex;id]
  tab:select from wpData where sym=id,exch=ex;
  tab:update sma:mavg[n;TP],sd:mdev[n;TP] from update TP:avg(high;low;close) from tab;
  select date,sd,TP,sma,up:sma+2*sd,down:sma-2*sd from tab}

q)bollB[wpData;20;`KRAKEN;`BTC_USD]
```

### Force Index

The Force Index measures power behind price movements using price and volume to assess force or possible turning points. It combines direction, extent, and volume in an unbounded oscillator oscillating between negative and positive values.

Calculation subtracts today's close from prior day's close and multiplies by daily volume, then calculates the 13-day EMA of this value.

```q
forceIndex:{[c;v;n]
  forceIndex1:1_deltas[0nf;c]*v;
  n#0nf,(n-1)_ema[2%1+n;forceIndex1] }

q)update ForceIndex:forceIndex[close;vol;13] by sym,exch from wpData
```

"The Force Index crosses the centre line as the price begins to increase. This would indicate that bullish trading is exerting a greater force. However, this changes towards the end of July where there is a significant change from a high positive force index to a negative one and the price drops dramatically."

### Ease of Movement Value

EMV combines momentum and volume information to decide if prices can rise or fall with little resistance in directional movement.

Formulas:

Distance Moved = ((High + Low) / 2) - ((Prior High + Prior Low) / 2)

Box Ratio = (Volume / Scale Factor) / (High - Low)

EMV = Distance Moved / Box Ratio

14-period EMV is the 14-day simple average of EMV. The scale factor is chosen to produce a normal number, generally relative to traded volume.

```q
emv:{[h;l;v;s;n]
  boxRatio:reciprocal[-[h;l]]*v%s;
  distMoved:deltas[0n;avg(h;l)];
  (n#0nf),n _mavg[n;distMoved%boxRatio] }

q)update EMV:emv[high;low;vol;1000000;14] by sym,exch from wpData
```

### Rate of Change

ROC measures the percentage change in close price over a specific period.

Formula:

ROC = ((Close - Close N Days Ago) / Close N Days Ago) × 100

```q
roc:{[c;n]
  curP:_[n;c];
  prevP:_[neg n;c];
  (n#0nf),100*reciprocal[prevP]*curP-prevP }

q)update ROC:roc[close;10] from bitcoinKraken
```

"A positive move in the ROC indicates that there was a sharp price advance. A downward drop indicates steep decline in the price. This oscillator is prone to whipsaw around the zero line."

### Stochastic Oscillator

The Stochastic Oscillator compares a particular closing price to a range of prices over a period, with 0-100 range. A security is overbought when greater than 80 and oversold when less than 20. Default period is 14 days. Sensitivity is adjusted by changing time period or taking moving average of results.

Formulas:

%K = (C - L(n)) / (H(n) - L(n))

where:
- C: Current Close
- L(n): Low across last n days
- H(n): High over last n days
- %K: slow stochastic indicator
- %D: fast stochastic indicator (n-day moving average of %K, generally n=3)

```q
stoOscCalc:{[c;h;l;n]
  lows:mmin[n;l];
  highs:mmax[n;h];
  (a#0n),(a:n-1)_100*reciprocal[highs-lows]*c-lows }

stoOscK:{[c;h;l;n;k] (a#0nf),(a:n+k-2)_mavg[k;stoOscCalc[c;h;l;n]] }

stoOscD:{[c;h;l;n;k;d] (a#0n),(a:n+k+d-3)_mavg[d;stoOscK[c;h;l;n;k]] }

q)update
    sC:stoOscCalc[close;high;low;5],
    sk:stoOscK[close;high;low;5;2],
    stoOscD[close;high;low;5;2;3]
    from bitcoinKraken
```

"Both these technical indicators are oscillators, but calculated quite differently. One of the main differences is that the Stochastic Oscillator is bound between zero and 100, while the CCI is unbounded."

### Aroon Oscillator

The Aroon Indicator identifies trend changes and trend strength. It has two parts: aroonUp and aroonDown, measuring time between highs and lows over period n (generally 25 days). Strong uptrends regularly see new highs; strong downtrends regularly see new lows. Range is 0-100.

Formulas:

aroonUp = ((n - periods since n-period high) / n) × 100

aroonDown = ((n - periods since n-period low) / n) × 100

```q
aroonFunc:{[c;n;f]
  m:reverse each a _'(n+1+a:til count[c]-n)#\:c;
  #[n;0ni],{x? y x}'[m;f] }

aroon:{[c;n;f] 100*reciprocal[n]*n-aroonFunc[c;n;f]}

aroonOsc:{[h;l;n] aroon[h;n;max] - aroon[l;n;min]}

q)update
    aroonUp:aroon[high;25;max],
    aroonDown:aroon[low;25;min],
    aroonOsc:aroonOsc[high;low;25]
    from krakenBitcoin
```

Aroon Oscillator subtracts aroonDown from aroonUp, creating range of -100 to 100.

aroonOsc = aroonUp - aroonDown

"The oscillator moves above the zero line when aroonUp moves above the aroonDown. The oscillator drops below the zero line when the aroonDown moves above the aroonUp."

## Conclusion

The paper demonstrates kdb+/q's application to produce common trade analytics efficiently using primitive functions. Functions range from moving averages to complex functions like RSI and MACD.

"This paper shows how kdb+/q can be applied to produce common trade analytics which are not available out of the box but which can be efficiently implemented using primitive functions."

"The common trend indicators discussed trigger buy/sell signals, and offer a clearer image of the current market."

"This touches the tip of the iceberg of what can be done in analytics and emphasizes the power of kdb+ in a data-analytics solution. Libraries of custom-built analytic functions can be created with ease, and in a short space of time applied to realtime and historical data."

"The combination of this library of functions and KX Analyst provides the user faster development and processing times to gain meaningful insights from the data."

## Licensing

This work is licensed under a Creative Commons Attribution 4.0 International License. Kx and kdb+ are registered trademarks of Kx Systems, Inc., a subsidiary of FD Technologies plc.
