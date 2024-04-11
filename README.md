# Trading Platform
## Rajesh Kumar Rana

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

In trade brokerage system, brokerage application receives huge number of requests from different users on multiple channels. These requests are always come from multiple platforms such as mobile, website or over a call etc. Please refer the architecture diagram for reference:

- Type some Markdown on the left
- See HTML in the right
- ✨Magic ✨

## Components
##### Order Validation Service
This service is responsible for validating the request before entering into our trading systems. The service would be responsible for performing the below checks:
- Ensure that the user's account exists and is active.
- Check if the stock symbol provided in the request is valid and exists in the market. 
- Ensure that the stock symbol format is correct and matches the expected format
- Validate that the transaction type (buy/sell) is provided and is one of the expected values
- Check if the transaction request is made during market hours.Restrict trading activities outside market hours if required.
- Implement limits on the maximum size of the order that can be placed to prevent large orders from impacting market stability
- Provide clear and informative error messages if any validation checks fail.
- Confirm the successful execution of the transaction and provide relevant details to the user.

After successfully validating the request, generate a valid **TradeID and store it in the TradeRequest Table**. Replicate TradeRequest for OrderStatus to ensure clear separation of data insertion and retrieval, maintaining integrity and consistency in the stock market system.
