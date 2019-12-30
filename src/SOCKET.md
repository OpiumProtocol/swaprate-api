# Swap Rate socket.io endpoint specification

## Version

v1.1.4

Use `socket.io` namespace `v1`

## Endpoint

[https://api.test.swaprate.finance/v1](https://api.test.swaprate.finance/v1)

## Description

Swap Rate socket.io endpoint allows clients to receive immediate updates on products' charts and users' orders, swaps and positions on third party projects

# Channels

Each channel serves it's own purpose. `Command` channels are used by clients to send commands to server, when `Data` channels are read-only.

## Labels description

- `PUBLIC` - publicly accessible channels
- `PROTECTED` - data channels accessible only with accessToken

## Command channel `subscribe`

Receives subscription commands with parameters and subscribes clients on data channels

### Message structure
```
{
  channel: $DATA_CHANNEL,
  param1: $VALUE_1,
  param2: $VALUE_2,
  param3: $VALUE_3,
  ...
}
```

#### Keys

- `channel` only required key, specifies data channel
- Other keys are optional and differs on different data channels, see below

## Command channel `unsubscribe`

Unsubscription command message should be **EXACTLY** the same as the one used for subscription.

All socket instances are automatically unsubscribed from all channels once disconnected.

## Data channel `error:message`


### LABELS
- `PUBLIC`

This channel is used to notify clients about errors happened during subscription and unsubscription

### Message structure

```
{
  message: $ERROR_MESSAGE,
  data: $SUBSCRIPTION_MESSAGE
}
```

#### Keys
- `message` shows error message as `String`
- `data` shows subscription message on which error has occurred as `Object`

## Data channel `products:chart`

### LABELS
- `PUBLIC`

Sends real time product chart data to client.

Product chart shows average fixed percentage on each of maturities (12 months).

### Subscription message
```
{
  channel: 'products:chart',
  id: $PRODUCT_ID
}
```

### Data message
```
{
  channel: 'products:chart',
  action: 'set',
  params: {
    id: $PRODUCT_ID
  },
  data: {
    payFixed: [{
      timestamp: number,
      value: number | null
    }],
    receiveFixed: [{
      timestamp: number,
      value: number | null
    }]
  }
}
```

#### Keys
- `action` is a constant made for further protocol upgrades, omit for now
- `params` is an object of params of subscribe message (i.e. subscribe message with excluded `channel` key)
- `data` is an object with arrays of charts data
- - `payFixed` is an array with chart data, when user selects `PAY: FIXED`
- - - `timestamp` is maturity UNIX timestamp in seconds
- - - `value` is current average fixed rate, could be `null` if there are no data, in range from `0..1`
- - `receiveFixed` is an array with chart data, when user selects `RECEIVE: FIXED`
- - - `timestamp` is maturity UNIX timestamp in seconds
- - - `value` is current average fixed rate, could be `null` if there are no data, in range from `0..1`

## Data channel `orders:address`

### LABELS
- `PROTECTED`

Sends real time users orders data to client.

Requires authentication to build subscription message.

### Subscription message
```
{
  channel: 'orders:address',
  accessToken: $ACCESS_TOKEN
}
```

### Data message
```
{
  channel: 'orders:address',
  action: 'set',
  params: {},
  data: [{
    orderId: string,
    createdAt: number,
    pay: {
      type: string, // FIXED, FLOATING
      rate: number,
      description: string,
    },
    receive: {
      type: string,
      rate: number,
      description: string,
    }
    status: string,
    nominal: number,
    token: string,
    filled: number,
    maturity: number
  }]
}
```

#### Keys
- `action` is a constant made for further protocol upgrades, omit for now
- `params` is an object of params of subscribe message (i.e. subscribe message with excluded `channel` key)
- `data` is an array of orders data
- - `orderId` is order ID
- - `createdAt` is UNIX timestamp of order creation in seconds
- - `pay` is an object which shows what user wanted to pay
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to pay
- - - `rate` is rate user wanted to pay, in range from `0..1`
- - - `description` is description for UI
- - `receive` is an object which shows what user wanted to receive
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to receive
- - - `rate` is rate user wanted to receive, in range from `0..1`
- - - `description` is description for UI
- - `status` is an ENUM [`PENDING`, `PROCESSED`, `NOT_ENOUGH_TOKENS`, `FAILED`, `EXPIRED`] of order status
- - `nominal` is nominal user wanted to cover
- - `token` is an address of token of nominal
- - `filled` is amount of nominal already covered
- - `maturity` is maturity UNIX timestamp in seconds

## Data channel `swaps:address`

### LABELS
- `PROTECTED`

Sends real time users swaps data to client.

Requires authentication to build subscription message.

### Subscription message
```
{
  channel: 'swaps:address',
  accessToken: $ACCESS_TOKEN
}
```

### Data message
```
{
  channel: 'swaps:address',
  action: 'set',
  params: {},
  data: [{
    swapId: string,
    createdAt: number,
    pay: {
      type: string, // FIXED, FLOATING
      rate: number,
      description: string,
      accInterest: number,
    },
    receive: {
      type: string,
      rate: number,
      description: string,
      accInterest: number,
    }
    nominal: number,
    token: string,
    maturity: number,
    txHash: string
  }]
}
```

#### Keys
- `action` is a constant made for further protocol upgrades, omit for now
- `params` is an object of params of subscribe message (i.e. subscribe message with excluded `channel` key)
- `data` is an array of swaps data
- - `swapId` is swap ID
- - `createdAt` is UNIX timestamp of order creation in seconds
- - `pay` is an object which shows what user wanted to pay
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to pay
- - - `rate` is rate user wanted to pay, in range from `0..1`
- - - `description` is description for UI
- - - `accInterest` is accumulated interest rate
- - `receive` is an object which shows what user wanted to receive
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to receive
- - - `rate` is rate user wanted to receive, in range from `0..1`
- - - `description` is description for UI
- - - `accInterest` is accumulated interest rate
- - `nominal` is nominal user wanted to cover
- - `token` is an address of token of nominal
- - `maturity` is maturity UNIX timestamp in seconds
- - `txHash` is transaction hash on Ethereum network (frontend leads users to etherscan)

## Data channel `positions:address` **NOT FINISHED YET**

### LABELS
- `PROTECTED`

Sends real time users third party projects positions data to client.

Requires authentication to build subscription message.

### Subscription message
```
{
  channel: 'positions:address',
  accessToken: $ACCESS_TOKEN
}
```

### Data message
```
{
  channel: 'positions:address',
  action: 'set',
  params: {},
  data: [{
    type: string, // L, D
    createdAt: number,
    pay: {
      type: string, // FIXED, FLOATING
      rate: number,
      description: string,
      accInterest: number // Exists only of pay.type == FLOATING
    } | null,,
    receive: {
      type: string,
      rate: number,
      description: string,
      accInterest: number // Exists only of receive.type == FLOATING
    } | null,
    nominal: number,
    token: string,
    populate: {
      productId: string,
      pay: {
        type: string, // FIXED, FLOATING
        rate: number
      }
      receive: {
        type: string,
        rate: number
      },
      nominal: number,
      maturity: number
    } | null
  }]
}
```

#### Keys
- `action` is a constant made for further protocol upgrades, omit for now
- `params` is an object of params of subscribe message (i.e. subscribe message with excluded `channel` key)
- `data` is an array of positions data
- - `type` is an ENUM [`L`, `D`] of position type
- - `createdAt` is UNIX timestamp of order creation in seconds
- - `pay` is an object which shows what user pays, could be `null` if there are no data
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user pays
- - - `rate` is rate user pays, in range from `0..1`
- - - `description` is description for UI
- - - `accInterest` is accumulated interest rate of Floating type
- - `receive` is an object which shows what user receives, could be `null` if there are no data
- - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user receives
- - - `rate` is rate user receives, in range from `0..1`
- - - `description` is description for UI
- - - `accInterest` is accumulated interest rate of Floating type
- - `nominal` is nominal user has in the position
- - `token` is an address of token of nominal
- - `populate` is an object with data to populate on web, if user clicks on `SWAP NOW` button, could be `null` if there are no data to populate with
- - - `pay` is an object which shows what user wanted to pay
- - - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to pay
- - - - `rate` is rate user wanted to pay, in range from `0..1`
- - - - `description` is description for UI
- - - `receive` is an object which shows what user wanted to receive
- - - - `type` is an ENUM [`FIXED`, `FLOATING`] of type user wanted to receive
- - - - `rate` is rate user wanted to receive, in range from `0..1`
- - - - `description` is description for UI
- - - `nominal` is nominal user wanted to cover
- - - `maturity` is maturity UNIX timestamp in seconds

