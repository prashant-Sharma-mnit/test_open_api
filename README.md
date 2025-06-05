
# 📘 5paisa Xtream Open APIs Documentation

---

## Table of Contents

### 1. User Authentication  
- [OAuth Login (Web Based)](#oauth-login-web-based)  
- [Access Token Generation](#access-token-generation)  

### 2. 🛒 Order Lifecycle  
- [PlaceOrder](#placeorder)  
- [ModifyOrder](#modifyorder)  
- [CancelOrder](#cancelorder)  

### 3. 📋 Order & Trade Tracking  
- [OrderStatus](#orderstatus)  
- [OrderBook](#orderbook)  
- [TradeBook](#tradebook)  

### 4. 📈 Positions & Holdings  
- [NetPositionNetWise](#netpositionnetwise)  
- [Holding](#holding)  

### 5. 💰 Margin  
- [Margin](#margin)  

### 6. 📊 Market Data  
- [MarketSnapshot](#marketsnapshot)  
- [MarketFeed](#marketfeed)  
- [MarketDepth](#marketdepth)
  
---


# OAuth Login (Web Based)

---

## 🔐 Part 1: OAuth Login (Web-Based)

Enables partners to redirect clients to the 5paisa login page to authenticate using client code, PIN, and OTP/TOTP as per SEBI 2FA regulations. After successful login, a **Request Token** is returned which is used to generate an **Access Token**.

### ➤ Endpoint

```
GET https://dev-openapi.5paisa.com/WebVendorLogin/VLogin/Index
```

### 📥 Parameters (Query)

| Parameter     | Required | Description                                                                 |
| ------------- | -------- | --------------------------------------------------------------------------- |
| `VendorKey`   | Yes      | API key provided by 5paisa (Partner/User App Key).                          |
| `ResponseURL` | Yes      | Callback URL to which the user will be redirected after login.              |
| `State`       | Optional | Any reference value the partner wants to pass and receive back on redirect. |

### 🔁 Flow

1. Redirect the user to the login URL with required query parameters.
2. User logs in via 5paisa Web Login page.
3. After successful authentication, the user is redirected to the `ResponseURL`.

### ✅ Example Redirect URL After Login

```bash
https://www.yourapp.com/oauth/callback?
RequestToken=eyJhbGciOi...&state=XYZ
```

### 🔑 Response Parameters (Redirected to `ResponseURL`)

| Parameter      | Description                                                      |
| -------------- | ---------------------------------------------------------------- |
| `RequestToken` | JWT token (valid for 60 minutes) to be used in Access Token API. |
| `state`        | Optional reference value returned from original request.         |

---
# Access Token Generation

---


## 🪪 Part 2: Generate Access Token (Using Request Token)

Once the **RequestToken** is obtained from OAuth login or TOTP login, use this API to generate a session-wide **AccessToken**.

### ➤ Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/GetAccessToken
```

### 🧾 Headers

| Key          | Value            |
| ------------ | ---------------- |
| Content-Type | application/json |

---

### 📤 Request Body (JSON)

```json
{
  "head": {
    "Key": "{{Partner/User App key}}"
  },
  "body": {
    "RequestToken": "{{request_token}}",
    "EncryKey": "{{Parter/User_encrykey}}",
    "UserId": "{{Parter/User_userid}}"
  }
}
```

| Field               | Mandatory | Description                                     |
| ------------------- | --------- | ----------------------------------------------- |
| `head.Key`          | Yes       | AppKey (API key provided by 5paisa).            |
| `body.RequestToken` | Yes       | Token from OAuth/TOTP login. Valid for 60 mins. |
| `body.EncryKey`     | Yes       | Encryption key shared in API credentials.       |
| `body.UserId`       | Yes       | User ID shared in API credentials.              |

---

### 🕐 Token Validity

* **Request Token:** Valid for **60 minutes**.
* **Access Token:** Valid until **11:59 PM** on the same day.

---

### 📥 Successful Response (Sample)

```json
{
  "body": {
    "AccessToken": "JWT_TOKEN",
    "ClientCode": "123456",
    "ClientName": "Mahendra",
    "POAStatus": "Y",
    "ClientType": "1",
    "CustomerType": "BASIC",
    "AllowBseCash": "Y",
    "AllowNseCash": "Y",
    ...
    "Status": 0,
    "Message": "Success"
  },
  "head": {
    "Status": 0,
    "StatusDescription": "Success"
  }
}
```

| Field                | Description                         |
| -------------------- | ----------------------------------- |
| `AccessToken`        | JWT to be used for all API calls.   |
| `ClientCode`         | 5paisa client code.                 |
| `ClientName`         | Name of logged-in user.             |
| `POAStatus`          | Power of Attorney status (`Y`/`N`). |
| `AllowBseCash`, etc. | Segment activation flags.           |
| `Status`             | `0`: Success, `2`: Invalid Inputs.  |

---

### ❌ Failure Response (Token Expired)

```json
{
  "body": {
    "AccessToken": "",
    "ClientCode": "",
    "ClientName": "",
    ...
    "Message": "Token Expired.",
    "Status": 2
  },
  "head": {
    "Status": 2,
    "StatusDescription": "Invalid Inputs"
  }
}
```

---

### 🔑 Segment Activation Flags

Flags indicating which trading segments are enabled for the user:

* `AllowBseCash`
* `AllowBseDeriv`
* `AllowBseMF`
* `AllowMCXComm`
* `AllowMcxSx`
* `AllowNSECurrency`
* `AllowNSEL`
* `AllowNseCash`
* `AllowNseComm`
* `AllowNseDeriv`
* `AllowNseMF`
* `CommodityEnabled`

---

## 📌 Summary of Authentication Flow

```text
[1] Redirect to OAuth URL (web/app)
      ↓
[2] User logs in with 2FA (client code + PIN + OTP/TOTP)
      ↓
[3] Receive RequestToken on your callback URL
      ↓
[4] Call Access Token API using RequestToken
      ↓
[5] Get AccessToken to use for other API calls
```

---


# PlaceOrder

---

## 🔍 Overview of V1/PlaceOrderRequest API

The `V1/PlaceOrderRequest` API is a RESTful endpoint designed to **place stock, derivative, currency, or commodity orders** programmatically. It supports multiple order types, and is ideal for **high-frequency and algorithmic trading systems**.

---

## 🧠 Use Case

This API is optimized for:
- **Algo trading bots**
- **Quantitative trading systems**
- **Live trading dashboards**
- **Automated risk management tools**

---

## 📬 Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/PlaceOrderRequest

````

---

## 🔐 Authentication

Authorization is **mandatory** via Bearer Token in the `Authorization` header.

| Header Key       | Value Example              |
|------------------|----------------------------|
| `Content-Type`   | `application/json`         |
| `Authorization`  | `Bearer {your_token_here}` |

---

## 📝 Request Body

### 📦 JSON Schema

```json
{
  "head": {
    "key": "YOUR_APP_KEY"
  },
  "body": {
    "Exchange": "N",
    "ExchangeType": "C",
    "ScripCode": "1660",
    "ScripData": "",
    "Price": 445,
    "StopLossPrice": 0,
    "OrderType": "Buy",
    "Qty": 1,
    "DisQty": 0,
    "IsIntraday": true,
    "iOrderValidity": 0,
    "AHPlaced": "N",
    "RemoteOrderID": "CustomOrder123",
    "ValidTillDate": "/Date(1725516056000)/",
    "AlgoID": 0,
    "DeviceID": "MyAppDevice001"
  }
}
````

---

## 📋 Field Reference

| Field            | Type    | Required | Description                                   |
| ---------------- | ------- | -------- | --------------------------------------------- |
| `head.key`       | string  | ✅        | Your registered app key                       |
| `Exchange`       | string  | ✅        | `N` = NSE, `B` = BSE, `M` = MCX               |
| `ExchangeType`   | string  | ✅        | `C` = Cash, `D` = Derivatives, `U` = Currency |
| `ScripCode`      | string  | ✅        | Unique code of the instrument (recommended)   |
| `ScripData`      | string  | ❌        | Optional (if ScripCode unavailbale )          |
| `Price`          | double  | ❌        | Order price. Set to `0` for market order      |
| `StopLossPrice`  | double  | ❌        | Trigger price for SL orders                   |
| `OrderType`      | string  | ✅        | `Buy` or `Sell`                               |
| `Qty`            | integer | ✅        | Total quantity                                |
| `DisQty`         | integer | ❌        | Disclosed quantity (≤ Qty)                    |
| `IsIntraday`     | boolean | ❌        | `true` = intraday, `false` = delivery         |
| `iOrderValidity` | integer | ❌        | 0 = Day, 5 = GTD, 3 = IOC, etc.               |
| `AHPlaced`       | string  | ❌        | After Market Order flag: `Y` or `N`           |
| `RemoteOrderID`  | string  | ✅        | user defined Custom ID for user tracking      |
| `ValidTillDate`  | string  | ❌        | For GTD/VTD orders in `/Date(Unix)` format    |
| `AlgoID`         | integer | ❌        | Strategy-specific algorithm ID                |
| `DeviceID`       | string  | ❌       | Unique device/app instance ID                 |

---

## 📘 iOrderValidity Enum

| Code | Meaning                   |
| ---- | ------------------------- |
| 0    | Day                       |
| 3    | IOC (Immediate or Cancel) |
| 5    | VTD                       |
---

### Supported Order Types

* **Limit Order**: `Price > 0`
* **Market Order**: `Price = 0`
* **Stop Loss Order**: Uses `StopLossPrice`
* **Stop Loss Market Order** : `Price = 0` & Uses `StopLossPrice`
* **After Market Order (AMO)**: `AHPlaced = "Y"`
* **Immediate or Cancel (IOC)**: `iOrderValidity = 3`

### Order Nature

* **Intraday**: `IsIntraday = true`
* **Delivery**: `IsIntraday = false`

---

## 🔢 ScripData Format Reference

| Instrument Type | Format                         | Example                       |
| --------------- | ------------------------------ | ----------------------------- |
| Cash Equity     | `RELIANCE_EQ`                  | `RELIANCE_EQ`                 |
| Futures         | `SYMBOL_YYYYMMDD`              | `NIFTY_20240930`              |
| Options         | `SYMBOL_YYYYMMDD_CE/PE_STRIKE` | `BANKNIFTY_20240329_CE_41600` |
| Currency        | `SYMBOL_EXPIRY_CE/PE_STRIKE`   | `GBPINR_1325255400_CE_107.25` |
| Commodity       | Same as above                  | `SILVER_1314489600_CE_63750`  |

---

## ✅ Sample Success Response

```json
{
  "body": {
    "BrokerOrderID": 672112769,
    "Exch": "N",
    "ExchType": "C",
    "ExchOrderID": "0",
    "LocalOrderID": 0,
    "Message": "Success",
    "RMSResponseCode": 1,
    "RemoteOrderID": "CustomOrder123",
    "ScripCode": 11915,
    "Status": 0,
    "Time": "/Date(1658255400000+0530)/"
  },
  "head": {
    "responseCode": "5PPlaceOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## ❌ Error Responses

### Invalid App Key

```json
{
  "head": {
    "responseCode": "5PPlaceOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid Head Parameters"
  }
}
```

### Invalid JWT Token

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Message": "Authentication Fails",
    "Status": 9
  }
}
```

### Invalid Session / Missing Client Code

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Message": "Invalid Session",
    "Status": 9
  }
}
```

### Invalid Input Parameters

```json
{
  "body": {
    "Message": "Invalid Input Parameters.",
    "Status": 2
  }
}
```

---

## 🔄 Order Tracking

You can track placed orders using the following APIs:

* **Order Book API**
* **Order Status API**
* **WebSocket Confirmations** (recommended for live trading)

-Use `RemoteOrderID` for consistent traceability across systems.
We have introduced the RemoteOrderId field as a user-defined identifier. Partners or clients can create their own RemoteOrderId and include it when placing an order. This identifier serves several purposes:

Track Order Status: Use the RemoteOrderId to track the order status via the Order Stand API.
Obtain ExchangeOrderId: Retrieve the ExchangeOrderId using the RemoteOrderId, which can then be used to modify the order.

Usage and Benefits

In certain scenarios, specifically with Stop-Loss (SL) orders, we have observed issues where the BrokerOrderId changes, causing clients to be unable to map the ExchangeOrderId from the broker ID. To mitigate this issue, we recommend using the RemoteOrderId.

-note:If the price is not passed in the request body, its value will be considered as 0. The 0 value of price indicates order to be of at-market type. It takes market price by default.


---

## 🧠 Best Practices

* Always log both `BrokerOrderID` and `RemoteOrderID`
* Use WebSockets for low-latency confirmation
* Validate request data to avoid rejections
* Assign unique `DeviceID` for each strategy runner
* Keep `AlgoID` dynamic per strategy or signal
* `AlgoID` is mandatory if order frequency is more than 10 orders per second 

---

## 🔗 Reference APIs

* \[🔍 Scrip Master API] — To retrieve valid ScripCode/ScripData
* \[📊 Order Status API] — To verify order status
* \[⚙️ Modify/Cancel Order API] — To update or cancel orders

---

## 🧠 Integration Tips

When integrating this API into your **algo trading system**, index the following for automation:

* `ScripData` formatting rules
* Common error response patterns
* Order status mapping
* Enum values (`OrderType`, `OrderValidity`)

This allows for **intelligent suggestions**, **error handling**, and **dynamic UI inputs**.

---

## 🛡️ Disclaimer

Placing an order through the API does **not guarantee** that it will be accepted by the exchange. Execution depends on:

* Market hours and liquidity
* Margin/funds availability
* Risk management parameters
* Exchange acceptance

---


# ModifyOrder

> ⚠️ _This (ModifyOrderV1 API) documentation is generalized for development, integration, and LLM-based use cases. It does **not** include personal credentials like API keys, tokens, or client codes._

---

## 🔁 Overview

The `ModifyOrderRequestV1` API allows clients to **modify an existing pending stock order** on the 5paisa trading platform. You can update selected fields such as:

- **Price**
- **Quantity**
- **Stop-loss price**
- **Disclosed quantity**
- **Order type** (by setting `Price = 0` for market orders)

> ✅ Only the **fields being modified** along with the **mandatory Exchange Order ID** need to be passed.

---

## 🔗 Endpoint

```http
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/ModifyOrderRequest
````

---

## 🔐 Headers

| Header        | Value Format            |
| ------------- | ----------------------- |
| Content-Type  | `application/json`      |
| Authorization | `Bearer {access_token}` |

---

## 📥 Request Format

### JSON Body

```json
{
  "head": {
    "key": "{{Your App Key}}"
  },
  "body": {
    "ExchOrderID": "1100000018012644",
    "Price": 0,
    "Qty": 10,
    "StopLossPrice": 449,
    "DisQty": 5
  }
}
```

### 🧾 Field Description

| Field                | Type   | Required | Description                              |
| -------------------- | ------ | -------- | ---------------------------------------- |
| `head.key`           | string | ✅        | Application key assigned to vendor       |
| `body.ExchOrderID`   | string | ✅        | Exchange-assigned order ID to modify     |
| `body.Price`         | float  | ❌        | New price; set `0` for market order      |
| `body.Qty`           | int    | ❌        | New total quantity                       |
| `body.StopLossPrice` | float  | ❌        | New stop loss trigger price              |
| `body.DisQty`        | int    | ❌        | New disclosed quantity (must be ≤ `Qty`) |

---

## ✅ Sample Success Response

```json
{
  "body": {
    "BrokerOrderID": 292699,
    "Exch": "B",
    "ExchOrderID": "116718512092173",
    "ExchType": "D",
    "LocalOrderID": 4,
    "Message": "Exchange is closed. Cannot Modify your order.",
    "RMSResponseCode": -15,
    "RemoteOrderID": "1716729926",
    "ScripCode": 86752,
    "Status": 1,
    "Time": "/Date(171674800000+0530)/"
  },
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## ❌ Sample Error Responses

### 🔸 Missing or Invalid Token

```
Authorization header missing or invalid
Response: Invalid Token
```

### 🔸 Invalid Head Parameters

```json
{
  "body": null,
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid head parameters."
  }
}
```

### 🔸 Missing Exchange Order ID

```json
{
  "body": {
    "BrokerOrderID": 0,
    "Exch": "N",
    "ExchOrderID": "0",
    "ExchType": "C",
    "LocalOrderID": 0,
    "Message": "Order does not exist",
    "RMSResponseCode": 0,
    "RemoteOrderID": "",
    "ScripCode": 0,
    "Status": 1,
    "Time": "/Date(1716811251790+0530)/"
  },
  "head": {
    "responseCode": "5PModifyOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## 📘 Status Codes (`head.status`)

| Code | Meaning                  |
| ---- | ------------------------ |
| 0    | Success                  |
| 1    | Invalid input parameters |
| 2    | Invalid head parameters  |
| -1   | Internal server error    |

---

## 🔤 Message Codes (`body.Message` or `RMSResponseCode`)

| Code / Text | Description                                        |
| ----------- | -------------------------------------------------- |
| `0`         | Success                                            |
| `1`         | RMS/system response                                |
| `2`         | Invalid request or parameters                      |
| `9`         | Authentication/session failed                      |
| *Text*      | Informational message (e.g., "Exchange is closed") |

---

## 💡 Developer Notes

* Use **OrderBook** or **OrderStatus** API to map `BrokerOrderID` or `RemoteOrderID` to the required `ExchOrderID`.
* **Set `Price = 0` to convert a Limit Order to a Market Order.**
* Validate that `DisQty` ≤ `Qty` before calling the API.
* Include retry logic for transient errors like network timeouts or server busy (`status = -1`).
* This API is critical for **algo-trading bots**, especially those adjusting stop-loss, target, or quantity dynamically.

---


## 🤖 Use Case: Algo Trading 

This API is designed to be easily integrated into:

* 🔁 **Automated trade modification bots**
* 📊 **Dynamic risk management systems**
* 🔄 **AI/ML model-based trade optimizers**

Use it to update open orders in real-time, respond to signals, or adjust trades in sync with your strategy logic.

---

## 📌 Summary

| Feature                 | Supported |
| ----------------------- | --------- |
| Partial Field Updates   | ✅         |
| Limit → Market Switch   | ✅         |
| Live Order Modification | ✅         |
| Algo-trade Ready        | ✅         |

---



# CancelOrder


## 🧠 API Purpose  **CancelOrderRequestV1**

`CancelOrderRequestV1` enables programmatic cancellation of orders that haven't been successfully executed yet on stock exchanges. Ideal for high-frequency and algo trading environments.

---

## 🔗 Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/CancelOrderRequest
```

---

## 📄 Headers

| Key           | Value                      |
| ------------- | -------------------------- |
| Content-Type  | application/json           |
| Authorization | bearer {Your Access Token} |

---

## 📥 Request Body (JSON)

### Top-level structure:

```json
{
  "head": {
    "key": "string"
  },
  "body": {
    "ExchangeOrderID": "string",
    "DeviceId": "string (max 100 characters)",
    "AlgoId": "integer (required for >10 orders/sec)"
  }
}
```

### Field Breakdown

| Field                  | Type    | Mandatory   | Description                                          |
| ---------------------- | ------- | ----------- | ---------------------------------------------------- |
| `head.key`             | string  | Yes         | Application key for the user or partner              |
| `body.ExchangeOrderID` | string  | Yes         | Order ID assigned by the exchange                    |
| `body.DeviceId`        | string  | Optional    | Device identifier (max length 100)                   |
| `body.AlgoId`          | integer | Conditional | Required when placing more than 10 orders per second |

---

## 📤 Response Body (JSON)

### Success Response

```json
{
  "body": {
    "BrokerOrderID": 555919893,
    "ClientCode": "string",
    "Exch": "N",
    "ExchOrderID": "0",
    "ExchType": "C",
    "LocalOrderID": 0,
    "Message": "Success",
    "RMSResponseCode": 0,
    "ScripCode": 2885,
    "Status": 0,
    "Time": "/Date(1637433000000+0530)/"
  },
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

### Failure Response Examples

#### Example 1 – Invalid `head.key`

```json
{
  "body": null,
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "2",
    "statusDescription": "Invalid Head Parameters"
  }
}
```

#### Example 2 – Missing Mandatory Fields

```json
{
  "body": {
    "BrokerOrderID": 0,
    "ClientCode": "string",
    "Exch": "?",
    "ExchOrderID": "0",
    "ExchType": "?",
    "LocalOrderID": 0,
    "Message": "Invalid Input Parameters.",
    "RMSResponseCode": 0,
    "ScripCode": 0,
    "Status": 2,
    "Time": "/Date(1637494757568+0530)/"
  },
  "head": {
    "responseCode": "5PCancelOrdReqV1",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## 🔍 Status Codes

### `head.status`

| Code | Meaning                              |
| ---- | ------------------------------------ |
| -1   | Server unable to process the request |
| 0    | Success                              |
| 1    | Invalid input parameters             |
| 2    | Invalid head parameters              |

### `body.Status` & `body.Message`

| Code | Message                      |
| ---- | ---------------------------- |
| 0    | Success                      |
| 1    | 5paisa System (RMS) Response |
| 2    | Invalid Input Parameters     |
| 9    | Authentication Fails         |

---

## 💡 Notes

* ✅ `ExchangeOrderID` is **mandatory** for cancelling orders.
* ⚙️ `AlgoId` becomes **mandatory** when placing orders at a high frequency (>10 per second).
* 🔐 Ensure correct bearer token and app key are passed in the request header for authentication.
* 🧠 Responses may contain RMS-level data, useful for risk-based strategies or real-time order validation.

---

## 📌 Quick Summary

| Aspect         | Details                                                        |
| -------------- | -------------------------------------------------------------- |
| API Name       | `CancelOrderRequestV1`                                         |
| Type           | RESTful JSON POST API                                          |
| Use Case       | Cancel a live or pending order before execution                |
| Authentication | Bearer token + Application key                                 |
| Ideal For      | Algo trading assistants, RAG workflows, high-frequency systems |
| Extras         | `DeviceId` & `AlgoId` parameters for extended control          |

---

## 📚 For Developers

* Add retry mechanisms for transient failures.
* Always log `ExchangeOrderID`, `Exch`, `ScripCode`, and timestamps for audit purposes.
* Use `Time` field in response to record actual cancellation time in logs.

---


# MultiOrderMargin

## 🛒 Multi Order Margin API

**Description:** Retrieve margin details for multiple orders.

### Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/MultiOrderMargin
```

### Headers

| Key           | Value                  |
| ------------- | ---------------------- |
| Content-Type  | application/json       |
| Authorization | Bearer {{AccessToken}} |

### Request Body

```json
{
  // Request parameters here
}
```

### Response

```json
{
  // Response details here
}
```

---



  


# OrderStatus

## 🔹 API Name
`v2/OrderStatus`

---

## 🎯 Purpose

The `OrderStatusV2` API allows you to **fetch real-time status of one or more orders** placed via the 5paisa trading system. Ideal for algorithmic trading, dashboards, and bot monitoring workflows.

---

## 📌 Endpoint

```
https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V2/OrderStatus
```


---

## 📥 Request Structure

### ➕ Method
`POST`

### ➕ Headers

| Key             | Value                        |
|----------------|------------------------------|
| Content-Type   | application/json             |
| Authorization  | Bearer `<YourAccessToken>`   |

---

### 🧾 Sample Request Body

```json
{
  "head": {
    "key": "<YourAppKey>"
  },
  "body": {
    "ClientCode": "<ClientCode>",
    "OrdStatusReqList": [
      {
        "Exch": "N",
        "RemoteOrderID": "0327020205139304480"
      },
      {
        "Exch": "N",
        "RemoteOrderID": "203051105331"
      }
    ]
  }
}
````

---

## 📤 Success Response Example

```json
{
  "body": {
    "Message": "Success",
    "OrdStatusResLst": [
      {
        "AveragePrice": 431.05,
        "Exch": "N",
        "ExchOrderID": 10000365323,
        "ExchOrderTime": "/Date(1715587966000+0530)/",
        "ExchType": "C",
        "OrderQty": 1,
        "OrderRate": 431.05,
        "PendingQty": 1,
        "ScripCode": 1660,
        "Status": "Modified",
        "Symbol": "ITC",
        "TradedQty": 0
      },
      {
        "AveragePrice": 431.05,
        "Exch": "N",
        "ExchOrderID": 100000365383,
        "ExchOrderTime": "/Date(1715587966000+0530)/",
        "ExchType": "C",
        "OrderQty": 1,
        "OrderRate": 431.05,
        "PendingQty": 0,
        "ScripCode": 1660,
        "Status": "Fully Executed",
        "Symbol": "ITC",
        "TradedQty": 1
      }
    ],
    "Status": 0
  },
  "head": {
    "responseCode": "5POrdStatusV2",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## ❌ Failure Response Example

```json
{
  "body": {
    "Message": "Success",
    "OrdStatusResLst": [],
    "Status": 0
  },
  "head": {
    "responseCode": "5POrdStatusV2",
    "status": "0",
    "statusDescription": "Success"
  }
}
```

---

## 📘 Field Definitions

### 🔸 Request Parameters

| Field           | Type   | Required | Description                               |
| --------------- | ------ | -------- | ----------------------------------------- |
| `Exch`          | string | Yes      | Exchange code (e.g. `"N"` for NSE)        |
| `RemoteOrderID` | string | Yes      | Order ID generated at the time of placing |
| `ClientCode`    | string | Yes      | User's client code                        |

---

### 🔸 Response Fields

| Field           | Type     | Description                                   |
| --------------- | -------- | --------------------------------------------- |
| `AveragePrice`  | float    | Average traded price                          |
| `Exch`          | string   | Exchange code                                 |
| `ExchOrderID`   | string   | Order ID from exchange                        |
| `ExchOrderTime` | datetime | Order entry timestamp                         |
| `ExchType`      | string   | Exchange type (e.g., C = Cash, D = Deriv.)    |
| `OrderQty`      | int      | Total quantity                                |
| `OrderRate`     | float    | Price at which order was placed               |
| `PendingQty`    | int      | Quantity yet to be traded                     |
| `ScripCode`     | int      | Unique instrument code                        |
| `Status`        | string   | Current status (e.g., "Modified", "Executed") |
| `Symbol`        | string   | Trading symbol (e.g., "ITC")                  |
| `TradedQty`     | int      | Quantity already traded                       |

---

## 📑 Order Status Values

| Status             | Meaning                             |
| ------------------ | ----------------------------------- |
| `Fully Executed`   | Order completed                     |
| `Modified`         | Order was modified                  |
| `Xmitted`          | Not reached or rejected by exchange |
| `Rejected By 5P`   | Rejected by 5paisa system           |
| `Rejected by Exch` | Rejected by exchange                |
| `Cancelled`        | Order was cancelled                 |
| `Pending`          | Order placed and awaiting execution |

---

## 🔁 Use Cases

* Checking real-time order status in trading dashboards
* Monitoring order lifecycle in algorithmic bots
* Integrating into RAG-based AI assistants for trading automation

---

## ⚙️ Best Practices

* Batch multiple order queries (up to \~50)
* Validate session token before calling
* Use proper exchange codes (`N`, `B`, `M`, `X`)

---

## 🔐 Authentication

* Requires a valid bearer token in the `Authorization` header
* App Key in the `head.key` field is mandatory
* Session tokens must be refreshed periodically

---

## 📎 Additional Notes

* Use this API instead of full Order Book API for specific order lookup
* Exchange Order ID from this API can be used for modifying/cancelling orders
* All timestamps returned are Unix/Epoch-style (can be converted)

---

## 🧪 Postman / cURL Example

```bash
curl --location 'https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V2/OrderStatus' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <YourAccessToken>' \
--data '{
  "head": {
    "key": "<YourAppKey>"
  },
  "body": {
    "ClientCode": "<ClientCode>",
    "OrdStatusReqList": [
      {
        "Exch": "N",
        "RemoteOrderID": "0327020205139304480"
      }
    ]
  }
}'
```

---

# OrderBook

## Overview of Order Book V4 API
The **Order Book V4 API** enables partners and clients to retrieve the order book details of a user for the current trading day. It supports orders across multiple segments including cash, derivatives, currency, and commodities.

This API is essential for tracking the status of placed orders by providing detailed order information such as average price, trigger rates, traded and pending quantities. It also provides key identifiers like remote order ID, broker order ID, and exchange order ID for mapping and managing orders efficiently.

> **Note:** For real-time order status updates, consider using WebSocket connections or the dedicated Order Status API.

---

## API Endpoint
```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V4/OrderBook
```
---

## Request Parameters

| Parameter            | Type    | Required | Description                                                                                  |
|----------------------|---------|----------|----------------------------------------------------------------------------------------------|
| `ClientCode`         | String  | Yes      | Unique identifier for the client whose order book is being requested.                        |
| `updatedInLastSeconds` | Integer | No       | Filters orders updated in the last *x* seconds (valid range: 0 to 600).                      |

### Parameter Details

- **ClientCode**  
  Identifier for the client. Must be provided to fetch the relevant order book.

- **updatedInLastSeconds** (Optional)  
  - If provided and between 1 and 600, only orders updated within the last *x* seconds are returned.  
  - If `0` or not provided, the full order book is returned without filtering.  
  - If no orders are updated in the specified time frame, response will indicate:  
    `"No Order found for this Client."`  
  - If value is out of range or negative, an error is returned:  
    `"updatedInLastSeconds should be in range from 0 to 600"`.

---

## Request Example (JSON)

```json
{
  "head": {
    // Standard request header fields
  },
  "body": {
    "ClientCode": "YOUR_CLIENT_CODE",
    "updatedInLastSeconds": 60
  }
}
````

---

## Response Structure

| Field       | Type    | Description                                                      |
| ----------- | ------- | ---------------------------------------------------------------- |
| `status`    | Integer | Execution status code (0 = Success, other codes indicate errors) |
| `message`   | String  | Descriptive message related to the API call                      |
| `orderBook` | Array   | List of order details objects                                    |

### Order Details Object

| Field                | Type    | Description                        |
| -------------------- | ------- | ---------------------------------- |
| `Exch`               | String  | Exchange code                      |
| `ExchType`           | String  | Exchange segment/type              |
| `ExchOrderID`        | String  | Exchange order ID                  |
| `BrokerOrderId`      | String  | Broker's order ID                  |
| `BrokerOrderTime`    | String  | Timestamp of broker order          |
| `BuySell`            | String  | Buy or sell indicator              |
| `AveragePrice`       | Decimal | Average traded price for the order |
| `TriggerRate`        | Decimal | Trigger price (if applicable)      |
| `TradedQty`          | Integer | Quantity traded                    |
| `PendingQty`         | Integer | Quantity pending                   |
| `OrderStatus`        | String  | Current status of the order        |
| `AtMarket`           | Boolean | Indicates if order is at market    |
| `AfterHours`         | String  | After hours flag                   |
| `AHProcess`          | String  | After hours process status         |
| `DisClosedQty`       | Integer | Disclosed quantity                 |
| `OrderRequesterCode` | String  | Code of the requester of the order |
| `OldOrderQty`        | Integer | Previous order quantity            |
| `DelvIntra`          | String  | Order for delivery or intraday     |

> The list above is a representative sample of key fields returned. Additional fields may be present depending on implementation.

---

## Response Example (JSON)

```json
{
  "head": {
    "responseCode": "5POrdBkV4",
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "Success",
    "orderBook": [
      {
        "Exch": "NSE",
        "ExchType": "C",
        "ExchOrderID": "123456789",
        "BrokerOrderId": "987654321",
        "BrokerOrderTime": "2025-05-19 10:30:15.000",
        "BuySell": "B",
        "AveragePrice": 1500.25,
        "TriggerRate": 0,
        "TradedQty": 100,
        "PendingQty": 50,
        "OrderStatus": "Open",
        "AtMarket": false,
        "AfterHours": "N",
        "AHProcess": "N",
        "DisClosedQty": 0,
        "OrderRequesterCode": "ORD123",
        "OldOrderQty": 150,
        "DelvIntra": "Delivery"
      }
    ]
  }
}
```

---

## Error Handling

| Status Code | Description                                        |
| ----------- | -------------------------------------------------- |
| 1           | Invalid value for `updatedInLastSeconds` parameter |
| 2           | Missing or invalid request parameters              |
| 3           | Invalid `ClientCode`                               |
| 9           | Session invalid or unauthorized access             |

---

## Notes

* The API requires valid authentication tokens in request headers.
* This API is ideal for fetching a snapshot of the order book.
* For real-time order status updates, use the WebSocket API or dedicated order status endpoints.
* The order mapping via `ExchOrderID`, `BrokerOrderId`, and `OrderRequesterCode` allows seamless order management like modifications or cancellations.

---

## Best Practices for Integration

* Use `updatedInLastSeconds` parameter to optimize API calls for recent order changes, reducing data transfer.
* Always verify response `status` and `message` fields before processing data.
* Keep client authentication credentials secure and refresh tokens as needed.
* Combine this API data with real-time WebSocket feeds for comprehensive order tracking.

---



# TradeBook

## Overview of TradeBookV1 API

The `TradeBookV1` API allows clients to fetch their trade book for the current trading day. It returns comprehensive trade details across **Cash**, **F&O**, **Currency**, and **Commodity** segments from **all supported exchanges** (e.g., NSE, BSE, MCX).

## 📌 Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/TradeBook

````

---

## 📄 Purpose

The API is designed for:

- **Monitoring** executed trades.
- **Trade analysis** based on quantity, rate, and exchange.
- **Mapping** trades to respective orders using `ExchangeOrderID` and `ExchangeTradeID`.

A single order may be split into multiple trades, and this mapping enables precise trade-order relationship tracking.

---

## ✅ Features

- Returns all executed trades for the current day.
- Includes buy/sell type, exchange details, scrip info, and rates.
- Compatible across multiple asset classes and exchanges.
- Designed for integration into **trading assistants**, **portfolio tools**, or **RAG systems**.

---

## 🧾 Request Format (JSON)

```json
{
  "head": {
    "key": "<your-app-key>"
  },
  "body": {
    "ClientCode": "<your-client-code>"
  }
}
````

---

## 📤 Response Format (Success)

```json
{
  "head": {
    "responseCode": "5PTrdBkV1",
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "",
    "TradeBookDetail": [
      {
        "Exch": "N",
        "ExchType": "C",
        "ScripCode": 500112,
        "ScripName": "SBIN",
        "BuySell": "B",
        "Qty": 100,
        "PendingQty": 0,
        "OrgQty": 100,
        "Rate": 540.25,
        "ExchOrderID": "ABC12345678",
        "ExchangeTradeID": "TRD987654",
        "ExchangeTradeTime": "2025-05-19T12:34:56Z",
        "DelvIntra": "D",
        "TradeType": "Online",
        "Multiplier": 1
      }
    ]
  }
}
```

---

## ❗ Error Response Example

```json
{
  "head": {
    "responseCode": "5PTrdBkV1",
    "status": 9,
    "statusDescription": "Invalid session"
  },
  "body": {
    "Status": 9,
    "Message": "Invalid session"
  }
}
```

---

## 📚 Response Field Description

| Field               | Type     | Description                              |
| ------------------- | -------- | ---------------------------------------- |
| `Exch`              | `char`   | Exchange code (e.g., `N` for NSE)        |
| `ExchType`          | `char`   | Segment type (e.g., `C` for Cash)        |
| `ScripCode`         | `int`    | Unique code of the instrument            |
| `ScripName`         | `string` | Instrument name                          |
| `BuySell`           | `char`   | `B` for Buy, `S` for Sell                |
| `Qty`               | `int`    | Traded quantity                          |
| `PendingQty`        | `int`    | Remaining quantity (if any)              |
| `OrgQty`            | `int`    | Original quantity of the order           |
| `Rate`              | `double` | Trade execution price                    |
| `ExchOrderID`       | `string` | Order reference ID from the exchange     |
| `ExchangeTradeID`   | `string` | Trade reference ID from the exchange     |
| `ExchangeTradeTime` | `string` | ISO timestamp of the trade               |
| `DelvIntra`         | `char`   | Delivery (`D`) or Intraday (`I`) flag    |
| `TradeType`         | `string` | Trade origin (`Online`, `Offline`, etc.) |
| `Multiplier`        | `int`    | Lot multiplier, useful in derivatives    |

---

## 🔐 Authorization

This API requires **JWT token-based authentication**. Ensure the token is passed in the `Authorization` header as:

```
Authorization: Bearer <your-jwt-token>
```

---

## 💡 Best Practices

* 🔍 **Map trades to orders** using `ExchOrderID` and `ExchangeTradeID`.

---
# NetPositionNetWise


## Overview of NetPosition_NetWiseV3 API
The **NetPosition_NetWiseV3** API provides detailed net position data for a client across multiple exchanges and product types. It aggregates positions in equity and commodity segments, delivering net quantities, average rates, and related metrics in a structured response.

---

## API Endpoint
```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V2/NetPositionNetWise
```
---

## Request Structure

```json
{
  "head": {
    "key": "string",
    "requestCode": "string",
    "appName": "string"
  },
  "body": {
    "ClientCode": "string"
  }
}
````

### Request Parameters

| Field              | Type   | Description                         |
| ------------------ | ------ | ----------------------------------- |
| `head.key`         | string | Authentication/authorization key    |
| `head.requestCode` | string | Identifier for the API request      |
| `head.appName`     | string | Application name making the request |
| `body.ClientCode`  | string | Unique client identifier            |

---

## Response Structure

```json
{
  "head": {
    "responseCode": "string",
    "status": int,
    "statusDescription": "string"
  },
  "body": {
    "Status": int,
    "Message": "string",
    "NetPositions": [
      {
        "Exch": "string",
        "ExchType": "string",
        "ScripCode": int,
        "ScripName": "string",
        "BuyQty": int,
        "SellQty": int,
        "NetQty": int,
        "AvgRate": double,
        "LastRate": double,
        "Multiplier": double,
        "Product": "string",
        "AdditionalFields": "..."
      }
    ]
  }
}
```

### Response Fields Description

| Field                    | Type   | Description                                               |
| ------------------------ | ------ | --------------------------------------------------------- |
| `head.responseCode`      | string | Code representing the API response                        |
| `head.status`            | int    | Status code (0 for success, other values indicate errors) |
| `head.statusDescription` | string | Text description of the status                            |
| `body.Status`            | int    | Business status code                                      |
| `body.Message`           | string | Response message or error description                     |
| `body.NetPositions`      | array  | List of net position details for various scrips           |

Each element in `NetPositions` includes:

| Field              | Type   | Description                                             |
| ------------------ | ------ | ------------------------------------------------------- |
| `Exch`             | string | Exchange code (e.g., N for NSE, M for MCX)              |
| `ExchType`         | string | Exchange segment/type                                   |
| `ScripCode`        | int    | Unique code identifying the trading instrument          |
| `ScripName`        | string | Name of the trading instrument                          |
| `BuyQty`           | int    | Total quantity bought                                   |
| `SellQty`          | int    | Total quantity sold                                     |
| `NetQty`           | int    | Net quantity (BuyQty - SellQty plus adjustments)        |
| `AvgRate`          | double | Average rate of the position                            |
| `LastRate`         | double | Last traded price                                       |
| `Multiplier`       | double | Multiplier used for value calculation                   |
| `Product`          | string | Product type indicator (e.g., Delivery, Intraday)       |
| `AdditionalFields` | varies | Other position-related metrics (e.g., BodQty, BuyValue) |

---

## Status Codes

| Code | Meaning                         |
| ---- | ------------------------------- |
| 0    | Success                         |
| 1    | No records found                |
| 2    | Validation error                |
| 9    | Invalid session or unauthorized |

---

## Example Request

```json
{
  "head": {
    "key": "your_auth_key",
    "requestCode": "5PNPNWV3",
    "appName": "YourAppName"
  },
  "body": {
    "ClientCode": "CLIENT1234"
  }
}
```

---

## Example Successful Response

```json
{
  "head": {
    "responseCode": "5PNPNWV3",
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "Success",
    "NetPositions": [
      {
        "Exch": "N",
        "ExchType": "Y",
        "ScripCode": 12345,
        "ScripName": "ABC Ltd",
        "BuyQty": 100,
        "SellQty": 50,
        "NetQty": 50,
        "AvgRate": 150.25,
        "LastRate": 152.00,
        "Multiplier": 1,
        "Product": "D"
      }
    ]
  }
}
```

---

## Notes

* Authorization is required; requests without valid authorization will be rejected.
* The API aggregates net positions by product type and exchange.
* If no data is found for the client, the response will indicate "No record found."
* Input validation is performed on request headers and body fields.
* The API supports multiple product types including Delivery (D), Intraday (I), and others.

---



## Best Practices

* Use secure storage and transmission for authorization tokens.
* Always check response status before processing data.
* Handle empty or partial data scenarios gracefully.

---
# Holding

## Overview HoldingV4 API

The **HoldingV4** API retrieves detailed holdings data for a client’s stock portfolio. It provides essential information such as instrument codes, quantity held, current price, pledge details, and other metadata related to each holding. This API requires authenticated access and is typically used to build the holdings section in trading or portfolio management applications.

---

## Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V4/Holding
```

---

## Request Headers

| Header        | Description                     | Example                       |
| ------------- | ------------------------------- | ----------------------------- |
| Content-Type  | Must be `application/json`      | `application/json`            |
| Authorization | Bearer token for authentication | `Bearer <JWT-token>`          |
| Cookie        | Optional session cookie         | `5paisacookie=<cookie_value>` |

---

## Request Body

```json
{
  "head": {
    "key": "<api-key>"
  },
  "body": {
    "ClientCode": "<client-code>"
  }
}
```

### Request Body Properties

| Property        | Type   | Description                | Required |
| --------------- | ------ | -------------------------- | -------- |
| head.key        | string | API key for access control | Yes      |
| body.ClientCode | string | Unique client identifier   | Yes      |

---

## Response

### Success Response (HTTP 200)

```json
{
  "head": {
    "responseCode": "5PHoldingV4",
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "Success",
    "CacheTime": 300,
    "Data": [
      {
        "Exch": "B",
        "ExchType": "C",
        "NseCode": 14366,
        "BseCode": 532822,
        "Symbol": "IDEA",
        "FullName": "VODAFONE IDEA LIMITED",
        "Quantity": 52,
        "CurrentPrice": 7.09,
        "PoolQty": 0,
        "DPQty": 44,
        "POASigned": "N",
        "ScripMultiplier": 1,
        "AvgRate": 7.2338,
        "ISIN": "INE669E01016",
        "MTFPledge": 0,
        "MTFQty": 0,
        "MarginPledge": 8,
        "PledgeQty": 8
      }
    ]
  }
}
```

### Response Properties

| Property               | Type   | Description                                     |
| ---------------------- | ------ | ----------------------------------------------- |
| head.responseCode      | string | API response identifier                         |
| head.status            | string | Status code (0 = success)                       |
| head.statusDescription | string | Status message                                  |
| body.Status            | int    | API call status (0 = success, non-zero = error) |
| body.Message           | string | Informational message                           |
| body.CacheTime         | int    | Cache validity duration (seconds)               |
| body.Data              | array  | List of holdings records                        |

### Holding Record Fields

| Property        | Type   | Description                                    |
| --------------- | ------ | ---------------------------------------------- |
| Exch            | char   | Exchange code (e.g., 'B' for BSE)              |
| ExchType        | char   | Exchange type (e.g., 'C' for Cash)             |
| NseCode         | int    | NSE instrument code                            |
| BseCode         | int    | BSE instrument code                            |
| Symbol          | string | Stock symbol                                   |
| FullName        | string | Full name of the instrument                    |
| Quantity        | long   | Quantity held                                  |
| CurrentPrice    | double | Current market price                           |
| PoolQty         | int    | Quantity in pool (if any)                      |
| DPQty           | int    | Demat Participant quantity                     |
| POASigned       | char   | POA (Power of Attorney) flag                   |
| ScripMultiplier | int    | Multiplier for the scrip                       |
| AvgRate         | double | Average rate (price) of the holding            |
| ISIN            | string | International Securities Identification Number |
| MTFPledge       | int    | Margin trading pledge quantity                 |
| MTFQty          | int    | Margin trading finance quantity                |
| MarginPledge    | int    | Quantity pledged as margin                     |
| PledgeQty       | int    | Quantity pledged                               |

---

## Error Responses

### Invalid Head Parameters

```json
{
  "head": {
    "responseCode": "5PHoldingV4",
    "status": "2",
    "statusDescription": "Invalid head parameters."
  },
  "body": null
}
```

### No Records Found

```json
{
  "head": {
    "responseCode": "5PHoldingV4",
    "status": "0",
    "statusDescription": "Success"
  },
  "body": {
    "Status": 1,
    "Message": "No record found.",
    "CacheTime": 300,
    "Data": []
  }
}
```

### Invalid Session / Authentication Failure

```json
{
  "head": {
    "responseCode": "5PHoldingV4",
    "status": "9",
    "statusDescription": "Invalid session."
  },
  "body": {
    "Status": 9,
    "Message": "Invalid session."
  }
}
```

---

## cURL Example

```bash
curl --location 'https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V4/Holding' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <your-jwt-token>' \
--header 'Cookie: 5paisacookie=<your-cookie>' \
--data '{
    "head": {
        "key": "<api-key>"
    },
    "body": {
        "ClientCode": "<client-code>"
    }
}'
```

---

## Notes

* **Authentication**: Requires valid Bearer token or session cookie.
* **ClientCode** should match the authenticated user’s client code.
* **CacheTime** indicates how long the response can be cached to optimize subsequent calls.
* This API is designed for retrieving real-time portfolio holdings and related analytics.

---

# Margin

## Overview of MarginV4 API

The `MarginV4` API provides detailed margin information for a specified client code. It retrieves equity and mutual fund margin details by querying the backend database and returning structured financial data. This API is essential for stock trading platforms needing to check real-time margin status for users.

---

## Endpoint

```
POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V4/Margin
```

---

## Authentication

- Requires Bearer Token authorization.
- Include the token in the `Authorization` header as:  
  `Authorization: Bearer <JWT_Token>`
- Cookie header may be required with valid session cookie:  
  `Cookie: 5paisacookie=<session_token>`

---

## Request Headers

| Header           | Description                 | Required |
|------------------|-----------------------------|----------|
| Content-Type     | Must be `application/json`  | Yes      |
| Authorization    | Bearer JWT token            | Yes      |
| Cookie           | Session cookie              | Optional |

---

## Request Body

```json
{
  "head": {
    "key": "dVOtp6A6mRzrBO6SdAs7Vf"
  },
  "body": {
    "ClientCode": "50"
  }
}
```

| Parameter  | Type   | Description                           | Required |
|------------|--------|-------------------------------------|----------|
| head.key   | string | API access key                      | Yes      |
| body.ClientCode | string | Unique client identifier for margin data | Yes      |

---

## cURL Example

```bash
curl --location 'https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V4/Margin' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' \
--header 'Cookie: 5paisacookie=kmfrozkfxvest' \
--data '{
    "head": {
        "key": "dfgh34567fgh
    },
    "body": {
        "ClientCode": "456789"
    }
}'
```

---

## Response Body

```json
{
  "head": {
    "responseCode": "5PMarginV4",
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "ClientCode": "50",
    "Status": 0,
    "Message": "",
    "EquityMargin": [
      {
        "AdhocMargin": 0,
        "CollateralValueAfterHairCut": 0,
        "NetAvailableMargin": 0,
        "MarginUtilized": 0,
        "TotalCollateralValue": 0,
        "DerivativeMargin": 0,
        "Ledgerbalance": 0,
        "DPFreeStockValue": 0,
        "GrossHoldingValue": 0,
        "GrossHoldingValueCoverPercentage": 0,
        "HairCut": 0,
        "MarginBlockedforOpenPostion_Cash": 0,
        "MarginBlockedforOpenPostion_Collateral": 0,
        "MarginBlockedforPendingOrder_Cash": 0,
        "MarginBlockedforPendingOrder_Collateral": 0,
        "MFCollateralValueAfterHaircut": 0,
        "OptionsPremium": 0,
        "FundsWithdrawal": 0,
        "MarginBlockedForPendingOrders": 0,
        "FundsPayln": 0,
        "TodaysLoss": 0,
        "Unsettled_Credits": 0
      }
    ],
    "MFMargin": [
      {
        "MFCollateralValue": 0,
        "MFFreeStockValue": 0,
        "MFHaircutValue": 0
      }
    ],
    "TimeStamp": "2025-05-19T12:00:00"
  }
}
```

---

## Response Fields Description

| Field                             | Type     | Description                                 |
|----------------------------------|----------|---------------------------------------------|
| head.responseCode                | string   | Internal API request code                    |
| head.status                      | int      | Status code (0 = Success, others indicate error) |
| head.statusDescription           | string   | Status message                              |
| body.ClientCode                 | string   | Client code reflected back                  |
| body.Status                     | int      | Business logic status (0 = Success)          |
| body.Message                    | string   | Error or success message                     |
| body.EquityMargin               | array    | Array of equity margin details               |
| body.EquityMargin[].AdhocMargin | decimal  | Adhoc margin amount                           |
| body.EquityMargin[].CollateralValueAfterHairCut | decimal | Collateral value after haircut               |
| body.EquityMargin[].NetAvailableMargin | decimal | Net margin available                          |
| ...                            | ...      | Other margin fields as shown above           |
| body.MFMargin                  | array    | Array of mutual fund margin details          |
| body.MFMargin[].MFCollateralValue | decimal | Mutual fund collateral value                   |
| body.MFMargin[].MFFreeStockValue | decimal  | Mutual fund free stock value                   |
| body.MFMargin[].MFHaircutValue   | decimal  | Mutual fund haircut value                      |
| body.TimeStamp                 | datetime | Timestamp of the response                     |

---

## Error Handling

| Status Code | Meaning                  | Description                    |
|-------------|--------------------------|--------------------------------|
| 9           | Invalid Session          | Authentication failed or token expired |
| 2           | Input Validation Error   | Missing or invalid ClientCode or head key |
| 1           | Data Not Found           | No margin info available for the ClientCode |

---


## Additional Notes

- The API supports multi-layer validation: header key, authorization JWT, and client session cookies.
- The margin data returned is consolidated from multiple tables and stored procedures.
- This version (V4) enhances margin details compared to earlier versions, including derivatives and mutual fund margins.
- Always ensure your client code is validated to avoid unauthorized data access.

---

# MarketSnapshot

## Overview of MarketSnapshotV1 API 

This API provides a comprehensive full market snapshot for one or multiple financial instruments (scrips). It delivers essential real-time and historical data such as Last Traded Price (LTP), High, Low, Previous Close, Open Interest, 52-week High/Low, Volume, and other key metrics. This detailed market data is crucial for algorithmic trading systems, enabling informed decision-making and strategic automation.

---

## Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/v1/MarketSnapshot

````

---

## Description

The Market Snapshot API fetches a full market snapshot for the requested scrip(s). It returns detailed market data required to analyze the current and historical state of each instrument, supporting trading strategies with timely and precise information.

- Supports multiple scrips in a single request (up to 50).
- Covers various exchanges and exchange types.
- Useful for live monitoring and backtesting trading algorithms.
- Provides snapshot fields such as LTP, High, Low, Previous Close, Open Interest, 52-week High/Low, Volume, Upper and Lower Circuit Limits, etc.

---

## Request Structure

### Headers

| Header         | Description                              |
|----------------|------------------------------------------|
| Authorization  | Bearer token for API authentication     |
| Content-Type   | application/json                         |

### Body Parameters

| Parameter          | Type              | Required | Description                                                                                  |
|--------------------|-------------------|----------|----------------------------------------------------------------------------------------------|
| ClientCode         | string            | Yes      | Unique client identifier for authentication and request validation.                          |
| Data               | array of objects  | Yes      | List of scrip details to fetch snapshots for.                                               |

### Each Data Object contains:

| Field          | Type    | Required | Description                                                      |
|----------------|---------|----------|------------------------------------------------------------------|
| Exchange       | string  | Yes      | Exchange code (e.g., NSE, BSE).                                  |
| ExchangeType   | string  | Yes      | Exchange segment/type (e.g., 'C' for Cash, 'F' for Futures).     |
| ScripCode      | integer | Conditionally | Numeric code for the scrip. If missing, ScripData must be used.  |
| ScripData      | string  | Conditionally | Symbol or name of the scrip. Used to resolve ScripCode internally.|

---

## Response Structure

### General Fields

| Field           | Type    | Description                           |
|-----------------|---------|-------------------------------------|
| Status          | integer | API execution status code (0=Success, others=Error) |
| Message         | string  | Human-readable status message       |
| Data            | array   | List of market snapshot objects     |

### Each Data Object contains:

| Field               | Type    | Description                                         |
|---------------------|---------|-----------------------------------------------------|
| Exchange            | string  | Exchange code                                      |
| ExchangeType        | string  | Exchange segment/type                              |
| ScripCode           | integer | Numeric scrip code                                |
| LastTradedPrice     | decimal | Last traded price (LTP)                            |
| High                | decimal | Highest price during the session                    |
| Low                 | decimal | Lowest price during the session                     |
| PreviousClose       | decimal | Previous trading day's closing price                 |
| OpenInterest        | integer | Open interest at the time of snapshot                |
| Volume              | integer | Total traded volume for the session                   |
| FiftyTwoWeekHigh    | decimal | 52-week highest price                               |
| FiftyTwoWeekLow     | decimal | 52-week lowest price                                |
| UpperCircuitLimit   | decimal | Upper circuit price limit                            |
| LowerCircuitLimit   | decimal | Lower circuit price limit                            |
| PrevOpenInterest    | integer | Previous day open interest (if applicable)           |

---

## Usage Notes

- The API validates the request for maximum 50 scrips per call to ensure performance and stability.
- ScripCode or ScripData must be provided per scrip object; if ScripCode is missing, the API internally resolves it using ScripData.
- The exchange and exchange type must be specified accurately for correct data retrieval.
- The API requires a valid authorization token.
- The response includes status codes and messages to handle common issues such as invalid client codes, scrip limits exceeded, or data retrieval errors.

---

## Sample Request Body

```json
{
  "body": {
    "ClientCode": "your_client_code",
    "Data": [
      {
        "Exchange": "N",
        "ExchangeType": "C",
        "ScripCode": 12345
      },
      {
        "Exchange": "B",
        "ExchangeType": "C",
        "ScripData": "RELIANCE"
      }
    ]
  }
}
````

---

## Sample Response

```json
{
  "head": {
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "Success",
    "Data": [
      {
        "Exchange": "NSE",
        "ExchangeType": "C",
        "ScripCode": 12345,
        "LastTradedPrice": 3456.78,
        "High": 3500.00,
        "Low": 3400.00,
        "PreviousClose": 3420.50,
        "OpenInterest": 15000,
        "Volume": 200000,
        "FiftyTwoWeekHigh": 4000.00,
        "FiftyTwoWeekLow": 2800.00,
        "UpperCircuitLimit": 3700.00,
        "LowerCircuitLimit": 3200.00,
        "PrevOpenInterest": 14500
      },
      {
        "Exchange": "BSE",
        "ExchangeType": "C",
        "ScripCode": 54321,
        "LastTradedPrice": 2540.10,
        "High": 2600.00,
        "Low": 2500.00,
        "PreviousClose": 2525.00,
        "OpenInterest": 0,
        "Volume": 100000,
        "FiftyTwoWeekHigh": 2700.00,
        "FiftyTwoWeekLow": 1900.00,
        "UpperCircuitLimit": 2650.00,
        "LowerCircuitLimit": 2400.00,
        "PrevOpenInterest": 0
      }
    ]
  }
}
```

---

## Integration Tips for Algo Trading

* Use this API to periodically poll market conditions of your watchlist or portfolio scrips.
* Integrate snapshot data into your trading algorithms for dynamic position sizing, stop loss adjustment, or signal generation.
* Leverage volume and open interest data to assess liquidity and market sentiment.
* Combine with historical data APIs for enhanced trend analysis.
* Monitor circuit limits and 52-week highs/lows to avoid trading outside permissible price bands.

---

## Summary of Market Snapshot API

The Market Snapshot API is an essential building block for any automated trading system targeting Indian markets. It provides timely, rich data across multiple instruments with efficient batch support and secure authorization. Use it to empower your algo trading strategies with real-time market intelligence.

---
# MarketFeed

## Overview of MarketFeedV1 API

This API fetches real-time market feed data for specified financial instruments (scrips). The response includes key market details such as:

- Last Traded Price (LTP)  
- High and Low prices of the day  
- Previous Close price  
- Volume and other market metrics  

The API supports batch requests for multiple scrips and delivers data suited for trading algorithms, decision-making, and market analysis.

---

## Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V1/MarketFeed

````

---

## Request Format

The request consists of:

- **Header:** Contains general metadata (authentication, client info, etc.)  
- **Body:** Includes a list of scrips with their exchange details, last request timestamp, and refresh rate.

### JSON Structure

```json
{
  "head": {
    // Generic request header fields
  },
  "body": {
    "MarketFeedData": [
      {
        "Exch": "string",       // Exchange identifier (e.g., NSE, BSE)
        "ExchType": "string",   // Market segment/type (e.g., equity, futures)
        "ScripCode": int,       // Unique identifier for the scrip
        "ScripData": "string"   // Optional scrip symbol or additional data
      }
      // Can include multiple scrip objects, max 50
    ],
    "LastRequestTime": "datetime",  // Timestamp of the last request (for incremental updates)
    "RefreshRate": "string"          // Desired refresh interval (optional)
  }
}
````

---

## Response Format

The response contains:

* **Status:** Execution status code (0 = success, others indicate errors)
* **Message:** Status message or error details
* **CacheTime:** Cache validity duration in seconds
* **TimeStamp:** Server timestamp for the data snapshot
* **Data:** List of market feed data objects, one per requested scrip

### Sample Response Structure (JSON)

```json
{
  "head": {
    "responseCode": "string",
    "status": int,
    "statusDescription": "string"
  },
  "body": {
    "Status": int,
    "Message": "string",
    "CacheTime": int,
    "TimeStamp": "datetime",
    "Data": [
      {
        "Exch": "char",
        "ExchType": "char",
        "Token": uint,
        "LastRate": double,
        "TotalQty": long,
        "High": double,
        "Low": double,
        "PClose": double,
        "Chg": double,
        "ChgPcnt": double,
        "TickDt": "datetime",
        "Symbol": "string"
      }
      // Multiple scrip data objects
    ]
  }
}
```

---

## Key Points & Usage Notes

* **Scrip Limit:** The maximum number of scrips per request is 50. Exceeding this will return an error status.
* **Incremental Updates:** Using the `LastRequestTime` field helps in fetching only updated market data since the last call, optimizing bandwidth and latency.
* **Market Open/Close Status:** The API internally detects market status and adjusts cache timing accordingly.
* **Error Handling:** Status codes and messages help identify issues such as invalid scrip codes or feed server problems.
* **Data Accuracy:** Calculated fields like price change and percentage change are included for quick algorithmic assessments.

---

## Classes Overview

| Class Name                | Description                            |
| ------------------------- | -------------------------------------- |
| `MarketFeedV1Req`         | Request wrapper with header & body     |
| `MarketFeedV1ReqBody`     | Contains list of scrips and timestamps |
| `MarketFeedV1DataListReq` | Individual scrip request details       |
| `MarketFeedRes`           | API response wrapper                   |
| `MarketFeedResBody`       | Response body with status & data       |
| `MarketFeedDataListRes`   | Market data per scrip                  |

---
### 📦 Response Parameters

#### Head Object

| Field               | Type    | Description                       |
| ------------------- | ------- | --------------------------------- |
| `status`            | Integer | 0 for success, others for errors  |
| `statusDescription` | String  | Status message                    |
| `responseCode`      | String  | Code representing the API version |

#### Body Object

| Field       | Type             | Description                               |
| ----------- | ---------------- | ----------------------------------------- |
| `Status`    | Integer          | 0 for success, others for specific errors |
| `Message`   | String           | Describes the response status             |
| `TimeStamp` | DateTime         | Response timestamp                        |
| `CacheTime` | Integer (ms)     | Suggested cache time in milliseconds      |
| `Data`      | Array of Objects | List of market feed records               |

#### MarketFeedData Object

| Field       | Type     | Description                  |
| ----------- | -------- | ---------------------------- |
| `Exch`      | String   | Exchange                     |
| `ExchType`  | String   | Exchange segment/type        |
| `ScripCode` | Int      | Code of the scrip            |
| `LTP`       | Decimal  | Last traded price            |
| `High`      | Decimal  | Highest price of the session |
| `Low`       | Decimal  | Lowest price of the session  |
| `PClose`    | Decimal  | Previous day close           |
| `Time`      | DateTime | Time when data was fetched   |

---
## Conclusion of Market Feed API

This Market Feed API provides reliable, real-time market data essential for algo trading systems, enabling dynamic decision-making and analysis. The API design supports bulk queries with incremental updates and clear status reporting, making it ideal for integration into automated trading assistants or research platforms.


---
# MarketDepth

## Overview of MarketDepthV3 API

**MarketDepthV3** API provides Level 5 market depth data for one or more stock instruments simultaneously. It returns detailed order book information including price levels, quantities, and number of orders on both the bid and ask sides for the requested scripts.

This API is designed for high-performance algo trading systems to analyze market liquidity and depth efficiently across multiple scripts.

---

## Endpoint

```

POST https://Openapi.5paisa.com/VendorsAPI/Service1.svc/V3/MarketDepth

````

---

## Request Structure

- The API accepts a list of scripts identified by exchange, exchange type, and script code or symbol.
- Supports bulk requests, allowing retrieval of market depth for multiple instruments in one call.

---

## Response Structure

The response contains the following:

### Header

- **responseCode**: API response identifier
- **status**: Numeric status code (0 = Success, 1 = Failure, etc.)
- **statusDescription**: Textual description of the response status

### Body

- **Status**: Status of the market depth data fetch (0 = Success, non-zero indicates error)
- **Message**: Status message or error details
- **MarketDepthData**: List of market depth details per script, each containing:

  - **Exch**: Exchange identifier
  - **ExchType**: Exchange type
  - **ScripCode**: Script identifier code
  - **TimeStamp**: Timestamp of the market depth snapshot
  - **Bids**: Array of bid-level data (up to 5 entries), each with:
    - Price
    - Quantity
    - NumberOfOrders
  - **Asks**: Array of ask-level data (up to 5 entries), each with:
    - Price
    - Quantity
    - NumberOfOrders

---

## Sample Response (Simplified JSON)

```json
{
  "head": {
    "responseCode": "5PMDV3",
    "status": 0,
    "statusDescription": "Success"
  },
  "body": {
    "Status": 0,
    "Message": "Success",
    "MarketDepthData": [
      {
        "Exch": "N",
        "ExchType": "C",
        "ScripCode": 12345,
        "TimeStamp": "2025-05-19T10:15:30Z",
        "Bids": [
          {"Price": 100.5, "Quantity": 200, "NumberOfOrders": 10},
          {"Price": 100.4, "Quantity": 150, "NumberOfOrders": 8}
        ],
        "Asks": [
          {"Price": 100.7, "Quantity": 100, "NumberOfOrders": 6},
          {"Price": 100.8, "Quantity": 120, "NumberOfOrders": 7}
        ]
      }
    ]
  }
}
````

---

## Key Notes

* If a requested script does not have data, the API returns zeroed bid and ask arrays with 5 empty entries each.
* Ensure all script symbols or codes follow the allowed alphanumeric and underscore format.
* The API efficiently aggregates and returns bulk market depth data, reducing the need for multiple API calls.

---


## Usage Tips

* Validate input scripts carefully to avoid invalid characters.
* Use the API to feed market depth data into AI/ML/DL models for enhanced trading strategies.
* The bulk nature of the API helps minimize latency in real-time algo trading systems.

---

*For detailed integration, please refer to the official vendor API documentation.*

---

## 🧾 Request Body Object

| Field             | Type             | Description                                      |
| ----------------- | ---------------- | ------------------------------------------------ |
| `MarketDepthData` | Array of Objects | List of script entries for which depth is needed |

---

## 🧾 Market Depth Request Script Object

| Field          | Type    | Description                                          |
| -------------- | ------- | ---------------------------------------------------- |
| `Exchange`     | String  | Exchange code (e.g., NSE, BSE)                       |
| `ExchangeType` | String  | Exchange segment/type (e.g., C, D)                   |
| `ScripCode`    | Integer | Unique scrip identifier                              |
| `ScripData`    | String  | Script symbol (used if `ScripCode` is not available) |

---

## 🧾 Response Head Object

| Field               | Type    | Description                           |
| ------------------- | ------- | ------------------------------------- |
| `responseCode`      | String  | Unique response code (e.g., `5PMDV3`) |
| `status`            | Integer | Status of the response (0 = success)  |
| `statusDescription` | String  | Description of the response status    |

---

## 🧾 Response Body Object

| Field             | Type             | Description                                      |
| ----------------- | ---------------- | ------------------------------------------------ |
| `Status`          | Integer          | Status of the API result (0 = success, 1 = fail) |
| `Message`         | String           | Message about the result                         |
| `MarketDepthData` | Array of Objects | Contains depth data for each requested scrip     |

---

## 🧾 Market Depth Script Response Object

| Field       | Type     | Description                        |
| ----------- | -------- | ---------------------------------- |
| `Exch`      | String   | Exchange code                      |
| `ExchType`  | String   | Exchange segment/type              |
| `ScripCode` | Integer  | Script code for the instrument     |
| `TimeStamp` | DateTime | Timestamp of market depth snapshot |
| `Bids`      | Array    | Array of bid side depth data       |
| `Asks`      | Array    | Array of ask side depth data       |

---

## 🧾 Market Depth Data Object (Bid/Ask Level)

| Field            | Type    | Description                          |
| ---------------- | ------- | ------------------------------------ |
| `Price`          | Decimal | Price at this depth level            |
| `Quantity`       | Integer | Quantity available at this level     |
| `NumberOfOrders` | Integer | Number of orders at this price level |

---



---
