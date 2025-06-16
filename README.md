# Quantsapp Python SDK

<picture align="center">
  <img alt="Quantsapp Logo" src="https://s3.ap-south-1.amazonaws.com/images.quantsapp.com/webapp/assets/splash_logo.png">
</picture>


<p align="center">
    <em>Quantsapp Python SDK to interact with Quantsapp API</em>
</p>
<p align="center">
    <a href="https://pypi.org/project/quantsapp" target="_blank">
    <img src="https://img.shields.io/pypi/v/quantsapp?color=%23f49b0f&label=PyPI%20Version" alt="Package version">
    </a>
    <a href="https://pypi.org/project/quantsapp" target="_blank">
    <img src="https://img.shields.io/pypi/pyversions/quantsapp?color=%23f49b0f&label=PyPI%20-%20Python%20Version" alt="Supported Python versions">
    </a>
    <a href="https://pypi.org/project/quantsapp" target="_blank">
    <img src="https://img.shields.io/pypi/dm/quantsapp?label=PyPI%20-%20Downloads&color=%23f49b0f" alt="Downloads">
    </a>
</p>


---

**Python SDK Documentation**: <a href="https://python-sdk.quantsapp.com" target="_blank">https://python-sdk.quantsapp.com</a>


**Python SDK Source Code**: <a href="https://github.com/quantsapp/python-sdk" target="_blank">https://github.com/quantsapp/python-sdk</a>


**API Documentation**: To use Python API instead of SDK interaction wth different programming language, please refer <a href="https://ordersapi.quantsapp.com" target="_blank">https://ordersapi.quantsapp.com</a>

---

Quantsapp Python SDK is a fast (high-performance) SDK for Quantsapp Order Execution.

The key features are:

-   Order placement, modification and cancellation
-   Fetching user info including holdings, positions, margin and order book.
-   Fetching order status and trade information.
-   Getting live orders streaming using websockets.

---
## Installation
<div>
```title="Command Prompt / Terminal"
pip install quantsapp
```
</div>

---

## Usage

### Authentication


Get your API key & Secret key from 
<a href="https://web.quantsapp.com/profile" target="_blank">
    here
    </a>
```py
import quantsapp

# Login to get session context which can be used for other modules
session_context = quantsapp.Login(
    api_key='<API_KEY>',
    secret_key='<SECRET_KEY>',
).login()
```

---

### Market Status


```py
# Get the Market Timings
qapp_market_timings = quantsapp.MarketTimings(
    exchange=qapp_execution.variables.Exchange.NSE_FNO,
)

print(f"Market opened now = {qapp_market_timings.is_market_open()}")
# Market opened now = True | False

print(f"Market Open Time = {qapp_market_timings.dt_open}")
# Market Open Time = datetime.datetime object

print(f"Market Close Time = {qapp_market_timings.dt_close = }")
# Market Open Time = datetime.datetime object
```

---


### Execution Modules
```py
def broker_order_update(update):
    """This is the callback function to where the real-time broker order updates are being sent"""
    print(f"Broker Order Update received -> {update}")

# Create the Execution object
qapp_execution = quantsapp.Execution(
    session_context=session_context,
    order_updates_callback=broker_order_update,  # Optional
)
```

---

#### Execution Variables
All execution variables are python `enum` datatypes
```py
# Exchange enums
quantsapp.variables.Exchange
    - NSE_FNO

# Broker enums
quantsapp.variables.Broker
    - MSTOCK
    - CHOICE
    - DHAN
    - FIVEPAISA
    - FIVEPAISA_XTS
    - FYERS
    - MOTILAL_OSWAL
    - UPSTOX
    - ALICEBLUE
    - NUVAMA
    - SHAREKHAN
    - ANGEL
    - ZERODHA
quantsapp.variables.BrokerRole
    - OWNER
    - READER
    - EXECUTOR

# Order enums
quantsapp.variables.Order.ProductType
    - INTRADAY
    - NRML
quantsapp.variables.Order.Validity
    - DAY
    - IOC
quantsapp.variables.Order.OrderType
    - LIMIT
    - MARKET
    - SLL
    - SL_M
quantsapp.variables.Order.BuySell
    - BUY
    - SELL
quantsapp.variables.Order.Status
    - CANCELLED
    - COMPLETED
    - PARTIAL
    - PENDING
    - FAILED
    - PLACED
    - REJECTED
    - TRANSIT
```
---

