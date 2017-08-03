
casinocoin-libjs-transactionparser
----------------------------

[![NPM](https://nodei.co/npm/casinocoin-libjs-transactionparser.png)](https://www.npmjs.org/package/casinocoin-libjs-transactionparser)

Parses transaction objects to a higher-level view.

### parseBalanceChanges(metadata)

Takes a transaction metadata object (as returned by a casinocoin-libjs response) and computes the balance changes that were caused by that transaction.

The return value is a javascript object in the following format:

```javascript
{ CASINOCOINADDRESS: [BALANCECHANGE, ...], ... }
```

where `BALANCECHANGE` is a javascript object in the following format:

```javascript
{
    counterparty: CASINOCOINADDRESS,
    currency: CURRENCYSTRING,
    value: DECIMALSTRING
}
```

The keys in this object are the Casinocoin [addresses] whose balances have changed and the values are arrays of objects that represent the balance changes. Each balance change has a counterparty, which is the opposite party on the trustline, except for CSC, where the counterparty is set to the empty string.

The `CURRENCYSTRING` is 'CSC' for CSC, a 3-letter ISO currency code, or a 160-bit hex string in the [Currency format].


### parseOrderbookChanges(metadata)

Takes a transaction metadata object and computes the changes in the order book caused by the transaction. Changes in the orderbook are analogous to changes in [`Offer` entries] in the ledger.


The return value is a javascript object in the following format:

```javascript
{ CASINOCOINADDRESS: [ORDERCHANGE, ...], ... }
```

where `ORDERCHANGE` is a javascript object with the following format:

```javascript
{
    direction: 'buy' | 'sell',
    quantity: {
        currency: CURRENCYSTRING,
        counterparty: CASINOCOINADDRESS,  (omitted if currency is 'XRP')
        value: DECIMALSTRING
    },
    totalPrice: {
        currency: CURRENCYSTRING,
        counterparty: CASINOCOINADDRESS,  (omitted if currency is 'XRP')
        value: DECIMALSTRING
    },
    makerExchangeRate: DECIMALSTRING,
    sequence: SEQUENCE,
    status: ORDER_STATUS,
    expirationTime: EXPIRATION_TIME   (omitted if there is no expiration time)
}
```


The keys in this object are the Casinocoin [addresses] whose orders have changed and the values are arrays of objects that represent the order changes.

The `SEQUENCE` is the sequence number of the transaction that created that create the orderbook change.
The `CURRENCYSTRING` is 'XRP' for XRP, a 3-letter ISO currency code, or a 160-bit hex string in the [Currency format].

The `makerExchangeRate` field provides the original value of the ratio of what the taker pays over what the taker gets (also known as the "quality").

The `ORDER_STATUS` is a string that represents the status of the order in the ledger:

*   `"created"`: The transaction created the order. The values of `quantity` and `totalPrice` represent the values of the order.
*   `"partially-filled"`: The transaction modified the order (i.e., the order was partially consumed). The values of `quantity` and `totalPrice` represent the absolute value of change in value of the order.
*   `"filled"`: The transaction consumed the order. The values of `quantity` and `totalPrice` represent the absolute value of change in value of the order.
*   `"cancelled"`: The transaction canceled the order. The values of `quantity` and `totalPrice` are the values of the order prior to cancellation.

The `EXPIRATION_TIME` is an ISO 8601 timestamp representing the time at which the order expires (if there is an expiration time).