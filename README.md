# assign5
import yfinance as yf
import pandas as pd
import streamlit as st



st.title("MSFT Monthly Sharpe Ratio (5 Years)")

ticker = "MSFT"
rf_monthly = 0.0025  # 0.25% per month


@st.cache_data(ttl=3600)  # cache the download for 1 hour so the app doesn't re-download on every refresh
def get_close_prices(symbol):
    # Download monthly stock price data for the past 5 years
    data = yf.download(symbol, period="5y", interval="1mo", progress=False)
    close = data["Close"]
    # Newer yfinance versions return a table with one column per ticker; pull out the MSFT column
    if isinstance(close, pd.DataFrame):
        close = close[symbol]
    return close


close_prices = get_close_prices(ticker)

if close_prices.empty:
    st.error(f"No data found for {ticker}.")
else:
    # Calculate monthly returns
    monthly_returns = close_prices.pct_change().dropna()

    # Calculate average excess return and standard deviation of returns
    excess_returns = monthly_returns - rf_monthly
    mean_excess_return = excess_returns.mean()
    std_dev_return = monthly_returns.std()

    # Calculate the Monthly Sharpe Ratio
    monthly_sharpe_ratio = mean_excess_return / std_dev_return

    # Display the results
    st.metric(label=f"{ticker} Monthly Sharpe Ratio", value=f"{monthly_sharpe_ratio:.3f}")

    st.write(f"Ticker: {ticker}")
    st.write(f"Monthly risk-free rate: {rf_monthly:.2%}")
    st.write(f"Average monthly excess return: {mean_excess_return:.2%}")
    st.write(f"Standard deviation of monthly returns: {std_dev_return:.2%}")
    st.write(f"Number of monthly returns used: {len(monthly_returns)}")
