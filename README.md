#trbot
import pandas as pd
import numpy as np
import binomo_api

# Initialize API client
api = binomo_api.Client()

# Define Bollinger Bands function
def bollinger_bands(data, window=20, std_dev=2):
    rolling_mean = data['Close'].rolling(window).mean()
    rolling_std = data['Close'].rolling(window).std()
    upper_band = rolling_mean + (rolling_std * std_dev)
    lower_band = rolling_mean - (rolling_std * std_dev)
    return upper_band, lower_band

# Define trading function
def trade(symbol, timeframe):
    # Get historical data
    data = api.get_candles(symbol, timeframe)

    # Calculate Bollinger Bands
    upper_band, lower_band = bollinger_bands(data)

    # Generate buy and sell signals
    signals = pd.DataFrame(index=data.index)
    signals['signal'] = 0
    signals['close'] = data['Close']
    signals['upper_band'] = upper_band
    signals['lower_band'] = lower_band
    signals['rolling_mean'] = data['Close'].rolling(window=20).mean()

    # Buy signal
    signals.loc[(signals['close'] < signals['lower_band']) & (signals['rolling_mean'] > signals['lower_band']), 'signal'] = 1

    # Sell signal
    signals.loc[(signals['close'] > signals['upper_band']) & (signals['rolling_mean'] < signals['upper_band']), 'signal'] = -1

    # Execute trades
    for i in range(len(signals)):
        if signals['signal'][i] == 1:
            api.place_order(symbol, 'UP', 1)
        elif signals['signal'][i] == -1:
            api.place_order(symbol, 'DOWN', 1)

# Example usage
trade('EURUSD', 'M5')
