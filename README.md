# Celery API

The Celery API is an ecommerce logic and storage engine designed to be an essential tool for any developer.

The API is currently in beta. Please contact [help@trycelery.com](mailto:help@trycelery.com) if you intend to deploy our API in production code.

Our API is organized around REST and designed to have predictable, resource-oriented URLs, and to use HTTP response codes to indicate API errors. We support cross-origin resource sharing to allow you to interact securely with our API from a client-side web application (though you should remember that you should never expose your secret API key in any public website's client-side code). Although both JSON and XML are supported, JSON is the preferred method for API consumption. By default, JSON will be returned in all responses from the API and webhooks.

To view `v1` API documentation, go to [https://legacy.trycelery.com/developer](https://legacy.trycelery.com/developer).

## Menu

* [Getting Started](#getting-started)
    * [API Basics](#api-basics)
    * [Authentication](#authentication)
    * [Errors](#errors)
    * [Pagination](#pagination)
    * [Rate Limits] (#rate-limits)
* [Orders Resource](#orders-resource)
    * [Checkout with Credit or Debit Card](#checkout-with-credit-or-debit-card)
    * [Checkout with PayPal](#checkout-with-paypal)
    * [Retrieve an Order](#retrieve-an-order)
    * [Retrieve a List of Orders](#retrieve-a-list-of-orders)
    * [Count Orders](#count-orders)
    * [Update an Order](#update-an-order)
    * [Cancel an Order](#cancel-an-order)
    * [Charge an Order](#charge-an-order)
    * [Refund an Order](#refund-an-order)
    * [Fulfill an Order](#fulfill-an-order)
* [Products Resource](#products-resource)
    * [Retrieve a List of Products](#retrieve-a-list-of-products)
    * [Retrieve a Product](#retrieve-a-product)
* [Collections Resource](#collections-resource)
    * [Retrieve a List of Collections](#retrieve-a-list-of-collections)
    * [Retrieve a Collection](#retrieve-a-collection)
* [Coupons Resource](#coupons-resource)
    * [Create a Coupon](#create-a-coupon)
    * [Retrieve a List of Coupons](#retrieve-a-list-of-coupons)
    * [Retrieve a Coupon](#retrieve-a-coupon)
    * [Validate Coupon Code](#validate-coupon-code)
* [Users Resource](#users-resource)
    * [Retrieve Tax Rate](#retrieve-tax-rate)
* [Webhooks](#webhooks)

## Getting Started

The base endpoint URL is `https://api.trycelery.com`. Celery is currently on version `v2`, so a request to retrieve all orders would resemble:

```
GET https://api.trycelery.com/v2/orders
```

To access the sandbox environment, create a new account at [https://dashboard-sandbox.trycelery.com](https://dashboard-sandbox.trycelery.com). The sandbox base endpoint URL is `https://api-sandbox.trycelery.com`. The sandbox environment is separate from production and does not use live money.

### API Basics

* **All requests must be made over HTTPS.**
* **Always set required headers.** The API expects requests to be sent with the header `Content-Type` and `Accept` set to `application/json` or `application/xml`.
* **All prices are in cents.** Prices are represented as integers in cents. Thus, an API response of `100` would represent `1.00`.

### Authentication

Tokens are used to authenticate your requests. Your access token can be retrieved in the Celery dashboard. Endpoints require authentication over HTTPS. The preferred method is to authenticate with HTTP header:

    Authorization: ACCESS_TOKEN

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

Attributes | Type | Description
-----------|------|------------
total | integer | Total number of records in list.
count | integer | Number of records on current page.
limit | integer | A limit on the number of records to be returned. Default is `100`.
offset | integer | Starting record to be returned. Default is `0`.
page | integer | Current page number.
pages | integer | Total number of pages.
has_more | boolean | Whether current page is last page.

All responses return with a similar structure. Collections returns an array and single objects return an object. Here's an example of a collection response:

```json
{
    "meta": {
        "code": 200,
        "paging": {
            "total": 45,
            "count": 10,
            "limit": 10,
            "offset": 5,
            "page": 2,
            "pages": 5,
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

By default, requests return a limit of 100 records per page. You can change this by setting the `limit` parameter in the request. For example, `"limit": 10` will return 10 records per page. The Celery API returns a maximum of `100` records per page.

When there are more records available than can be returned in a single page, you can paginate through them by setting the `offset` or `page`.

### Rate Limits

There is a default rate limit of 120 requests per minute. If for any reason this is not enough, please contact us.

## Orders Resource

Attributes | Type | Description
-----------|------|------------
_id | string | Order unique identifier.
user_id | string | Seller unique identifier.
order_status | string | Possible values: `open`, `completed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `refunded`, `failed`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `processing`, `failed`.
currency | string | 3-letter ISO currency code (lowercase). Possible values: `usd`, `cad`, `gbp`, `eur`, `aud`.
notes | string | Seller order notes.
type | string | Possible values: `card`, `paypal_adaptive`, `paypal_adaptive_chained`, `affirm`.
shipping_method | string | Shipping method for fulfillment partners.
number | string | Human-readable order number.
linetotal | integer | Sum of the line item amounts. Amount in cents.
discount | integer | Discount amount applied to the order. Amount in cents.
subtotal | integer | Sum of line totals less discount. Amount in cents.
shipping | integer | Shipping cost applied. Amount in cents.
taxes | integer | Sales tax applied. Amount in cents.
adjustment | integer | Price adjustments applied. Amount in cents.
total | integer | Total = subtotal + shipping + taxes + adjustments. Amount in cents.
balance | integer | Amount owed to the seller. Amount in cents.
paid | integer | Gross amount paid to the seller. Amount in cents.
refunded | integer | Amount refunded by the seller. Amount in cents.
fee | integer | Celery application fee. Amount in cents.
created | integer | Unix timestamp in milliseconds.
created_date | string | ISO 8601 timestamp.
updated | integer | Unix timestamp in milliseconds.
updated_date | string | ISO 8601 timestamp.
**Buyer**|object|
buyer.email | string | Buyer's email address.
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.name | string | Buyer's combined first and last name.
buyer.company | string | Buyer's company.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address** |object|
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.name | string | Shipping address first and last name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shipping address building, apartment, unit, etc.
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
**Payment Source** | object |
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
payment_source.paypal.seller_email | string | Seller's PayPal email address. PayPal only.
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
**Client Details** | object |
client_details.ip | string | string | Browser IP from Celery checkout.
client_details.user_agent | string | Browser user agent from Celery checkout.
client_details.accept_language | string | Browser language from Celery checkout.
client_details.referer | string | Browser referer from Celery checkout.
**Line Items** | [objects] |
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
**Payments** | [objects] |
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
payments[].captured | boolean | Whether payment was captured.
payments[].refunded | boolean | Whether payment was refunded.
payments[].created | integer | Charge date. Unix timestamp in milliseconds.
payments[].created_date | string | Charge date. ISO 8601 timestamp.
payments[].card.name | string | Stripe only.
payments[].card.last4 | string | Stripe only.
payments[].card.exp_month | integer | Stripe only.
payments[].card.exp_year | integer | Stripe only.
payments[].card.type | string | Stripe only.
payments[].card.country | string | Stripe only.
**Adjustments** | [objects] |
adjustments[].id | string | Adjustment identifier.
adjustments[].type | string | Possible values: `flat`.
adjustments[].reason | string | Reasoning for price adjustment.
adjustments[].issuer | string | Authorizer of price adjustment.
adjustments[].amount | number | Amount of price adjustment.
discount_codes | [strings]| Array of discount codes used.
**Discounts** | [objects] |
discounts[].code | string | Discount code applied.
discounts[].type: | string | Discount type. Possible values: `flat`, `percent`.
discounts[].amount | integer | Discount amount (5% off means 500 and $10 off means 1000).
discounts[].product_id | string | Product id that discount belongs to.
discounts[].apply | string | Possible values: `once`, `every_time`.
**History** | [objects] |
history[].type | string | [See events](#webhooks).
history[].body | string | Brief description of history type.
history[].created | integer | Unix timestamp in milliseconds.
history[].created_date | string | ISO 8601 timestamp.

### Checkout with Credit or Debit Card

This is a public endpoint to create a new order with a credit or debit card. It does not require authentication.

For cross-browser compatibility with older versions of Internet Explorer, we recommend using [jQuery-ajaxTransport-XDomainRequest](https://github.com/MoonScript/jQuery-ajaxTransport-XDomainRequest).

To place an order with a credit/debit card in the sandbox environment, please use [Stripe test cards](https://stripe.com/docs/testing).

```
POST /v2/orders/checkout
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
user_id | string | **Required**. Your user id.
currency | string | 3-letter ISO currency code (lowercase). Possible values: `usd`, `cad`, `gbp`, `eur`, `aud`.
**Buyer**|object|
buyer.email | string | **Required**. Buyer's email address (used for Celery's email notifications).
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.company | string | Buyer's company.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address** |object|
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.name | string | Shipping address full name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shipping address building, apartment, unit, etc.
shipping_address.city | string | Shipping address city.
shipping_address.state | string | Shipping address state, province, or region. If country US or CA, use 2-letter ISO state code (lowercase).
shipping_address.zip | string | Shipping address ZIP or postal code.
shipping_address.country | string | Shipping address 2-letter ISO country code (lowercase).
shipping_address.phone | string | Shipping address phone number.
**Billing Address** |object|
billing_address.first_name | string | Billing address first name.
billing_address.last_name | string | Billing address last name.
billing_address.line1 | string | Billing address street address.
billing_address.line2 | string | Billing address building, apartment, unit, etc.
billing_address.city | string | Billing address city.
billing_address.state | string | Billing address state, province, or region. If country US or CA, use 2-letter ISO state code (lowercase).
billing_address.zip | string | Billing address ZIP or postal code.
billing_address.country | string | Billing address 2-letter ISO country code (lowercase).
billing_address.phone | string | Billing address phone number.
**Line Items** | [objects] |
line_items[].product_id | string | **Required**. Product id.
line_items[].variant_id | string | Variant id (required if product has variants).
line_items[].quantity | integer | Number of units ordered.
**Payment Source** | object |
payment_source.card.name | string | Name on credit/debit card. Stripe only.
payment_source.card.number | string | Credit/debit card number. Stripe only.
payment_source.card.exp_month | integer | **Required**. Expiration month of credit/debit card. Stripe only.
payment_source.card.exp_year | integer | **Required**. Expiration year of credit/debit card. Stripe only.
payment_source.card.cvc | string | **Required**. Credit/debit card CVC. Stripe only.
discount_codes | [strings] | List of coupon codes to be applied to order.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H \
https://api.trycelery.com/v2/orders/checkout \
-d'
{
    "user_id": "514a114080feb60200000001",
    "buyer": {
        "email": "first@last.com",
        "first_name": "First",
        "last_name": "Last",
        "phone": "555-555-5555"
    },
    "shipping_address": {
        "first_name": "First",
        "last_name": "Last",
        "company": "Celery",
        "line1": "123 Street",
        "line2": null,
        "city": "New York",
        "state": "ny",
        "zip": "10012",
        "country": "us",
        "phone": "555-555-5555"
    },
    "billing_address": {
        "first_name": "First",
        "last_name": "Last",
        "line1": "123 Main Street",
        "line2": null,
        "city": "San Francisco",
        "state": "ca",
        "zip": "94105",
        "country": "us",
        "phone": "555-555-5555"
    },
    "line_items": [
        {
            "product_id": "531e0b012cf9766885f781b7",
            "variant_id": "",
            "quantity": 1
        }
    ],
    "payment_source": {
        "card": {
            "name": "First Last",
            "number": "4242 4242 4242 4242",
            "exp_month": 12,
            "exp_year": 2020,
            "cvc": "123"
        }
    },
    "discount_codes": [
        "10dollarsoff"
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
        "_id": "544c220151feb60200000002",
        "user_id": "514a114080feb60200000001",
        "order_status": "open",
        "payment_status": "unpaid",
        "fulfillment_status": "unfulfilled",
        ...,
        "history": [
            {
                "type": "order.created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            }
        ],
        ...
    }
}
```

### Checkout with PayPal

This is a public endpoint to create a new order with PayPal. It does not require authentication. Celery uses [PayPal Adaptive Payments](https://developer.paypal.com/docs/classic/products/adaptive-payments/).

For cross-browser compatibility with older versions of Internet Explorer, we recommend using [jQuery-ajaxTransport-XDomainRequest](https://github.com/MoonScript/jQuery-ajaxTransport-XDomainRequest).

The response will include a URL that the buyer needs to be redirected to initiate the PayPal payment flow. This URL can be found found in `data.payment_source.paypal.redirect_url`. After the buyer completes the PayPal payments flow, they will be redirected back to return URL provided. Please note that the Celery appends the order number to the return URL. Thus, if you set the return URL to be `https://yourshop.com/confirmation`, then the buyer will be redirected to `https://yourshop.com/confirmation/101340827`. If the buyer cancels during the PayPal payment flow, then they will be redirected to the cancel URL.

```
POST /v2/orders/checkout/paypal
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
user_id | string | **Required**. Your user id.
cancel_url | string | **Required.** URL that buyer will go to if they cancel order.
return_url | string | **Required.** URL that buyer will go to after they complete PayPal payment flow.
currency | string | 3-letter ISO currency code (lowercase). Possible values: `usd`, `cad`, `gbp`, `eur`, `aud`.
**Buyer**|object|
buyer.email | string | **Required**. Buyer's email address (used for Celery's email notifications).
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.company | string | Buyer's company.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address** |object|
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shipping address building, apartment, unit, etc.
shipping_address.city | string | Shipping address city.
shipping_address.state | string | Shipping address state, province, or region. If country US or CA, use 2-letter ISO state code (lowercase).
shipping_address.zip | string | Shipping address ZIP or postal code.
shipping_address.country | string | Shipping address 2-letter ISO country code (lowercase).
shipping_address.phone | string | Shipping address phone number.
**Line Items** | [objects] |
line_items[].product_id | string | **Required**. Product id.
line_items[].variant_id | string | Variant id (required if product has variants).
line_items[].quantity | integer | Number of units ordered.
discount_codes | [strings] | List of coupon codes to be applied to order.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H \
https://api.trycelery.com/v2/orders/checkout/paypal \
-d'
{
    "user_id": "514a114080feb60200000001",
    "cancel_url": "https://www.yourshop.com",
    "return_url": "https://www.yourshop.com/confirmation",
    "buyer": {
        "email": "first@last.com",
        "first_name": "First",
        "last_name": "Last",
        "phone": "555-555-5555"
    },
    "shipping_address": {
        "first_name": "First",
        "last_name": "Last",
        "company": "Celery",
        "line1": "123 Street",
        "line2": null,
        "city": "New York",
        "state": "ny",
        "zip": "10012",
        "country": "us",
        "phone": "555-555-5555"
    },
    "line_items": [
        {
            "product_id": "531e0b012cf9766885f781b7",
            "quantity": 1
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
        "_id": null,
        "user_id": "514a114080feb60200000001",
        "number": "101340827",
        "order_status": "open",
        "payment_status": "unpaid",
        "fulfillment_status": "unfulfilled",
        "type": "paypal_adaptive",
        "cancel_url": "https://www.yourshop.com/101340827",
        "return_url": "https://www.yourshop.com/confirmation/101340827",
        "payment_source": {
            "paypal": {
                "redirect_url": "https://www.sandbox.paypal.com/cgi-bin/webscr?cmd=_ap-preapproval&preapprovalkey=PA-3KC050249N5937812",
                "preapproval_key": "PA-3KC050249N5937812"
            }
        },
        ...
    }
}
```


### Retrieve an Order

```
GET /v2/orders/{id}
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Order unique identifier.

##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
offset | string | Default is `0`.
page | integer | Current page number.
limit | string | A limit on the number of objects to be returned. Default is `100`.
created | integer | A filter on the list based on object `created` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
updated | integer | A filter on the list based on object `updated` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
order_status | string | Possible values: `open`, `completed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `failed`, `refunded`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `procesing`, `failed`.
buyer.name | string | A filter on the list based on buyer name. This filter will perform a regex on the value.
buyer.email | string | A filter on the list based on buyer email. This filter will perform a regex on the value.
line_items.product_id | string | A filter on the list based on product id.
line_items.variant_id | string | A filter on the list based on product variant id.
line_items.sku | string | A filter on the list based on sku.
type | string | A filter on order type: `card`, `paypal_adaptive`, `paypal_adaptive_chained`, `affirm`.
shipping_method | string | A filter on shipping method.
discounts.code | string | A filter on discount code.
currency | string | A filter on currency code.

- gt:  Return values where the relevant field is after this timestamp (in ms).
- gte: Return values where the relevant field is after or equal to this timestamp (in ms).
- lt:  Return values where the relevant field is before this timestamp (in ms).
- lte: Return values where the relevant field is before or equal to this timestamp (in ms).

If I wanted to query for orders created at or after January 1, 2014 12:00 AM UTC, my query string would include `created[gte]=1388534400000`.

If I wanted to query for orders purchased by `Brian Nguyen`, my query string would include `buyer.name=Brian+Nguyen`.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
            "page": 1,
            "pages": 1,
            "limit": 100,
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

### Count orders

```
GET /v2/orders/count
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
created | integer | A filter on the list based on object `created` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
updated | integer | A filter on the list based on object `updated` field. The value can be a string with an integer timestamp (in ms), or it can be a dictionary with the following options: `gt`, `gte`, `lt`, `lte`.
order_status | string | Possible values: `open`, `completed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `failed`, `refunded`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `procesing`, `failed`.
buyer.name | string | A filter on the list based on buyer name. This filter will perform a regex on the value.
buyer.email | string | A filter on the list based on buyer email. This filter will perform a regex on the value.
line_items.product_id | string | A filter on the list based on product id.
line_items.variant_id | string | A filter on the list based on product variant id.
line_items.sku | string | A filter on the list based on sku.
type | string | A filter on order type: `card`, `paypal_adaptive`, `paypal_adaptive_chained`, `affirm`.
shipping_method | string | A filter on shipping method.
discounts.code | string | A filter on discount code.
currency | string | A filter on currency code.

- gt:  Return values where the relevant field is after this timestamp (in ms).
- gte: Return values where the relevant field is after or equal to this timestamp (in ms).
- lt:  Return values where the relevant field is before this timestamp (in ms).
- lte: Return values where the relevant field is before or equal to this timestamp (in ms).

If I wanted to count how many orders were created at or after January 1, 2014 12:00 AM UTC, my query string would include `created[gte]=1388534400000`.

If I wanted to count how many orders were purchased by `Brian Nguyen`, my query string would include `buyer.name=Brian+Nguyen`.

If I wanted to count how many orders include the product with id `531e0b012cf9766885f781b7`, my query string would be include `line_items.product_id=531e0b012cf9766885f781b7`.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/orders/count?created[gte]=1388534400000
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "total": 507
    }
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
_id | string | **Required**. Order unique identifier.
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
shipping_address.line2 | string | Shipping address building, apartment, unit, etc.
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
$ curl -X PUT -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
$ curl -X PUT -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
id | string | **Required**. Order unique identifier.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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


### Charge an Order

This will charge payment for the order. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/payment_charge
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Order unique identifier.

##### Example Request

```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
                "body": "Your order was charged."
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
id | string | **Required**. Order unique identifier.

##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
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
                "body": "Your order was charged."
            },
            {
                "type": "order.refund.succeeded",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was refunded."
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
id | string | **Required**. Order unique identifier.
courier | string | Courier that will fulfill order. Possible values: `ups`, `usps`, `fedex`, `dhl`, `other`.
number | string | Tracking number.
serial_numbers | [string] | Serial numbers associated with the order.

##### Example Request

```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/fulfillment_succeed \
-d'
{
    "courier" : "ups",
    "number" : "1zasdfajkfsdljasdf",
    "serial_numbers": [
        "abc123"
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
        "fulfillment_status": "fulfilled",
        "fulfillment": {
            "created": 1401993491000,
            "created_date": "2014-06-05T06:38:00.000Z",
            "courier": "ups",
            "number": "1zasdfajkfsdljasdf",
            "serial_numbers": [
                "abc123"
            ]
        },
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

## Products Resource

Attributes | Type | Description
-----------|------|------------
_id | value | Product unique identifier.
slug | string | Seller-defined identifier for hosted shop page.
name | string | Product name.
description | string | Description of Product and supports `HTML` markup.
sku | integer | Base product sku.
price | integer | Base product price.
weight | float | Default value is `0`.
published | boolean | Defines whether a product is shown.
created | integer | Product created date. Unix timestamp in milliseconds.
updated | integer | Product updated date. Unix timestamp in milliseconds.
created_date | string | Product created date. ISO 8601 timestamp.
updated_date | string | Product updated date. ISO 8601 timestamp.
**Flags** | object |
flags.enable_taxes | boolean | Whether to collect tax.
flags.enable_options | boolean | Whether product has variants.
**Images** | object |
images[].url | string | Product image URL.
**Options** | objects |
options[].id | string | Option id.
options[].name | string | Option group name (e.g., size)
options[].values | object | Option group values (e.g., small, medium, large)
**Variants** | object |
variants[].id | string | Variant id. Used to checkout products with variants.
variants[].name | string | Variant name.
variants[].sku | string | Variant sku.
variants[].price | integer | Variant price.
variants[].weight | integer | Variant weight.
variants[].options | object | Variant settings.

### Retrieve a list of products

```
GET /v2/products
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
sort | string | Sort by. Default is `created`.
order | string | Order by. Default is `desc`. Possible values: `desc`, `asc`.
offset | string | Default is `0`.
page | integer | Current page number.
limit | string | A limit on the number of objects to be returned. Default is `100`.


##### Example Request

```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/products
```

##### Example Response

```json
{
    "meta": {
        "code": 200,
        "paging": {
            "total": 3,
            "count": 3,
            "limit": 100,
            "offset": 0,
            "page": 1,
            "pages": 1,
            "has_more": false
        }
    },
    "data": [
        {
            "_id": "546aab7b1657500600a781bc",
            "slug": "90ca88bd-3830-4ee0-b39f-e5f200d0a489",
            "name": "Koala 1",
            "description": "<p>Koala party</p>",
            "sku": null,
            "price": 1500,
            "deposit": 0,
            "weight": 0,
            "inventory": 0,
            "published": true,
            "flags": {
                "enable_taxes": true,
                "enable_options": false,
                "enable_inventory": false
            },
            "images": [],
            "options": [],
            "variants": [],
            "user_id": "54629b565f388707003de6ea",
            "version": "v2",
            "created": 1416276859872,
            "updated": 1416276859970,
            "created_date": "2014-11-18T02:14:19.872Z",
            "updated_date": "2014-11-18T02:14:19.970Z",
            "locked": false,
            "metadata": {}
        },
        {
            "_id": "546a9d731657500600a781ba",
            "slug": "12b1f0a1-b18c-407a-9f9c-470f262f1853",
            "name": "Koala 2",
            "description": null,
            "sku": null,
            "price": 2000,
            "deposit": 0,
            "weight": 0,
            "inventory": 0,
            "published": true,
            "flags": {
                "enable_taxes": true,
                "enable_options": false,
                "enable_inventory": false
            },
            "images": [],
            "options": [],
            "variants": [],
            "user_id": "54629b565f388707003de6ea",
            "version": "v2",
            "created": 1416273267130,
            "updated": 1416273287831,
            "created_date": "2014-11-18T01:14:27.130Z",
            "updated_date": "2014-11-18T01:14:47.831Z",
            "locked": false,
            "metadata": {}
        },
        {
            "_id": "546540f2793b4c050015d96c",
            "slug": "7dc66c4a-99ca-48b2-b584-23de515272c0",
            "name": "Koala 3",
            "description": null,
            "sku": null,
            "price": 2500,
            "deposit": 0,
            "weight": 0,
            "inventory": 0,
            "published": true,
            "flags": {
                "enable_taxes": true,
                "enable_options": false,
                "enable_inventory": false
            },
            "images": [],
            "options": [],
            "variants": [],
            "user_id": "54629b565f388707003de6ea",
            "version": "v2",
            "created": 1415921906675,
            "updated": 1415922135705,
            "created_date": "2014-11-13T23:38:26.675Z",
            "updated_date": "2014-11-13T23:42:15.705Z",
            "locked": false,
            "metadata": {}
        }
    ]
}
```

### Retrieve a Product

```
GET /v2/products/{id}
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Product unique identifier.


##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/products/546540f2793b4c050015d96c
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "slug": "7dc66c4a-99ca-48b2-b584-23de515272c0",
        "name": "Koala Bears",
        "description": "<p>Cute but dangerous.</p>",
        "sku": null,
        "price": 1000,
        "deposit": 0,
        "weight": 0,
        "inventory": 0,
        "published": true,
        "flags": {
            "enable_taxes": true,
            "enable_options": false,
            "enable_inventory": false
        },
        "images": [],
        "options": [],
        "variants": [],
        "_id": "546540f2793b4c050015d96c",
        "user_id": "54629b565f388707003de6ea",
        "version": "v2",
        "created": 1415921906675,
        "updated": 1415922135705,
        "created_date": "2014-11-13T23:38:26.675Z",
        "updated_date": "2014-11-13T23:42:15.705Z",
        "locked": false,
        "metadata": {}
    }
}
```

## Collections Resource

Attributes | Type | Description
-----------|------|------------
_id | string | **Required**. Collections unique identifier.
slug | string | Seller-defined identifier for hosted shop page.
name | string | Collection name.
published | boolean | Defines whether collection is shown.
product_ids | [objects] | List of product ids in collection.
user_id | string | Seller unique identifier.
created | integer | Collection created date. Unix timestamp in milliseconds.
updated | integer | Collection updated date. Unix timestamp in milliseconds.
created_date | string | Collection created date. ISO 8601 timestamp.
updated_date | string | Collection updated date. ISO 8601 timestamp.
products | [objects] | List of products in collection.


### Retrieve a list of Collections

```
GET /v2/collections/
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
sort | string | Sort by. Default is `created`.
order | string | Order by. Default is `desc`. Possible values: `desc`, `asc`.
offset | string | Default is `0`.
page | integer | Current page number.
limit | string | A limit on the number of objects to be returned. Default is `100`.


##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/collections
```

##### Example Response

```json
{
    "meta": {
        "code": 200,
        "paging": {
            "total": 1,
            "count": 1,
            "limit": 100,
            "offset": 0,
            "page": 1,
            "pages": 1,
            "has_more": false
        }
    },
    "data": {
        "slug": "53171e1a-a272-4a73-9f91-214159672197",
        "name": "Christmas Koala Collection",
        "published": true,
        "product_ids": [
            "546540f2793b4c050015d96c",
            "54653d6e793b4c050015d96a",
            "54653d42793b4c050015d969"
        ],
        "_id": "54736eb0b588d10400fbe7cc",
        "user_id": "54629b565f388707003de6ea",
        "version": "v2",
        "created": 1416851120197,
        "updated": 1416851141330,
        "created_date": "2014-11-24T17:45:20.197Z",
        "updated_date": "2014-11-24T17:45:41.330Z",
        "locked": false,
        "metadata": {},
        "products": [
            {
                "_id": "546540f2793b4c050015d96c",
                "slug": "7dc66c4a-99ca-48b2-b584-23de515272c0",
                "name": "Koala Two-Pack",
                "description": "<p>A mommy koala with baby koala packed inside.</p>",
                "sku": null,
                "price": 1000,
                "deposit": 0,
                "weight": 0,
                "inventory": 0,
                "published": true,
                "flags": {
                    "enable_taxes": true,
                    "enable_options": false,
                    "enable_inventory": false
                },
                "images": [],
                "options": [],
                "variants": [],
                "user_id": "54629b565f388707003de6ea",
                "version": "v2",
                "created": 1415921906675,
                "updated": 1415922135705,
                "created_date": "2014-11-13T23:38:26.675Z",
                "updated_date": "2014-11-13T23:42:15.705Z",
                "locked": false,
                "metadata": {}
            },
            ...
        ]
    }
}
```

### Retrieve a Collection

```
GET /v2/collections/{id}
```

##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/collections/{id}
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required** Collection unique identifier.


##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "slug": "53171e1a-a272-4a73-9f91-214159672197",
        "name": "Christmas Koala Collection",
        "published": true,
        "product_ids": [
            "546540f2793b4c050015d96c",
            "54653d6e793b4c050015d96a",
            "54653d42793b4c050015d969"
        ],
        "_id": "54736eb0b588d10400fbe7cc",
        "user_id": "54629b565f388707003de6ea",
        "version": "v2",
        "created": 1416851120197,
        "updated": 1416851141330,
        "created_date": "2014-11-24T17:45:20.197Z",
        "updated_date": "2014-11-24T17:45:41.330Z",
        "locked": false,
        "metadata": {},
        "products": [
            {
                ...
            },
        ]
    }
}
```


## Coupons Resource

Attributes | Type | Description
-----------|------|------------
_id | string | Coupon unique identifier.
user_id | string | Seller unique identifier.
type | string | Possible values: `flat`, `percent`.
code | string | Coupon code (must be unique).
filter | string | Possible values: `order`, `order_with_minimum`, `product`.
apply | string | Possible values: `once`, `every_item`.
free_shipping | boolean | Possible values: `true`, `false`.
product_id | string | Product id for product-specific coupons.
enabled | boolean | Whether coupon is valid.
amount | integer | Coupon amount. $5 should be `500` (prices in cents). 10% should be `10`.
order_minimum | integer | Minimum line item amount before coupon can be applied (amount in cents). Filter must be set to `order_with_minimum`.
quantity | integer | How many times coupon can be redeemed. (`null` or `undefined` represents unlimited usage)
times_used | integer | Number of times coupon was redeemed.
begins | integer | Coupon begin date. Unix timestamp in milliseconds.
begins_date | string | Coupon begin date. ISO 8601 timestamp.
expires | integer | Coupon expiration date. Unix timestamp in milliseconds.
expires_date | string | Coupon expiration date. ISO 8601 timestamp.
used_emails | [string] | List of buyer email addresses who have redeemed coupon.
created | integer | Unix timestamp in milliseconds.
created_date | string | ISO 8601 timestamp.
updated | integer | Unix timestamp in milliseconds.
updated_date | string | ISO 8601 timestamp.


### Create a Coupon

```
POST /v2/coupons
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
type | string | **Required** Possible values: `flat`, `percent`.
code | string | **Required** Coupon code (must be unique).
filter | string | **Required** Possible values: `order`, order_with_minimum, `product`.
apply | string | **Required** Possible values: `once`, `every_item`.
product_id | string | Product id for product-specific coupons.
enabled | boolean | Whether coupon is valid. Defaults to `true`.
free_shipping | boolean | **Required** Possible values: `true`, `false`.
amount | integer | **Required** Coupon amount. $5 should be `500` (prices in cents). 10% should be `10`.
order_minimum | integer | Minimum line item amount before coupon can be applied (amount in cents). Filter must be set to `order_with_minimum`.
quantity | integer | How many times coupon can be redeemed. (`null` or `undefined` represents unlimited usage)
begins | integer | Coupon begin date. Unix timestamp in milliseconds.
expires | integer | Coupon expiration date. Unix timestamp in milliseconds.


##### Example Request
```
$ curl -X POST -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/coupons
-d'
{
    "type": "flat",
    "code": "15off-1",
    "filter": "flat",
    "free_shipping": false,
    "amount": 1500,
    "expired": 1461705000000
}
```

##### Example Response

```json
{
    "meta": {
        "code": 200,
    },
    "data": [
        {
          "_id": "571fd4b900598e208992eed8",
          "version": "v2",
          "created": 1461703865552,
          "updated": 1461703865556,
          "created_date": "2016-04-26T20:51:05.552Z",
          "updated_date": "2016-04-26T20:51:05.556Z",
          "locked": false,
          "metadata": {},
          "user_id": "514a114080feb60200000001",
          "type": "flat",
          "code": "15off-1",
          "filter": "order",
          "apply": "once",
          "product_id": null,
          "enabled": true,
          "free_shipping": false,
          "amount": 1500,
          "quantity": null,
          "times_used": 0,
          "order_minimum": 0,
          "begins": 1461703865552,
          "begins_date": "2016-04-26T20:51:05.552Z",
          "expires": 1461705000000,
          "expires_date": null,
          "used_emails": []
        }
    ]
}
```


### Retrieve a List of Coupons

```
GET /v2/coupons
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
sort | string | Sort by. Default is `created`.
order | string | Order by. Default is `desc`. Possible values: `desc`, `asc`.
offset | string | Default is `0`.
page | integer | Current page number.
limit | string | A limit on the number of objects to be returned. Default is `100`.

##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/coupons
```

##### Example Response

```json
{
    "meta": {
        "code": 200,
        "paging": {
            "total": 3,
            "count": 3,
            "limit": 100,
            "offset": 0,
            "page": 1,
            "pages": 1,
            "has_more": false
        }
    },
    "data": [
        {
            "_id": "5488e5689b43260400cb3d1a",
            "type": "percent",
            "code": "566e8816d360",
            "filter": "order",
            "apply": "once",
            "free_shipping": false,
            "product_id": null,
            "enabled": true,
            "amount": 20,
            "quantity": null,
            "times_used": 0,
            "order_minimum": 0,
            "begins": 1418198400000,
            "begins_date": "2014-12-11T00:29:28.108Z",
            "expires": null,
            "expires_date": null,
            "used_emails": [],
            "user_id": "54629b565f388707003de6ea",
            "version": "v2",
            "created": 1418257768108,
            "updated": 1418257768108,
            "created_date": "2014-12-11T00:29:28.108Z",
            "updated_date": "2014-12-11T00:29:28.108Z",
            "locked": false,
            "metadata": {}
        },
        {
            "_id": "5488e559fe3e2a0500b82556",
            "type": "flat",
            "code": "94cf61c4d205",
            "filter": "order",
            "apply": "once",
            "free_shipping": false,
            "product_id": null,
            "enabled": true,
            "amount": 2000,
            "quantity": null,
            "times_used": 0,
            "order_minimum": 0,
            "begins": 1418198400000,
            "begins_date": "2014-12-11T00:29:13.319Z",
            "expires": null,
            "expires_date": null,
            "used_emails": [],
            "user_id": "54629b565f388707003de6ea",
            "version": "v2",
            "created": 1418257753319,
            "updated": 1418257753319,
            "created_date": "2014-12-11T00:29:13.319Z",
            "updated_date": "2014-12-11T00:29:13.319Z",
            "locked": false,
            "metadata": {}
        }
    ]
}
```


### Retrieve a Coupon

```
GET /v2/coupons/{id}
```

##### Example Request
```
$ curl -X GET -H Content-Type:application/json -H Authorization:ACCESS_TOKEN \
https://api.trycelery.com/v2/coupons/{id}
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required** Coupon unique identifier.


##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "type": "flat",
        "code": "93000fdb71fc",
        "filter": "order",
        "apply": "once",
        "free_shipping": true,
        "product_id": null,
        "enabled": true,
        "amount": 0,
        "quantity": null,
        "times_used": 0,
        "order_minimum": 0,
        "begins": 1418198400000,
        "begins_date": "2014-12-11T00:29:35.785Z",
        "expires": null,
        "expires_date": null,
        "used_emails": [],
        "_id": "5488e56ffe3e2a0500b82557",
        "user_id": "54629b565f388707003de6ea",
        "version": "v2",
        "created": 1418257775785,
        "updated": 1418257775785,
        "created_date": "2014-12-11T00:29:35.785Z",
        "updated_date": "2014-12-11T00:29:35.785Z",
        "locked": false,
        "metadata": {}
    }
}
```


### Validate coupon code

This public endpoint returns whether a coupon code is valid. If the coupon is not valid, then the endpoint will respond with an error. This endpoint does not require authentication.

```
POST /v2/coupons/validate
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
user_id | string | **Required**. Seller unique identifier.
code | string | **Required**. Coupon code to validate.
**Line Items** | [object] |
line_items[].product_id | string | Product id to validate against product-specific coupons.


##### Example Request
```
$ curl -X POST -H Content-Type:application/json \
https://api.trycelery.com/v2/coupons/validate
-d'
{
    "user_id": "514a114080feb60200000001",
    "code": "036dabee2bd3",
    "line_items": [
        {
            "product_id": null
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
        "type": "flat",
        "code": "036dabee2bd3",
        "filter": "order",
        "apply": "once",
        "free_shipping": false,
        "product_id": null,
        "enabled": true,
        "amount": 10000,
        "quantity": 0,
        "times_used": 1,
        "order_minimum": 0,
        "begins": 1412146800000,
        "begins_date": "2014-08-08T21:52:37.055Z",
        "expires": 1413010800000,
        "expires_date": null,
        "used_emails": [],
        "_id": "53e546a5fc08e40400631da9",
        "user_id": "5330bc8c5ade8207002df9f6",
        "version": "v2",
        "created": 1407534757055,
        "updated": 1407534757055,
        "created_date": "2014-08-08T21:52:37.055Z",
        "updated_date": "2014-08-08T21:52:37.055Z",
        "locked": false,
        "metadata": {}
    }
}
```

##### Error Example Response

```json
{
    "meta": {
        "code": 400,
        "error": {
            "message": "Coupon code is expired.",
            "code": 400
        }
    },
    "data": "Coupon code is expired."
}
```

## Users Resource

### Retrieve tax rate

This public endpoint returns the sales tax rate based on the seller's tax rules and the shipping address provided. This endpoint does not require authentication.

```
GET /v2/users/{id}/tax_rates
```

##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Seller unique identifier.
shipping_country | string | **Required**. 2-letter ISO country code (lowercase).
shipping_state | string | 2-letter ISO state code if United States or Canada (lowercase).
shipping_zip | string | 5-digit zip code if United States. Supports optional 4 digits.

##### Example Request
```
$ curl -X GET -H Content-Type:application/json \
https://api.trycelery.com/v2/users/530ec58358b6ee0000f5d440/tax_rates?shipping_country=us&shipping_state=ca&shipping_zip=94105
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "base": 0.0875
    }
}
```


## Webhooks

Events are our way of letting you know about something interesting has happened with your order. The Celery API can send events directly to your server using webhooks. Please contact Celery if you would like to implement webhooks.

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
        "_id": "530ec58358b6ee0000f5d440",
        "order_status": "open",
        "payment_status": "unpaid",
        "fulfillment_status": "unfulfilled",
        ...
    }
}
