# Examples

## Example 1

```python
import requests
import re
import time
import json
from bs4 import BeautifulSoup

# class-based approach - makes it more organized and modular
# separate methods - makes code more readable and maintainable
class FinvizScraper:

  BASE_URL = 'https://finviz.com/screener.ashx?v=111&f=sh_avgvol_o100'

  def __init__(self):
    self.universe_library = {}

  def get_stock_table_rows(self, soup):
    """Extract stock table rows from the soup."""
    return soup.select('div#screener-content table')[3].find_all('tr')

  def update_universe_library(self, stock_tr_data):
    """Update the universe library with stock data."""
    for index, row_data in enumerate(stock_tr_data):
      # Skip the header row
      if index == 0:
        continue

      td_data - row_data.find_all('td')[1:5]
      ticker, company, sector, industry = [d.get_text() for d in td_data]
      self.universe_library[ticker] = {
        'company': company,
        'sector': sector,
        'industry': industry
      }

  def fetch_and_parse(self, url):
    """Fetch the URL content and parse it with BeautifulSoup."""
    response = requests.get(url)
    response.raise_for_status()  # Raise an exception for HTTP errors
    return BeautifulSoup(response.content, 'html.parser')

  def scrape(self):
    """Main scraping logic."""
    soup = self.fetch_and_parse(self.BASE_URL)

    # Extract universe and page totals
    count_result = soup.select('div#screener-content td.count-text')
    universe_total = re.sub(r'Total: | #1', '', count_result[0].get_text()).strip()
    pages_total = int(re.sub(r'(.)*1\/|Page(.)*', '', count_result[1].get_text()).strip())

    self.update_universe_library(self.get_stock_table_rows(soup))

    # Loop through the remaining pages
    for i in range(1, pages_total):
      time.sleep(1.5)  # Rate limiting
      row = 20 * i + 1
      next_url = f'{self.BASE_URL}&r={row}'
      soup = self.fetch_and_parse(next_url)
      self.update_universe_library(self.get_stock_table_rows(soup))

    # Add metadata
    self.universe_library['meta'] = {
      'universe_total': universe_total,
      'data_source': 'Finviz scraping',
      'data_timestamp': int(time.time() * 1000)
    }

  # instance methods (function defined inside a class and operate on instance of class)
  # always take a reference to the instance as their first parameter (`self`)
  def save_to_file(self, filename='universe_libary.json'):
    """Save the universe library to a JSON file."""
    # "Open the file and assign the resulting file object to the variable `outfile`"
    with open(filename, 'w') as outfile:
      json.dump(self.universe_library, outfile, indent=2)

if __name__ == '__main__':
  scraper = FinvizScraper()
  scraper.scrape()
  scraper.save_to_file()
```

## `if __name__ == '__main__':`

- Common Python idiom to determine whether script should be run as standalone or imported as module into another script
- Every python script has built-in variable called `__name__`
  - `__main__`: The script is being run as the main program
  - `__name__`: The script is being imported as a module into another script

```python
# mymodule.py
def print_hello():
  print('Hello from the function!')

print('Hello from the top level!')

if __name__ == 'main'
  print('Hello from the __main__ block!')
```

Run directly:

```shell
python mymodule.py
# Hello from the top level!
# Hello from the __main__block!
```

Import into another script:

```python
# another_script.py
import mymodule

mymodule.print_hello()
```

Then run that script:

```shell
python another_script.py

# Hello from the top level!
# Hello from the function!
```