#### Broker Accounts Management

##### Listing Available Brokers
```py
available_brokers = qapp_execution.list_available_brokers()

if available_brokers.success:
    print(available_brokers.body)
    """sample
    {
        'access_token_login': [
            Broker.CHOICE,
            Broker.DHAN,
        ],
        'oauth_login': [
            Broker.MSTOCK,
            Broker.FIVEPAISA,
            Broker.FIVEPAISA_XTS,
            Broker.FYERS,
            Broker.ZERODHA,
            Broker.MOTILAL_OSWAL,
            Broker.UPSTOX,
            Broker.ALICEBLUE,
            Broker.NUVAMA,
        ]
    }
    """
else:
    print(f"Error on listing available brokers -> {available_brokers.error}")

if qapp_execution.variables.Broker.CHOICE in available_brokers.body['access_token_login']:
    print('CHOICE broker can be added to Quantsapp via this SDK')

if qapp_execution.variables.Broker.MSTOCK in available_brokers.body['access_token_login']:
    print("MSTOCK broker can't be added to Quantsapp via this SDK. Please add it via https://web.quantsapp.com/broker")
```

---


##### Listing Mapped Brokers
```py
mapped_brokers_resp = qapp_execution.list_mapped_brokers(
    resync_from_broker=False,
    from_cache=False,
)
if mapped_brokers_resp.success:
    print(mapped_brokers_resp.body)
    """sample
    {
        'brokers': {
            Broker.CHOICE: {
                '<CLIENT_ID>': {
                    'margin': {
                        Exchange.NSE_FNO: 7958.67,
                        'dt': datetime.datetime(2025, 5, 23, 13, 8, 1, tzinfo=datetime.timezone.utc)
                    },
                    'name': '<NAME>',
                    'role': BrokerRole.EXECUTOR,
                    'valid': True,
                    'validity': datetime.datetime(2025, 6, 19, 11, 57, 18, tzinfo=datetime.timezone(datetime.timedelta(seconds=19800), 'IST'))
                }
            },
        },
        'next_margin': datetime.datetime(2025, 5, 23, 13, 23, tzinfo=datetime.timezone.utc),
        'version': 42
    }
    """
else:
    print(f"Error on getting mapped brokers -> {mapped_brokers_resp.error}")
```

---


##### Add Broker

> ###### DHAN
```py
add_dhan_broker_resp = qapp_execution.add_broker(
    broker=qapp_execution.variables.Broker.DHAN,
    login_credentials={
        'access_token': '<DHAN_ACCESS_TOKEN>',
    },
)
if add_dhan_broker_resp.success:
    print(f"Dhan Broker added -> {add_dhan_broker_resp.body = }")
else:
    print(f"Dhan broker failed to add -> {add_dhan_broker_resp.error}")
```

> ###### CHOICE
```py
add_choice_broker_resp = qapp_execution.add_broker(
    broker=qapp_execution.variables.Broker.CHOICE,
    login_credentials={
        'mobile': '1234567890',
        'client_access_token': '<CHOICE_ACCESS_TOKEN>',
    },
)
if add_choice_broker_resp.success:
    print(f"Choice Broker added -> {add_choice_broker_resp.body = }")
else:
    print(f"Choice broker failed to add -> {add_choice_broker_resp.error}")
```

---



