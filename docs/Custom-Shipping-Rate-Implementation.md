# Custom Shipping Rate Specifications

To use Custom Shipping Rates with ReCharge, you will need middleware that can return and receive payloads that follow the guidelines in this article.


## Requirements
Your Custom Shipping Rate API must send and receive data over HTTPS using the `application/json` header.

> HMAC support for your application and the ability to send/recieve custom headers are optional.

**Request headers**

```json
Accept: application/json
Content-Type: application/json
```

### Callback URL

ReCharge will send a request to your callback URL with the below fields.

### Example request:

```json
{
  "rate": {
    "destination": {
      "country": "USA",
      "postal_code": "31904",
      "province": "GA",
      "city": "Columbus",
      "name": "Bill Harden",
      "address1": "1001 Red Maple Way",
      "address2": "Suite B"
    },
    "items": [{
      "name": "Vacuum Tube",
      "sku": "678968943234",
      "quantity": 1,
      "grams": 1000,
      "price": 1999,
      "vendor": "Bolton Hifi",
      "requires_shipping": true,
      "fulfillment_service": "manual",
      "product_id": 48447225880,
      "variant_id": 258644705304
    }],
    "currency": "USD",
    "locale": "en"
  }
}
```
**Top-level field**

|Field|Description|Type|Required|
|-|-|-|-|
|Rate|Parent that contains all other fields|Object|Yes|

**Secondary levels**

|Fields|Description|Type|Required|
|-|-|-|-|
|destination|Contains destination fields|object|yes|
|items|Contains list of items|array|yes|
|currency|Currency value|string|yes|
|locale|Locale value|string|yes|

**Destination properties**

|Fields|Description|Type|Required|
|-|-|-|-|
|country|Country code|string|yes|
|postal_code|Address postal code|string|yes|
|province|Address state/province|string|yes|
|city|Address city|string|yes|
|name|Customer name|string|yes|
|address1|Line 1 of address|string|yes|
|address2|Line 1 of address|string|no|

|Fields|Description|Type|Required|
|-|-|-|-|
|vendor|Product vendor|string|yes|
|variant_id||integrer|no|
|sku|Product SKU|string|no|
|requires_shipping|Indicates whether shipping is required for this product|Boolean|yes|
|quantity|Product count|integer|yes|
|product_id|Unique product identifier|integer|no|
|price|Price of product in cents|integer|yes|
|name|Product name|string|yes|
|grams|Product weight|integer|yes|
|fulfillment_service||string|yes|

## Response

Your application should send a `200 OK` response and JSON payload with the following properties.

<!-- theme: warning-->
> Any response to ReCharge's request other than `200` to our will be considered an error.

**Example response**

```json
{
   "rates": [
       {
           "service_name": "Expedited",
           "service_code": "EX",
           "total_price": "1295",
           "description": "expedited shipping",
           "currency": "USD",
       },
       {
           "service_name": "fedex-2dayground",
           "service_code": "2D",
           "total_price": "2934",
           "description": "fed ex 2 day",
           "currency": "USD",
       },
       {
           "service_name": "fedex-priorityovernight",
           "service_code": "1D",
           "total_price": "3587",
           "description": "fedex priority overnight",
           "currency": "USD",
       }
   ]
}
```

**Top-level field**

|Field|Description|Type|
|-|-|-|
|rates|Parent list of rates|array|

**Secondary level**

|Fields|Description|Type|
|-|-|-|
|service_name|Name of shipping service|string|
|service_code|Service code|string|
|total_price|Price in cents|string|
|description|Description of shipping service|string|
|currency|Currency code|string|


## Response codes

See the following response codes for troubleshooting.

### 200 Response

If the service finds rates, return them along with a status of 200.

**Example**
```json
{
   "rates": [
       {
           "service_name": "Expedited",
           "service_code": "EX",
           "total_price": "1295",
           "description": "expedited shipping",
           "currency": "USD",
       }
}
```

### 200 Response (Empty)

If the custom shipping rate service cannot find any shipping rates, return an empty rates list.

**Example**
```json
{
    "rates":[]
}
```

### 401 Response
If the service expects an api_key token and it is missing or invalid, reply with a 401.

**Example**
```json
{
    "error": "HMAC_INVALID_MISSING"
}
```

### 429 Response
 If the service must rate limit, reply with 429.

**Example**
```json
{
    "error": "RATE_LIMITED"
}
```

### 400 Response
If the payload sent to the service is invalid, return 400.

### Other
Appropriate status code
```json
{
    "error": "<error code>"
}
```

## HMAC 

You can add additional security to your shipping rate service using HMAC by providing a secret ReCharge will then use to create a timestamp to append to the end of a URL.

`https://example.com/?timestamp=785923045&hmac=somevaluehere`

### How to verify HMAC

When your app receives a request, you can check for this HMAC and verify it matches.

1. Receive the HMAC and timestamp from the request.

2. In your custom shipping rate service, create a new HMAC digest using your own copy of the secret key and the request timestamp as the message.  Use SHA256 as the digest mode.

**Example Message:** `timestamp=785923045`

3. Compare this HMAC to the HMAC on the URL.  If they match, then the message is valid.

If your client is expecting an HMAC and it was missing or invalid, you can reply with a `401` HTTP Status code.

**Sample Python code**

```python
import hmac
from hashlib import sha256

secret_key = "your secret key"

# get timestamp and hmac from request
request_timestamp = request.args.get("timestamp")
request_hmac = request.args.get("hmac")

digest_maker = hmac.new(
    secret_key.encode("utf-8"),
    msg="timestamp={}".format(request_timestamp).encode("utf-8"),
    digestmod=sha256,
)

# If they do not match, the request is invalid
if digest_maker.hexdigest() != request_hmac:
    raise Exception()
```

## Testing

You can can use our testing form to verify the custom shipping rate service.

Use the **Service Tester** on the **integrations** page to verify your service.  We've pre-populated the request payload values, but you can change these.  Click **Test** and the results of the request will be displayed in the form.














