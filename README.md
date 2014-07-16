# Celery API

The Celery API is an ecommerce logic and storage engine designed to be an essential tool for any developer.

The API is currently in beta. Please contact [help@trycelery.com](mailto:help@trycelery.com) if you intend to deploy our API in production code.

Our API is organized around REST and designed to have predictable, resource-oriented URLs, and to use HTTP response codes to indicate API errors. We support cross-origin resource sharing to allow you to interact securely with our API from a client-side web application (though you should remember that you should never expose your secret API key in any public website's client-side code). JSON will be returned in all responses from the API, including errors.

## Menu

* [Getting Started](#getting-started)
    * [API Basics](#api-basics)
    * [Authentication](#authentication)
    * [Errors](#errors)
    * [Pagination](#pagination)
* [Orders Resources](#orders-resource)
    * [Retrieve an Order](#retrieve-an-order)
    * [Retrieve a List of Orders](#retrieve-a-list-of-orders)
    * [Update an Order](#update-an-order)
    * [Cancel an Order](#cancel-an-order)
    * [Capture an Order](#capture-an-order)
    * [Refund an Order](#refund-an-order)
    * [Fulfill an Order](#fulfill-an-order)
* [Webhooks](#webhooks)

## Getting Started

The base endpoint URL is `https://api.trycelery.com`. Celery is currently on version `v2`, so a request to retrieve all orders would resemble:

```
GET https://api.trycelery.com/v2/orders
```

### API Basics

* **All requests must be made over HTTPS.**
* **Always set required headers.** The API expects requests to be sent with the header `Content-Type` set to `application/json`.
* **All prices are in cents.** Prices are represented as integers in cents. Thus, an API response of `100` would represent `1.00`.

### Authentication

Tokens are used to authenticate your requests. Endpoints require authentication over HTTPS. The preferred method is to authenticate with HTTP header:

    Authorization: {ACCESS_TOKEN}

Alternatively, you can authenticate by adding the query string `?access_token={ACCESS_TOKEN}` to any request URL.

### Errors

Celery uses conventional HTTP response codes to indicate success or failure of an API request. In general, codes in the 2xx range indicate success, codes in the 4xx range indicate an error that resulted from the provided information (e.g. a required parameter was missing, a payment failed, etc.), and codes in the 5xx range indicate an error with Celery's servers.

Our error responses have the format:
```json
{
    "meta": {
        "code": 409,
        "error": {
           "code": 409,
           "message": "User with email already exists."
         }
    },
    "data": "User with email already exists."
}
```

### Pagination

All responses return with a similar structure. Collections returns an array and single objects return an object. Here's an example of a collection response:

```json
{ 
    "meta": {
        "code": 200,
        "paging": {
            "total": 45,
            "count": 10,
            "offset": 5,
            "limit": 10,
            "has_more": true
        }
    },
    "data": [
        {
            ...
        }
    ]
}
```

## Orders Resource

Attributes | Type | Description
-----------|------|------------
_id | string | Unique identifier for the order
user_id | string | Seller unique identifier.
order_status | string | Possible values: `open`, `completed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `refunded`, `failed`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `processing`, `failed`.
currency | string | 3-letter ISO currency code (lowercase). Possible values: `usd`, `cad`, `gbp`, `eur`, `aud`.
notes | string | Seller order notes.
type | string | Possible values: `card`, `paypal_adaptive`, `paypal_adaptive_chained`, `affirm`
shipping_method | string | Shipping method for fulfillment partners.
number | string | Human-readable order number.
linetotal | integer | Sum of the line item amounts. Amount in cents.
discount | integer | Discount amount applied to the order. Amount in cents.
subtotal | integer | Sum of line total less discount. Amount in cents.
shipping | integer | Shipping cost applied. Amount in cents.
taxes | integer | Sales tax applied. Amount in cents.
adjustment | integer | Price adjustments applied. Amount in cents.
total | integer | Total = subtotal + shipping + taxes + adjustments. Amount in cents.
balance | integer | Amount owed to the seller. Amount in cents.
paid | integer | Amount paid to the seller. Amount in cents.
refunded | integer | Amount refunded by the seller. Amount in cents.
created | integer | Unix timestamp in milliseconds.
created_date | string | ISO 8601 timestamp.
updated | integer | Unix timestamp in milliseconds.
updated_date | string | ISO 8601 timestamp.
**Buyer**|object|
buyer.email | string | Buyer's email address.
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.name | string | Buyer's combined first and last name.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address** |object|
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.name | string | Shipping address first and last name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shiping address building, apartment, unit, etc.
shipping_address.city | string | Shipping address city.
shipping_address.state | string | Shipping address state, province, or region.
shipping_address.zip | string | Shipping address ZIP or postal code.
shipping_address.country | string | Shipping address 2-letter ISO country code (lowercase).
shipping_address.phone | string | Shipping address phone number.
**Fulfillment**|object|
fulfillment.created | integer | Date marked fulfilled (Unix timestamp in milliseconds).
fulfillment.created_date | string | Date marked fulfilled (ISO 8601 timestamp).
fulfillment.courier | string | Courier used. Possible values: `ups`, `usps`, `fedex`, `dhl`, `other`.
fulfillment.number | string | Tracking number.
**Payment Source** |object|
payment_source.card.name | string | Name on credit/debit card. Stripe only.
payment_source.card.celery_token | string | Stripe only.
payment_source.card.exp_month | integer | Expiration month of credit/debit card. Stripe only.
payment_source.card.exp_year | integer | Expiration year of credit/debit card. Stripe only.
payment_source.card.last4 | string | Last 4 digits of credit/debit card. Stripe only.
payment_source.card.type | string | Credit/debit card type.
payment_source.card.country | string | Country of credit/debit card. Stripe only.
payment_source.paypal.email | string | Buyer's paypal email address. PayPal only.
payment_source.paypal.preapproval_key | string | PayPal only.
payment_source.paypal.pay_key | string | PayPal only.
payment_source.paypal.redirect_url | string | PayPal only.
payment_source.paypal.ipn | string | PayPal only.
payment_source.paypal.ending | integer | Preapproval expiration date (Unix timestamp in milliseconds). PayPal only.
payment_source.paypal.ending_date | string | Preapproval expiration date (ISO 8601 timestamp). PayPal only.
payment_source.affirm.checkout_token | string | Affirm only.
payment_source.affirm.charge_id | string | Affirm only.
payment_source.airbrite.customer_token | string | Airbrite orders only.
payment_source.airbrite.card_token | string | Airbrite orders only.
payment_source.airbrite.fingerprint | string | Airbrite orders only.
payment_source.airbrite.name | string | Airbrite orders only.
payment_source.airbrite.exp_month | integer | Airbrite orders only.
payment_source.airbrite.exp_year | integer | Airbrite orders only.
payment_source.airbrite.last4 | string | Airbrite orders only.
payment_source.airbrite.country | string | Airbrite orders only.
payment_source.airbrite.type | string | Airbrite orders only.
**Line Items** | [objects]|
line_items[].id | string | Line item identifier (uuid).
line_items[].product_id | string | Product id.
line_items[].product_name | string | Product name.
line_items[].variant_id | string | Variant id.
line_items[].variant_name | string | Variant name.
line_items[].sku | string | Seller defined sku.
line_items[].celery_sku | string | Celery's internal sku.
line_items[].price | integer | Line_item unit price.
line_items[].quantity | integer | Number of units ordered.
line_items[].weight | integer | Line item unit weight.
line_items[].enable_taxes | boolean | Whether taxes apply to line item.
**Payments** | [objects]|
payments[].id | string | Payment identifier.
payments[].gateway | string | Gateway identifier. Possible values: `stripe`, `affirm`, `paypal`.
payments[].charge_id | string | Gateway identifier for the charge (Stripe and Affirm only).
payments[].balance_transaction | string | Stripe only.
payments[].pay_key | string | PayPal only.
payments[].preapproval_key | string | PayPal only.
payments[].capture_id | string | Affirm only.
payments[].transaction_id | string | Affirm only.
payments[].currency | string | Currency.
payments[].amount | integer | Amount captured.
payments[].amount_refunded | integer | Amount refunded.
payments[].paid | boolean | Whether payment was paid.
payments[].capture | boolean | Whether payment was captured.
payments[].refunded | boolean | Whether payment was refunded.
payments[].created | integer | Charge date. Unix timestamp in milliseconds.
payments[].created_date | string | Charge date. ISO 8601 timestamp.
payments[].card.name | string | Stripe only.
payments[].card.last4 | string | Stripe only.
payments[].card.exp_month | string | Stripe only.
payments[].card.exp_year | string | Stripe only.
payments[].card.type | string | Stripe only.
payments[].card.country | string | Stripe only.
**Adjustments** | [objects]|
adjustments[].id | string | Adjustment identifier.
adjustments[].type | string | Possible values: `flat`.
adjustments[].reason | string | Reasoning for price adjustment.
adjustments[].issuer | string | Authorizer of price adjustment.
adjustments[].amount | number | Amount of price adjustment.
**Discounts** | [objects]|
discounts[].code | string | Discount code applied.
discounts[].type: | string | Discount type. Possible values: `flat`, `percent`.
discounts[].amount | integer | Discount amount (5% off means 500 and $10 off means 1000).
discounts[].product_id | string | Product id that discount belongs to.
discounts[].apply | string | Possible values: `once`, `every_time`.
**History** | [objects]|
history[].type | string | [See events](#webhooks).
history[].body | string | Brief description of history type.
history[].created | integer | Unix timestamp in milliseconds.
history[].created_date | string | ISO 8601 timestamp.

### Retrieve an Order

```
GET /v2/orders/{id}
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order

##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "locked": false,
        "order_status": "open",
        "payment_status": "paid",
        "fulfillment_status": "unfulfilled",
        "currency": "usd",
        "notes": null,
        "type": "card",
        "number": "101335478",
        "cancel_url": null,
        "return_url": null,
        "shipping_method": null,
        "deposit": 0,
        "discount": 0,
        "balance": 0,
        "subtotal": 1000,
        "total": 2075,
        "paid": 2075,
        "refunded": 0,
        "adjustment": 0,
        "shipping": 1000,
        "taxes": 75,
        "linetotal": 1000,
        "fee": 42,
        "seller": {
            "_id": "514a114080feb60200000001",
            "email": null,
            "name": null,
            "paypal_email": null
        },
        "buyer": {
            "sort_name": "brian nguyen",
            "email": "brian@trycelery.com",
            "first_name": "Brian",
            "last_name": "Nguyen",
            "name": "Brian Nguyen",
            "phone": null,
            "company": null,
            "notes": null
        },
        "shipping_address": {
            "email": "brian@trycelery.com",
            "first_name": "Brian",
            "last_name": "Nguyen",
            "name": "Brian Nguyen",
            "line1": "123 Main Street",
            "line2": "Unit 101",
            "city": "San Francisco",
            "state": "ca",
            "zip": "94105",
            "country": "us",
            "phone": null,
            "company": null
        },
        "billing_address": {
            "email": null,
            "first_name": null,
            "last_name": null,
            "name": null,
            "line1": null,
            "line2": null,
            "city": null,
            "state": null,
            "zip": null,
            "country": null,
            "phone": null
        },
        "fulfillment": {
            "courier": null,
            "number": null,
            "created": null,
            "created_date": null
        },
        "payment_source": {
            "card": {
                "name": null,
                "celery_token": "c31310e652f5a7d7254484668c52686d09cd665414c6d3cae04f92660c3d930cad55a378cfa2c76fcbf8dc0b129a4f0baac0e3794c4485510a36cfbef7461c45dfc1d52d6e03a28d03af86f7ec405577f0d1c7c7a0487ca780003f86b595bca0",
                "exp_month": 11,
                "exp_year": 20,
                "last4": "4242",
                "country": "us",
                "type": "visa"
            },
            "paypal": {
                "email": null,
                "preapproval_key": null,
                "pay_key": null,
                "redirect_url": null,
                "ipn": null,
                "ending": null,
                "ending_date": null
            },
            "affirm": {
                "checkout_token": null,
                "charge_id": null
            },
            "airbrite": {
                "customer_token": null,
                "card_token": null,
                "fingerprint": null,
                "name": null,
                "exp_month": 0,
                "exp_year": 0,
                "last4": null,
                "country": null,
                "type": null
            }
        },
        "discount_codes": [],
        "line_items": [
            {
                "id": "0fcfa92c-5214-d4bf-ac41-86f26fb9c9b9",
                "product_id": "531e0b012cf9766885f781b7",
                "product_name": "koala 10",
                "variant_id": null,
                "variant_name": null,
                "sku": "koala_10",
                "celery_sku": "531e0b012cf9766885f781b7",
                "price": 1000,
                "quantity": 1,
                "weight": 0.5,
                "enable_taxes": true
            }
        ],
        "adjustments": [],
        "discounts": [],
        "payments": [
            {
                "id": "9969f10f-6c29-4d36-ba84-c60abd4bd7ed",
                "charge_id": "ch_4OrpNFZKd4NOBz",
                "balance_transaction": "txn_4Orp4IPXvI0QCc",
                "pay_key": null,
                "preapproval_key": null,
                "capture_id": null,
                "transaction_id": null,
                "currency": "usd",
                "amount": 2075,
                "amount_refunded": 0,
                "fee": 132,
                "paid": true,
                "captured": true,
                "created": 1405323722000,
                "created_date": "2014-07-14T07:42:02.000Z",
                "refunded": null,
                "refunded_date": null,
                "card": {
                    "name": "",
                    "last4": "4242",
                    "exp_month": null,
                    "exp_year": null,
                    "type": "visa",
                    "country": "us"
                }
            }
        ],
        "fulfillments": [],
        "history": [
            {
                "type": "order.created",
                "body": "Order was created.",
                "created": 1405323703464,
                "created_date": "2014-07-14T07:41:43.464Z"
            },
            {
                "type": "order.charge.succeeded",
                "body": "Order was charged.",
                "created": 1405323723032,
                "created_date": "2014-07-14T07:42:03.032Z"
            }
        ],
        "_id": "53c389b7eba65a000061e12f",
        "user_id": "514a114080feb60200000001",
        "version": "v2",
        "created": 1405323701278,
        "updated": 1405323722893,
        "created_date": "2014-07-14T07:41:41.278Z",
        "updated_date": "2014-07-14T07:42:02.893Z",
        "metadata": {}
    }
}
```

### Retrieve a List of Orders

```
GET /v2/orders
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
sort | string | Sort by. Default is `created`.
order | string | Order by. Default is `desc`. Possible values: `desc`, `asc`.
skip | string | Default is `0`.
limit | string | A limit on the number of objects to be returned. Default is `50`.
created | integer | A filter on the list based on object `created` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
updated | integer | A filter on the list based on object `updated` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
order_status | string | Possible values: `open`, `completed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `failed`, `refunded`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `procesing`, `failed`.
buyer.name | string | A filter on the list based on buyer name. This filter will perform a regex on the value.
buyer.email | string | A filter on the list based on buyer email. This filter will perform a regex on the value.

- gt:  Return values where the relevent field is after this timestamp (in ms).
- gte: Return values where the relevant field is after or equal to this timestamp (in ms).
- lt:  Return values where the relevant field is before this timestamp (in ms).
- lte: Return values where the relevant field is before or equal to this timestamp (in ms).

If I wanted to query for orders created at or after January 1, 2014 12:00 AM UTC, my query string would include `created[gte]=1388534400000`.

If I wanted to query for orders purchased by `Brian Nguyen`, my query string would include `buyer.name=Brian+Nguyen`.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders?created[gte]=1388534400000
```

##### Example Response
```json
{
    "meta": {
        "code": 200,
        "paging": {
            "total": 2,
            "count": 2,
            "offset": 0,
            "limit": 50,
            "has_more": false
        }
    },
    "data": [
        {
            ...
        },
        {
            ...
        }
    ]
}
```

### Update an Order

Warning! Modifying data that is not `buyer`, `shipping_address`, or `line items` may have unintended effects on the order. If you are modifying these objects, please be sure to include entire object.

```
PUT /v2/orders/{id}
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
_id | string | **Required**. Unique identifier for the order.
**Buyer** | object |
buyer.email | string | Buyer's email address.
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address** | object |
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shiping address building, apartment, unit, etc.
shipping_address.city | string | Shipping address city.
shipping_address.state | string | Shipping address state, province, or region. US or CA states should use their 2-letter ISO state code and be lowercase.
shipping_address.zip | string | Shipping address ZIP or postal code.
shipping_address.country | string | Shipping address 2-letter ISO country code (lowercase).
shipping_address.phone | string | Shipping address phone number.
**Line Items** | [object] |
line_items[].id | string | Line item identifier (uuid).
line_items[].product_id | string | Product id.
line_items[].variant_id | string | Variant id.
line_items[].quantity | integer | Number of units ordered.

##### Example Request
This example request updates both the buyer and shipping_address information.

```
$ curl -X PUT -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440 \
-d'
{
    "buyer": {
        "email": "first@last.com",
        "first_name": "First",
        "last_name": "Last",
        "phone": "555-555-5555"
    },
    "shipping_address": {
        "first_name": "First"
        "last_name": "Last",
        "company": "Celery",
        "line1": "123 Street",
        "line2": null,
        "city": "New York",
        "state": "ny",
        "zip": "10012",
        "country": "us",
        "phone": "555-555-5555"
    }
}
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "buyer": {
            "email": "first@last.com",
            "first_name": "First",
            "last_name": "Last",
            "name": "First Last",
            "sort_name": "first last",
            "phone": "555-555-5555"
        },
        "shipping_address": {
            "first_name": "First",
            "last_name": "Last",
            "name": "First Last",
            "company": "Celery",
            "line1": "123 Street",
            "line2": null,
            "city": "New York",
            "state": "ny",
            "zip": "10012",
            "country": "us",
            "phone": "555-555-5555"
        },
        ...
    }
}
```

##### Example Request

This example request updates the quantity of a line item to 2 units. Please be sure to include the entire line_items array, even if updating only one line item; otherwise, you may accidentally delete any others. At minimum, the required line item properties include `id`, `product_id`, `quantity`, and `variant_id` (if applicable).

WARNING: Updating line items will fetch the latest product details and may cause order prices (subtotal, taxes, shipping, total, balance) to change.

```
$ curl -X PUT -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440 \
-d'
{
    "line_items":
    [
        {
            "id": "550e8400-e29b-41d4-a716-446655440000",
            "product_id": "530e40d428ee4100002bfa78",
            "quantity": 2
        }
    ]
}
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        ...,
        "line_items": [
            {
                "id": "550e8400-e29b-41d4-a716-446655440000",
                "product_id": "530e40d428ee4100002bfa78",
                "quantity": 2,
                "price": 1000,
                "weight": 0,
                "enable_taxes": true,
                "product_name": "Koala Bears",
                "variant_name": null,
                "variant_id": null,
                "celery_sku": "530e40d428ee4100002bfa78",
                "sku": "KOALA-1"
            }
        ],
        ...
    }
}
```

### Cancel an Order

This will cancel the order. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/order_cancel
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/order_cancel
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "order_status": "cancelled",
        "history": [
            {
                "type": "order.created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "order.cancelled",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was cancelled."
            }
        ],
        ...
    }
}
```


### Capture an Order

This will capture payment for the order. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/payment_charge
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order

##### Example Request

```
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/53c389b7eba65a000061e12f/payment_charge
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "payment_status": "paid",
        "payments": [
            {
                "id": "9969f10f-6c29-4d36-ba84-c60abd4bd7ed",
                "charge_id": "ch_4OrpNFZKd4NOBz",
                "balance_transaction": "txn_4Orp4IPXvI0QCc",
                "pay_key": null,
                "preapproval_key": null,
                "capture_id": null,
                "transaction_id": null,
                "currency": "usd",
                "amount": 2075,
                "amount_refunded": 0,
                "fee": 132,
                "paid": true,
                "captured": true,
                "created": 1405323722000,
                "created_date": "2014-07-14T07:42:02.000Z",
                "refunded": null,
                "refunded_date": null,
                "card": {
                    "name": "",
                    "last4": "4242",
                    "exp_month": null,
                    "exp_year": null,
                    "type": "visa",
                    "country": "us"
                }
            }
        ],
        "history": [
            {
                "type": "order.created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "order.charge.succeeded",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was charged 20.75."
            }
        ],
        "balance": 0,
        "paid": 2075,
        ...
    }
}
```

### Refund an Order

This will refund payment for the order. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/payment_refund
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/53c389b7eba65a000061e12f/payment_refund
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "payment_status": "refunded",
        "payments": [
            {
                "id": "9969f10f-6c29-4d36-ba84-c60abd4bd7ed",
                "charge_id": "ch_4OrpNFZKd4NOBz",
                "balance_transaction": "txn_4Orp4IPXvI0QCc",
                "pay_key": null,
                "preapproval_key": null,
                "capture_id": null,
                "transaction_id": null,
                "currency": "usd",
                "amount": 2075,
                "amount_refunded": 2075,
                "fee": 132,
                "paid": true,
                "captured": true,
                "created": 1405323722000,
                "created_date": "2014-07-14T07:42:02.000Z",
                "refunded": 1405377769000,
                "refunded_date": "2014-07-14T22:42:49.000Z",
                "card": {
                    "name": "",
                    "last4": "4242",
                    "exp_month": null,
                    "exp_year": null,
                    "type": "visa",
                    "country": "us"
                }
            }
        ],
        "history": [
            {
                "type": "order.created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "order.charge.succeeded",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was charged 20.75."
            },
            {
                "type": "order.refund.succeeded",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was refunded 20.75."
            }

        ],
        "balance": 0,
        "paid": 2075,
        "refunded": 2075,
        ...
    }
}
```


### Fulfill an Order

This convenience endpoint will mark the fulfillment_status as `fulfilled` and allow you to optionally add `courier` and `number`. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/fulfillment_succeed
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order
courier | string | Courier that will fulfill order. Possible values: `ups`, `usps`, `fedex`, `dhl`, `other`.
number | string | Tracking number.

##### Example Request

```
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/fulfillment_succeed \
-d'
{
    "courier" : "ups",
    "number" : "1zasdfajkfsdljasdf"
}
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        ...,
        "fulfillment_status": "fulfilled",
        "fulfillments": 
        [
            {
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "courier": "ups",
                "number": "1zasdfajkfsdljasdf"
            }
        ],
        "history": [
            {
                "type": "order.created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "order.fulfillment.succeeded",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was fulfilled."
            }
        ],
        ...
    }
}
```

## Webhooks

Events are our way of letting you know about something interesting has happened with your order. The Celery API can send events directly to your server using webhooks. Webhooks are managed in your account settings.

To acknowledge that you received the webhook without any problem, your server should return a 200 HTTP status code. Any response code outside the 2xx and 3xx range will indicate to Celery that you did not receive the webhook. When a webhook is not received for whatever reason, Celery will continue trying to send the webhook for up to 3 times.

This is a list of all the types of events we currently send. We may add more at any time, so you shouldn't rely on only these types existing in your code.

* order.created
* order.completed
* order.cancelled
* order.line_items.updated
* order.shipping_address.updated
* order.adjustments.updated
* order.payment_source.updated
* order.charge.succeeded
* order.charge.failed
* order.refund.succeeded
* order.refund.failed
* order.fulfillment.succeeded
* order.fulfillment.processing
* order.fulfillment.failed

##### Example Response

```json
{
    "type": "order.created",
    "data": {
        ...
    }
}