##### Delete Broker
```py
broker_delete_resp = qapp_execution.delete_broker(
    broker=quantsapp.Broker.CHOICE,
    client_id='<CLIENT_ID>',
)
if broker_delete_resp.success:
    print(f"Broker deleted -> {broker_delete_resp.body = }")
else:
    print(f"Broker failed to delete -> {broker_delete_resp.error}")
```

---

#### Order Management

##### Get Orderbook
```py

get_orders_resp = qapp_execution.get_orderbook(
    broker=qapp_execution.variables.Broker.DHAN,
    client_id='<CLIENT_ID>',
    ascending=False,
    from_cache=True,            # Return the orders from either local cache or Quantsapp server
    resync_from_broker=False,   # Resync the orders from Broker api again

    # Optional (any combo of below filters)
    filters={
        'product': qapp_execution.variables.Order.ProductType.INTRADAY,
        'order_status': qapp_execution.variables.Order.Status.CANCELLED,
        'order_type': qapp_execution.variables.Order.OrderType.LIMIT,
        'instrument': 'NIFTY:22-May-25:p:250000',  # instrument structure = 'SYMBOL:EXPIRY:OPTION_TYPE:STRIKE'
    },
)
if get_orders_resp.success:
    print(get_orders_resp.body)
    """sample
    [
        {
            'b_orderid': '42250523454209',
            'b_usec_update': datetime.datetime(2025, 5, 23, 9, 50, 44, tzinfo=datetime.timezone.utc),
            'broker_client': BrokerClient(broker=Broker.DHAN, client_id='1100735577'),
            'buy_sell': OrderBuySell.BUY,
            'e_orderid': '1200000163510266',
            'instrument': 'NIFTY:05-Jun-25:c:25650',
            'o_ctr': 10,
            'order_status': OrderStatus.CANCELLED,
            'order_type': OrderType.LIMIT,
            'price': 1.45,
            'product_type': OrderProductType.NRML,
            'q_usec': datetime.datetime(2025, 5, 23, 9, 50, 44, tzinfo=datetime.timezone.utc),
            'qty': 75,
            'qty_filled': 0,
            'stop_price': 0.0,
            'userid': 500131
        },
    ]
    """
else:
    print(f"Error on order listing -> {get_orders_resp.error}")
```

---



##### Place Order
```py
place_order_resp = qapp_execution.place_order(
    broker_accounts=[
        {
            'broker': qapp_execution.variables.Broker.CHOICE,
            'client_id': '<CLIENT_ID>',
            'lot_multiplier': 1,  # Optional - Default is 1
        }
    ],
    exchange=qapp_execution.variables.Exchange.NSE_FNO,
    product=qapp_execution.variables.Order.ProductType.NRML,
    order_type=qapp_execution.variables.Order.OrderType.LIMIT,
    validity=qapp_execution.variables.Order.Validity.DAY,
    margin_benefit=True,  # Optional - Default = True
    legs=[
        {
            'qty': 75,
            'price': 0.05,
            'instrument': 'NIFTY:26-Jun-25:c:26500',  # Call Option
            'buy_sell': 'b',  # Buy='b', Sell='s'
            # 'stop_price': 5.4,  # Only for Stop Loss Limit Order
        },
        {
            'qty': 75,
            'price': 0.05,
            'instrument': 'NIFTY:26-Jun-25:p:23500',  # Put Option
            'buy_sell': 's',
        },
        {
            'qty': 75,
            'price': 25100,
            'instrument': 'NIFTY:26-Jun-25:x',  # Future
            'buy_sell': 'b',
        },
    ],
)
if place_order_resp.success:
    print(place_order_resp.body)
else:
    print(f"Error on placing order -> {place_order_resp.error}")
```

---



##### Modify Order
```py
modify_order_resp = qapp_execution.modify_order(
    broker=qapp_execution.variables.Broker.CHOICE,
    client_id='<CLIENT_ID>',
    b_orderid='<BROKER_ORDER_ID>',
    e_orderid='<EXCHANGE_ORDER_ID>',
    qty=75,
    price=0.15,
    stop_price=0.05,  # Only for Stop Loss Order
)
if modify_order_resp.success:
    print(modify_order_resp.body)
else:
    print(f"Error on modifying order -> {modify_order_resp.error}")
```

