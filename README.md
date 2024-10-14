# CodeAlpha_Stock_Portfolio_Tracker
from alpha vantage you can use my API Key = "XRFYEYSI6E84NW2F"
___________________________________________________________________________
import pandas as pd
import yfinance as yf


def initialize_portfolio():
    return pd.DataFrame(columns=['Ticker', 'Shares'])


def add_stock(portfolio, ticker, shares):
    new_stock = pd.DataFrame({'Ticker': [ticker], 'Shares': [shares]})
    portfolio = pd.concat([portfolio, new_stock], ignore_index=True)
    return portfolio


def remove_stock(portfolio, ticker):
    portfolio = portfolio[portfolio['Ticker'] != ticker]
    return portfolio


def fetch_stock_prices(tickers):
    data = yf.download(tickers, period='1d', group_by='ticker')
    prices = {ticker: data[ticker]['Close'][0] for ticker in tickers}
    return prices


def calculate_portfolio_value(portfolio, stock_prices):
    total_value = 0
    for index, row in portfolio.iterrows():
        ticker = row['Ticker']
        shares = row['Shares']
        price = stock_prices[ticker]
        total_value += shares * price
    return total_value


def main():
    portfolio = initialize_portfolio()

    while True:
        print("\nPortfolio Tracker")
        print("1. Add Stock")
        print("2. Remove Stock")
        print("3. View Portfolio")
        print("4. Calculate Portfolio Value")
        print("5. Exit")

        choice = input("Select an option (1-5): ")

        if choice == '1':
            ticker = input("Enter the stock ticker: ")
            shares = int(input("Enter the number of shares: "))
            portfolio = add_stock(portfolio, ticker, shares)
            print(f"Added {shares} shares of {ticker}.")

        elif choice == '2':
            ticker = input("Enter the stock ticker to remove: ")
            portfolio = remove_stock(portfolio, ticker)
            print(f"Removed {ticker} from portfolio.")

        elif choice == '3':
            print("\nCurrent Portfolio:")
            print(portfolio)

        elif choice == '4':
            if not portfolio.empty:
                stock_prices = fetch_stock_prices(portfolio['Ticker'].tolist())
                total_value = calculate_portfolio_value(portfolio, stock_prices)
                print("\nTotal Portfolio Value: $", round(total_value, 2))
                print("Current Stock Prices:")
                print(stock_prices)
            else:
                print("Portfolio is empty.")

        elif choice == '5':
            print("Exiting the Portfolio Tracker. Goodbye!")
            break

        else:
            print("Invalid option. Please try again.")


if __name__ == "__main__":
    main()
