# Quick Start Guide for the OpenBB Data API

This guide is intended to provide some starting points for querying data from the OpenBB API. Please refer to the API documentation page [here](https://hackathon.openbb.dev/docs) for more detail.

Some of the uses for the available data are:

- Training a model
- Testing a hypothesis
- Building trading strategies
- Market and macroeconomic forecasting
- Due diligence of individual companies
- Interactive dashboards and reports

## API Authorization

The API requires encoding the username and password as a Base64 string, added to the headers in every request.  Adjust the Python syntax below for your specific system.

User: openbb
Password: bcn2024finance

```python
import base64
import requests

msg = "openbb:bcn2024finance"
msg_bytes = msg.encode("ascii")
base64_bytes = base64.b64encode(msg_bytes)
base64_msg = base64_bytes.decode("ascii")

symbol="SPY"
url = f"http://hackathon.openbb.dev/api/v1/equity/price/quote?provider=intrinio&symbol={symbol}&source=intrinio_mx"
headers = {"accept": "application/json", "Authorization": f"Basic {base64_msg}"}

response = requests.get(url=url, headers=headers)

response.json()
```

## Curl Request

```console
! curl -X 'GET' \
  'https://hackathon.openbb.dev/api/v1/economy/cpi?provider=fred&countries=australia&countries=china&units=growth_same&frequency=annual&harmonized=false' \
  -H 'accept: application/json' \
  -H 'Authorization: Basic b3BlbmJiOm1pbmRzZGIyMDI0'
```

## Swagger Docs and Parameters

- The API has Swagger documentation that is available at this url: [http://hackathon.openbb.dev/docs](http://hackathon.openbb.dev/api/v1/)

- The docs page shows all available endpoints and their parameters.

- The `openapi.json` file is located [here](http://hackathon.openbb.dev/openapi.json)

- Requests made with invalid parameter names will be ignored.

- Data sources (`provider`) are entered as a choice of: ["benzinga", "intrinio", "fmp", "fred", "sec"] to the `provider` parameter of any endpoint.

- Parameters with only one choice do not need to be defined.

- Supply lists of symbols as a string - `symbols = "AAPL,MSFT,NFLX,GOOG,AMZN"` - but not every function is equipped for multi-ticker requests.

- Date parameters should be defined as: "YYYY-MM-DD"

- Functions with a `limit` or `page_size` parameter may have a default state that does not return all results.  Set this parameter as an integer.

## Example Endpoints

- Base URL: "<http://hackathon.openbb.dev/api/v1/>"

- Daily Historical Prices: "<http://hackathon.openbb.dev/api/v1/equity/price/historical?provider={provider}&symbol={symbol}&start_date={start_date}&end_date={end_date}>"
  - Use `provider=fmp` along with `interval={interval}` to get intraday data.
  - Valid intervals are: ["1m", "5m", "15m", "30m", "1h", "4h"]

- Company News:"<http://hackathon.openbb.dev/api/v1/news/company?provider=benzinga&symbols={symbols}&display=full&start_date={start_date}&end_date={end_date}&limit=1000>"

- SEC Filings by ticker: "<https://hackathon.openbb.dev/api/v1/equity/fundamental/filings?provider=sec&symbol={symbol}&limit={limit}>"
  - Add `form_type` as a parameter to query a specific filing type - i.e, "8-K", "10-K", "4", "10-Q"
  - Queries can be made by CIK number instead of the symbol - replace "symbol={symbol}" with "cik={cik}"
  - URL returned to the "complete_submission_url" field is a string representation of  every document in the filing.
  - Querying the SEC directly requires specific user agent: {"User-Agent": "Some Person <definintelyreal@person.com>"}

- Financial statements are: "balance", "cash", "income".  All have the same parameters, just swap out the statement word in the URL.
  - "<https://hackathon.openbb.dev/api/v1/equity/fundamental/balance?provider={provider}&symbol={symbol}&period={period}&limit={limit}>"
  - Returned fields are different between providers ["intrinio", "fmp"]
  - Use `period` parameter to specify "annual" or "quarter".
  - Use `limit` parameter to request more results (N results beginning from the most recent report)

- FRED Search: "<https://hackathon.openbb.dev/api/v1/economy/fred_search?provider=fred&query={query}>"
  - Refer to the Swagger docs for parameters.
  - This function is for finding the "series_id" for any particular data point published to FRED. Use the "series_id" to request the data from `fred_series` end point.
  - Only metadata is returned here.

- FRED Series: "<https://hackathon.openbb.dev/api/v1/economy/fred_series?provider=fred&symbol={series_id}>"
  - Refer to the Swagger docs for transformation parameters.
  - Units of measurement, for scaling each series, are returned under the "warnings" attribute of the response object.

- Options Chains: "<https://hackathon.openbb.dev/api/v1/derivatives/options/chains?provider=intrinio&symbol={symbol}>"