---



##### Cancel Orders
Cancel specific orders of multiple Broker accounts
```py
cancel_orders_resp = qapp_execution.cancel_orders(
    orders_to_cancel=[
        {
            'broker': qapp_execution.variables.Broker.CHOICE,
            'client_id': '<CLIENT_ID>',
            'order_ids': [
                {
                    'b_orderid': '<BROKER_ORDER_ID>',
                    'e_orderid': '<EXCHANGE_ORDER_ID>',
                },
            ],
        },
        {
            'broker': qapp_execution.variables.Broker.DHAN,
            'client_id': '<CLIENT_ID>',
            'order_ids': [
                {
                    'b_orderid': '<BROKER_ORDER_ID>',
                    'e_orderid': '<EXCHANGE_ORDER_ID>',
                },
            ],
        },
    ],
)
if cancel_orders_resp.success:
    print('Cancel Orders:-')
    pprint(cancel_orders_resp.body)
else:
    print(f"Error on cancel order -> {cancel_orders_resp.error}")
```


---


##### Cancel All Orders
Cancel all orders related to specific Broker account
```py
cancel_all_orders_resp = qapp_execution.cancel_all_orders(
    broker=qapp_execution.variables.Broker.CHOICE,
    client_id='<CLIENT_ID>',
)
if cancel_all_orders_resp.success:
    print(cancel_all_orders_resp.body)
else:
    print(f"Error on cancel all orders -> {cancel_all_orders_resp.error}")
```


---


##### Get Positions
```py
get_positions_resp = qapp_execution.get_positions(
    broker_clients=[
        {
            'broker': qapp_execution.variables.Broker.MSTOCK,
            'client_id': '<CLIENT_ID>',
        },
        {
            'broker': qapp_execution.variables.Broker.FIVEPAISA,
            'client_id': '<CLIENT_ID>',
        },
    ],
    resync_from_broker=False,
    from_cache=True,
)
if get_positions_resp.success:
    print(get_positions_resp.body)
else:
    print(f"Error on get positions -> {get_positions_resp.error}")
```


---


##### Get Positions Combined
```py
get_positions_consolidated_resp = qapp_execution.get_positions_combined(
    broker_clients=[
        {
            'broker': qapp_execution.variables.Broker.MSTOCK,
            'client_id': '<CLIENT_ID>',
        },
        {
            'broker': qapp_execution.variables.Broker.CHOICE,
            'client_id': '<CLIENT_ID>',
        },
    ],
    resync_from_broker=False,
    from_cache=True,
)
if get_positions_consolidated_resp.success:
    print(get_positions_consolidated_resp.body)
else:
    print(f"Error on get consolidated positions -> {get_positions_consolidated_resp.error}")
```

---



##### Get Broker Websocket Status
```py
get_broker_ws_conn_status_resp = qapp_execution.get_broker_websocket_status(
    broker=qapp_execution.variables.Broker.MSTOCK,
    client_id='<CLIENT_ID>',
)
if get_broker_ws_conn_status_resp.success:
    print(get_broker_ws_conn_status_resp.body)
else:
    print(f"Error on get ws connection status -> {get_broker_ws_conn_status_resp.error}")
```


---


##### Broker Websocket Reconnect
```py
broker_ws_re_conn_resp = qapp_execution.broker_websocket_reconnect(
    broker=qapp_execution.variables.Broker.MSTOCK,
    client_id='<CLIENT_ID>',
)
if broker_ws_re_conn_resp.success:
    print(broker_ws_re_conn_resp.body)
else:
    print(f"Error on ws re-connection -> {broker_ws_re_conn_resp.error}")
```

---
