# CodeAlpha_Stock_Portfolio_Tracker
from alpha vantage you can use my API Key = "XRFYEYSI6E84NW2F"

import requests
import pandas as pd


class StockPortfolio:
    def __init__(self, api_key):
        self.api_key = api_key
        self.portfolio = pd.DataFrame(columns=['Ticker', 'Shares', 'Buy Price'])

    def add_stock(self, ticker, shares, buy_price):
        """Add a stock to the portfolio."""
        new_stock = pd.DataFrame({'Ticker': [ticker], 'Shares': [shares], 'Buy Price': [buy_price]})
        self.portfolio = pd.concat([self.portfolio, new_stock], ignore_index=True)
        print(f"Added {shares} shares of {ticker} at ${buy_price} each.")

    def remove_stock(self, ticker):
        """Remove a stock from the portfolio."""
        if ticker in self.portfolio['Ticker'].values:
            self.portfolio = self.portfolio[self.portfolio['Ticker'] != ticker]
            print(f"Removed {ticker} from the portfolio.")
        else:
            print(f"{ticker} not found in the portfolio.")

    def fetch_current_price(self, ticker):
        """Fetch the current price of a stock using Alpha Vantage API."""
        url = f"https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={ticker}&interval=1min&apikey={self.api_key}"
        response = requests.get(url)
        data = response.json()

        if 'Time Series (1min)' in data:
            latest_time = sorted(data['Time Series (1min)'].keys())[-1]
            return float(data['Time Series (1min)'][latest_time]['4. close'])
        else:
            print(f"Error fetching data for {ticker}: {data.get('Note', 'No data available.')}")
            return None

    def update_portfolio(self):
        """Update portfolio with current prices."""
        for index, row in self.portfolio.iterrows():
            current_price = self.fetch_current_price(row['Ticker'])
            if current_price is not None:
                self.portfolio.at[index, 'Current Price'] = current_price
                self.portfolio.at[index, 'Total Value'] = current_price * row['Shares']
                self.portfolio.at[index, 'Profit/Loss'] = (current_price - row['Buy Price']) * row['Shares']

    def display_portfolio(self):
        """Display the portfolio."""
        if self.portfolio.empty:
            print("Your portfolio is empty.")
        else:
            print(self.portfolio)


def main():
    api_key = input("Enter your Alpha Vantage API key: ")
    portfolio = StockPortfolio(api_key)

    while True:
        print("\nStock Portfolio Tracker")
        print("1. Add Stock")
        print("2. Remove Stock")
        print("3. Update Portfolio")
        print("4. Display Portfolio")
        print("5. Exit")

        choice = input("Select an option (1-5): ")

        if choice == '1':
            ticker = input("Enter stock ticker: ").upper()
            shares = int(input("Enter number of shares: "))
            buy_price = float(input("Enter buy price: "))
            portfolio.add_stock(ticker, shares, buy_price)

        elif choice == '2':
            ticker = input("Enter stock ticker to remove: ").upper()
            portfolio.remove_stock(ticker)

        elif choice == '3':
            portfolio.update_portfolio()
            print("Portfolio updated with current prices.")

        elif choice == '4':
            portfolio.display_portfolio()

        elif choice == '5':
            print("Exiting the portfolio tracker.")
            break

        else:
            print("Invalid choice. Please select a valid option.")


if __name__ == "__main__":
    main()

