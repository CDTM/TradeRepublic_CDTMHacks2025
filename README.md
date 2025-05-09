# <p align="center"> Trade Republic: Democratise Wealth </p>

## <p align="center"> Case Introduction: </p>

For years, we've interacted with our finances in one particular way: one balance, with a list of transactions below. Trade Republic is on a mission to democratise wealth and make finance accessible to anyone. Now, we want you to go back to the drawing board and create interfaces how users will – or should – interact with their finance in the future. Let your creativity run free – there are no boundaries and we want you to make use of any technology you deem suitable. 

## <p align="center"> The Pitch: </p>

[Trade Republic Case Pitch](TR@CDTMHacks_GitHub.pdf)


## <p align="center"> Use Case: </p>

The end user? You – and half a billion Europeans.

##  <p align="center"> Resources: </p>

For working on the case, we provide some sample data, as well as some websocket APIs.

### Data

You can find the sample data in the `data` directory of this repository.

#### Trading Transactions

Format: CSV  
Fields:
- user ID
- executed at - timestamp
- ISIN
- direction
    - BUY
    - SELL
- execution size - number of shares bought or sold 
- execution price - price per share
- currency
- execution fee
- type 
    - REGULAR
    - SAVEBACK - one of the benefits for using the TR card
    - SAVINGSPLAN - recurring investments
    - SPARECHANGE - another TR card related product. People can round up their purchases and invest the difference automatically
    - BONUS - these are bonus trades for actions like referrals or onboarding incentives


#### Banking Transactions
Format: CSV  
Fields:
- user ID
- booking date
- side
    - CREDIT - money booked onto the account
    - DEBIT - money deducted from the account
- amount
- currency
- type
    - CARD - Use of the TR Debit Card
    - CARD_ORDER - Ordering of a TR Debit Card
    - EARNINGS - Dividends, bond coupon payments, etc.
    - INTEREST
    - OTHER
    - PAYIN 
    - PAYOUT
    - TRADING
