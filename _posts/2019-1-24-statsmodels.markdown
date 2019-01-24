---
layout: post
title:  statsmodels
date:   2019-1-24 20:35:35 +0800
categories: command
---

时间序列预测
```
from statsmodels.tsa.arima_model import ARIMA

# load series
series = read_csv('shampoo-sales.csv', header=0, parse_dates=[0], index_col=0, squeeze=True, date_parser=parser)

# fit model
model = ARIMA(series, order=(5,1,0))
model_fit = model.fit(disp=0)
print(model_fit.summary())

# forecast
output = model_fit.forecast()
yhat = output[0]
```

