<!--
Copyright 2022 Ocean Protocol Foundation
SPDX-License-Identifier: Apache-2.0
-->

# Quickstart: Predict Future ETH Price

This quickstart describes a flow to predict future ETH price via a local AI model using data published in Ocean.

## Setup

From [challenge 1](../challenges/main1.md), do:
- [x] Setup


### Script

In Python:

```python

# Download file from Ocean market (free)
ETH_USDT_did = "did:op:0dac5eb4965fb2b485181671adbf3a23b0133abf71d2775eda8043e8efc92d19"
file_name = ocean.assets.download_file(ETH_USDT_did, alice_wallet)

# Extracts dates and ether price values
allcex_uts, allcex_vals = load_from_ohlc_data(file_name)
print_datetime_info("CEX data info", allcex_uts)

# Transform timestamps to dates
dts = to_datetimes(allcex_uts)

# create a Data Frame with two columns [date,eth-prices] with dates given in intervals of 1-hour
import pandas as pd
data = pd.DataFrame({"ds": dts, "y": allcex_vals})

# use the last 12 hours of testing set, all the previous data is used as training
train_data = data.iloc[0:-12,:]
test_data = data.iloc[-12:,:]

# fit a linear model (Open sourced Facebook's Prophet model: https://facebook.github.io/prophet/)
# As the data is subdaily, the model will fit daily seasonality
from prophet import Prophet
model = Prophet()
model.fit(train_data)

#Predict ETH values over the range of the test set
forecast = model.predict(pd.DataFrame({"ds":test_data.ds}))
pred_vals = forecast.set_index('ds')['yhat'][-12:].to_numpy()

# Calculate Normalized Mean Squared Error between predictions and true (test) values
nmse = calc_nmse(test_data.y, pred_vals)
print(f"NMSE = {nmse}")

# Plot predicted and real values for ETH price
plot_prices(test_data.y, pred_vals)

