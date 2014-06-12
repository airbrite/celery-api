# Celery API

The Celery API is an ecommerce logic and storage engine designed to be an essential tool for any developer.

The API is currently in beta. Please contact [support@trycelery.com](mailto:support@trycelery.com) if you intend to deploy our API in production code.

Our API is organized around REST and designed to have predictable, resource-oriented URLs, and to use HTTP response codes to indicate API errors. We support cross-origin resource sharing to allow you to interact securely with our API from a client-side web application (though you should remember that you should never expose your secret API key in any public website's client-side code). JSON will be returned in all responses from the API, including errors.


## Menu
* [Getting Started](#getting-started)
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

## Getting Started

The base endpoint URL is `https://api.trycelery.com`. Celery is currently on version `v2`, so a request to retrieve all orders would resemble:

```
GET https://api.trycelery.com/v2/orders
```

**NOTE**: All POST and PUT requests should be sent with the following header `Content-Type="application/json"`.

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
        "code": 400,
        "error_message": "User with email already exists.",
        "error_type": "client_error"
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
id | string | Unique identifier for the order
user_id | string | Seller unique identifier.
order_status | string | Possible values: `open`, `closed`, `cancelled`.
payment_status | string | Possible values: `unpaid`, `paid`, `refunded`, `failed`.
fulfillment_status | string | Possible values: `unfulfilled`, `fulfilled`, `ready_to_fulfill`, `failed`.
currency | string | 3-letter ISO currency code (lowercase).
type | string | Possible values: `card`, `paypal_adaptive`, `paypal_adaptive_chained`, `affirm`
number | string | Human-readable order number.
discount | integer | Discount amount applied to the order. Amount in cents.
subtotal | integer | Sum of line item amounts less discount. Amount in cents.
shipping | integer | Shipping cost applied. Amount in cents.
taxes | integer | Sales tax applied. Amount in cents.
adjustments | integer | Price adjustments applied. Amount in cents.
total | integer | Total = subtotal + shipping + taxes + adjustments. Amount in cents.
balance | number | Amount owed to the seller. Amount in cents.
paid | number | Amount paid to the seller. Amount in cents.
refunded | number | Amount refunded by the seller. Amount in cents.
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
**Shipping Address**|object|
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
fulfillment.created | string | Date marked fulfilled (Unix timestamp in milliseconds).
fulfillment.created_date | string | Date marked fulfilled (ISO 8601 timestamp).
fulfillment.courier | string | Courier used (e.g., `ups`, `usps`, `fedex`, `dhl`, `other`).
fulfillment.number | string | Tracking number.
**Line Items**|[objects]|
line_items[].id | string | Line item identifier (uuid).
line_items[].product_name | string | Product name.
line_items[].variant_name | string | Variant name.
line_items[].price | integer | Line_item unit price.
line_items[].quantity | integer | Number of units ordered.
line_items[].weight | integer | Line item unit weight.
line_items[].enable_taxes | boolean | Whether taxes apply to line_item.
**Payments**|[objects]|
payments[].id | string | Payment identifier.
payments[].amount | integer | Amount capture.
payments[].amount_refunded | integer | Amount refunded.
payments[].refunded | boolean | Whether payment has been refunded.
payments[].charge_id | string | Gateway's identifier for charge.
payments[].gateway | string | Possible values: `stripe`, `papyal`, `affirm`.
payments[].refunds | [objects] | Listing of refunds for this charge.
payments[].created | integer | Charge date. Unix timestamp in milliseconds.
payments[].created_date | string | Charge date. ISO 8601 timestamp.
**Adjustments**|[objects]|
adjustments[].id | string | Adjustment identifier.
adjustments[].type | string | Possible values: `flat`.
adjustments[].reason | string | Reasoning for price adjustment.
adjustments[].issuer | string | Authorizer of price adjustment.
adjustments[].amount | number | Amount of price adjustment.
**Discounts**|[objects]|
discounts[].code | string | Discount code applied.
discounts[].type: | string | Discount type. Possible values: `flat`, `percent`.
discounts[].discount | number | Discount amount (5% off means 500 and $10 off means 1000).
discounts[].product_id | string | Product ID that discount belongs to.
discounts[].apply | string | Possible values: `once`, `every_time`.
**History**|[objects]|
history[].type | string | Possible values: `order_created`, `order_cancelled`, `charge_success` `charge_failed`, `refund_success`, `refund_failed`, `fulfillment_success`.
history[].body | string | Description of history type.
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
$ curl -X POST -H Content-Type:application/json -H Authorization:{{ACCESS_TOKEN}} \
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440
```

##### Example Response

```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "created": 1393476995707,
        "updated": 1393481805634,
        "created_date": "2014-02-27T04:56:35.707Z",
        "updated_date": "2014-02-27T06:16:45.634Z",
        "user_id": "514a114080feb60200000001",
        "version": "v2",
        "_id": "530ec58358b6ee0000f5d440",
        "number": "101211405",
        "locked": false,
        "order_status": "open",
        "payment_status": "unpaid",
        "fulfillment_status": "unfulfilled",
        "currency": "usd",
        "notes": "",
        "shipping": 0,
        "taxes": 0,
        "discount": 0,
        "buyer": {
            "first_name": "Brian",
            "last_name": "Nguyen",
            "name": "Brian Nguyen",
            "sort_name": "brian nguyen",
            "email": "bnguyen06@gmail.com",
            "notes": "I'm excited for this product!"
        },
        "shipping_address": {
            "first_name": "Brian",
            "last_name": "Nguyen",
            "name": "Brian Nguyen",
            "company": "Celery",
            "line1": "123 Main Street",
            "line2": "Unit 101",
            "city": "San Francisco",
            "state": "ca",
            "zip": "94107",
            "country": "us",
            "phone": "555-555-5555"
        },
        "most_recent_card": {
            "exp_month": "2",
            "exp_year": "2014",
            "last4": "0341",
            "type": "visa"
        },
        "line_items":
        [
            {
                "id": "550e8400-e29b-41d4-a716-446655440000",
                "product_id": "530e40d428ee4100002bfa78",
                "quantity": 1,
                "price": 1000,
                "weight": 0,
                "enable_taxes": true,
                "product_name": "Koala Bears",
                "variant_name": null,
                "celery_sku": "530e40d428ee4100002bfa78",
                "sku": "KOALA-1"
            }
        ],
        "adjustments": [],
        "payments": [],
        "fulfillments": [],
        "discount_codes": [],
        "history": 
        [
            {
                "type": "order_created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            }
        ],
        "balance": 1000,
        "subtotal": 1000,
        "total": 1000,
        "paid": 0,
        "refunded": 0
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

- gt:  Return values where the relevent field is after this timestamp (in ms).
- gte: Return values where the relevant field is after or equal to this timestamp (in ms).
- lt:  Return values where the relevant field is before this timestamp (in ms).
- lte: Return values where the relevant field is before or equal to this timestamp (in ms).

Thus, if I wanted to query for orders created at or after January 1, 2014 12:00 AM UTC, my query string would include `created[gte]=1388534400000`.

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
    "data":
    [
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

Warning! Modifying data that is not `buyer` or `shipping_address` may have unintended effects on the order. If you are modifying these objects, please be sure to include entire object.

```
PUT /v2/orders/{id}
```
##### Arguments

Attributes | Type | Description
-----------|------|------------
id | string | **Required**. Unique identifier for the order.
**Buyer**|object|
buyer.email | string | Buyer's email address.
buyer.first_name | string | Buyer's first name.
buyer.last_name | string | Buyer's last name.
buyer.phone | string | Buyer's phone number.
buyer.notes | string | Buyer notes to the seller.
**Shipping Address**|object|
shipping_address.first_name | string | Shipping address first name.
shipping_address.last_name | string | Shipping address last name.
shipping_address.company | string | Shipping address company name.
shipping_address.line1 | string | Shipping address street address.
shipping_address.line2 | string | Shiping address building, apartment, unit, etc.
shipping_address.city | string | Shipping address city.
shipping_address.state | string | Shipping address state, province, or region. US or CA states should be lowercase.
shipping_address.zip | string | Shipping address ZIP or postal code.
shipping_address.country | string | Shipping address 2-letter ISO country code (lowercase).
shipping_address.phone | string | Shipping address phone number.

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

This example request updates the quantity of a line item. Please be sure to include the entire line_items array, even if updating only one line item; otherwise, you may accidentally delete any others. At minimum, the required line item properties include `id`, `product_id`, `quantity`, and `variant_id` (if applicable).

WARNING: Updating line items may causes order prices (subtotal, taxes, shipping, total) to change.

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
        "line_items":
        [
            {
                "id": "550e8400-e29b-41d4-a716-446655440000",
                "product_id": "530e40d428ee4100002bfa78",
                "quantity": 2,
                "price": 1000,
                "weight": 0,
                "enable_taxes": true,
                "product_name": "Koala Bears",
                "variant_name": null,
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
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/cancel
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "order_status": "cancelled",
        "history": 
        [
            {
                "type": "order_created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "order_cancelled",
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
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/charge
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "payment_status": "paid",
        "payments":
        [
            {
                "id": "647f2cbf-4e86-42de-90b7-7a325a42a80b",
                "amount": 1000,
                "amount_refunded": 0,
                "refunded": false,
                "fee": 20,
                "charge_id": "ch_3yLwBnzD7oqprW",
                "gateway": "stripe",
                "refunds": [],
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "card": {
                    "exp_month": 1,
                    "exp_year": 2017,
                    "last4": "0466",
                    "type": "MasterCard"
                }
            }
        ],
        "history": 
        [
            {
                "type": "order_created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "charge_success",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was charged 10.00."
            }
        ],
        "balance": 0,
        "paid": 1000,
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
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440/refund
```

##### Example Response
```json
{
    "meta": {
        "code": 200
    },
    "data": {
        "payment_status": "refunded",
        "payments":
        [
            {
                "id": "647f2cbf-4e86-42de-90b7-7a325a42a80b",
                "amount": 1000,
                "amount_refunded": 1000,
                "refunded": true,
                "fee": 20,
                "charge_id": "ch_3yLwBnzD7oqprW",
                "gateway": "stripe",
                "refunds":
                [
                    {
                        "created": 1401994491000,
                        "created_date": ...,
                        "amount": 1000,
                        "refund_id": null
                    }
                ],
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "card": {
                    "exp_month": 1,
                    "exp_year": 2017,
                    "last4": "0466",
                    "type": "MasterCard"
                }
            }
        ],
        "history": 
        [
            {
                "type": "order_created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "charge_success",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was charged 10.00."
            },
            {
                "type": "refund_success",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was refunded 10.00."
            }

        ],
        "balance": 0,
        "paid": 1000,
        "refunded": 1000,
        ...
    }
}
```


### Fulfill an Order

This convenience endpoint will mark the fulfillment_status as `fulfilled` and allow you to optionally add `courier` and `number`. This endpoint will trigger email notifications to the buyer (if enabled).

```
POST /v2/orders/{id}/fulfillment_fulfill
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
https://api.trycelery.com/v2/orders/530ec58358b6ee0000f5d440 \
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
        "history": 
        [
            {
                "type": "order_created",
                "created": 1393476995707,
                "created_date": "2014-02-27T04:56:35.707Z",
                "body": "Your order was created."
            },
            {
                "type": "fulfillment_success",
                "created": 1401993491000,
                "created_date": "2014-06-05T06:38:00.000Z",
                "body": "Your order was fulfilled."
            }
        ],
        ...
    }
}
```