- mcc - Merchant Category Code for card useages. You can find a documentation of the MCCs from Visa [here](https://usa.visa.com/dam/VCOM/download/merchants/visa-merchant-data-standards-manual.pdf)

### APIs

How to connect? (wscat example - you can use any websocket client)
```
% wscat --connect wss://api.traderepublic.com
Connected (press CTRL+C to quit)
> connect 30
< connected
```

Every further request is a subscription in the format `sub <incrementing number> <payload>`.  
Responses will come back with the subscription number, starting with an `A` for "Answer" and then contain the JSON response.  
You can also receive updates for a subscription, as long as you do not run `unsub <subscription number>`.

#### Get Instrument Metadata
Instrument metadata are retrieved with a subscription called `instrument`. The `id` in the subscription is an ISIN.
```
sub 1 {"type": "instrument", "id": "DE000BASF111"}
< 1 A {"active":true,"exchangeIds":["LSX","TDG","TIB"],"exchanges":[{"slug":"LSX","active":true,"nameAtExchange":"BASF AG O.N.","symbolAtExchange":"BAS","band":6,"firstSeen":1547548103379,"lastSeen":1746673522056,"firstTradingDay":null,"lastTradingDay":null,"tradingTimes":null,"fractionalTrading":{"minOrderSize":"0.0000010","maxOrderSize":null,"stepSize":"0.0000010","minOrderAmount":"1","maxOrderAmount":null},"settlementRoute":"DEFAULT","weight":null},{"slug":"TDG","active":true,"nameAtExchange":"BASF SE","symbolAtExchange":"BAS","band":null,"firstSeen":1585287001108,"lastSeen":1746682222427,"firstTradingDay":null,"lastTradingDay":null,"tradingTimes":null,"fractionalTrading":null,"settlementRoute":"DEFAULT","weight":null},{"slug":"TIB","active":true,"nameAtExchange":"BASF SE","symbolAtExchange":"DE000BASF111","band":null,"firstSeen":1718096444166,"lastSeen":1746716424091,"firstTradingDay":null,"lastTradingDay":null,"tradingTimes":null,"fractionalTrading":{"minOrderSize":"0.0000010","maxOrderSize":null,"stepSize":"0.0000010","minOrderAmount":null,"maxOrderAmount":null},"settlementRoute":"DEFAULT","weight":null}],"jurisdictions":{"DE":{"active":true,"kidLink":null,"kidRequired":false,"savable":true,"fractionalTradingAllowed":true,"proprietaryTradable":true,"usesWeightsForExchanges":false,"weights":null},"AT":{"active":true,"kidLink":null,"kidRequired":false,"savable":true,"fractionalTradingAllowed":true,"proprietaryTradable":true,"usesWeightsForExchanges":false,"weights":null},[...]},"dividends":[],"splits":[],"cfi":"ESXXXX","name":"BASF AG O.N.","typeId":"stock",[...],"isin":"DE000BASF111","priceFactor":1,"shortName":"BASF","nextGenName":"BASF","alarmsName":"BASF",[...]}
```

#### Get Current Price of an Instrument
For this we have a subscription called `ticker`.  
To retrieve prices for an instrument, you will need the ISIN and the exchange. For the exchange, just use the first entry in the `exchangeIds` list of the `instrument` subscription described above. In this case that would be `LSX`. 
You then form the `id` of the `ticker` subscription by using `<ISIN>.<exchange ID>`.

```
 sub 2 {"type": "ticker", "id": "DE000BASF111.LSX"}
< 2 A {"bid":{"time":1746723164930,"price":"42.85","size":349},"ask":{"time":1746723164930,"price":"42.92","size":349},"last":{"time":1746723164930,"price":"42.85","size":349},"pre":{"time":1746651587908,"price":"42.14","size":0},"open":{"time":1746682200581,"price":"42.24","size":0},"qualityId":"realtime","leverage":null,"delta":null}
< 2 A {"bid":{"time":1746723177773,"price":"42.85","size":585},"ask":{"time":1746723177773,"price":"42.92","size":585},"last":{"time":1746723177773,"price":"42.85","size":585},"pre":{"time":1746651587908,"price":"42.14","size":0},"open":{"time":1746682200581,"price":"42.24","size":0},"qualityId":"realtime","leverage":null,"delta":null}
```

#### Retrieve Historic Prices of an Instrument
For this you use a subscription called `aggregateHistoryLight`. The `id` in the payload is formed the same way as for the `ticker` subscription above.

The following will give you the day end prices of an instrument for up to 5 years.
```
> sub 3 {"type": "aggregateHistoryLight", "id": "DE000BASF111.LSX", "range": "5y", "resolution": 86400000}
< 3 A {"expectedClosingTime":1746738000000,"aggregates":[{"time":1588896000000,"open":"46.24","high":"46.405","low":"45.64","close":"46.25","volume":0,"adjValue":"46.25"},{"time":1589155200000,"open":"46.25","high":"46.49","low":"45.04","close":"45.37","volume":0,"adjValue":"45.37"},{"time":1589241600000,"open":"45.14","high":"45.705","low":"44.73","close":"45.2","volume":0,"adjValue":"45.2"},{"time":1589328000000,"open":"45.2","high":"45.2","low":"42.645","close":"43.045","volume":0,"adjValue":"43.045"},{"time":1589414400000,"open":"42.735","high":"42.915","low":"41.14","close":"42.78","volume":0,"adjValue":"42.78"}, [...]]
```


## <p align="center"> Judging Criteria: </p>

| **Category**                     | **Description**                                                                                                                                                                                                                      |
|----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Complexity & Technical Sophistication** | How technically impressive is the solution? Were any nifty techniques, algorithms, or combinations of different components used? Are appropriate services, technologies, and infrastructure applied?                            |
| **Design**                       | Did the team put thought into the user experience? How well designed is the interface? Is the user experience intuitive? Consider the aesthetics, human-computer interaction, and overall flow of the solution.                    |
| **Viability**                    | How significant and relevant is the prototype's use case to the tackled business problem? Is it something people would actually use? Does it fulfill a real need people have?                                                      |
| **Feasibility**                  | Is the hack usable in its current state? Did the team achieve everything they wanted? Could the prototype scale as a real solution with multiple users?                                                                             |
| **Creativity & Innovation**      | How creative or innovative is the idea behind the prototype? How aware is the team of other existing solutions? How did they position their entry? Does it accomplish something new or in a novel way?                            |
| **Presentation**                | Was the team able to explain what the prototype actually did? Did the presentation and demo show how their hack fulfilled the judging criteria? Was the motivation and relevance of their work communicated effectively?          |





## <p align="center"> Point of Contact: </p>

<p align="center">  Max, Alex, Dmitry, Andrea, Bilge & Kathi will be glad to answer your questions during the Deep Dive. We’ll also be available on Discord. </p>

## <p align="center"> Prize - the winning team members will each receive: </p>

## €200 Stock Gift + free Mirror Card for each member of the winning team.

