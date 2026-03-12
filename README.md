The API is REST, using JSON for serialization, with no root element.

We also have one webhook endpoint available. See the bottom of this file.

   * [Making a request](#making-a-request)
      * [Authentication](#authentication)
      * [Identify your application in the User-Agent string](#identify-your-application-in-the-user-agent-string)
      * [Example request](#example-request)
      * [Custom contact fields](#custom-contact-fields)
      * [Need anything?](#need-anything)
   * [Contacts](#contacts)
      * [List contacts](#list-contacts)
      * [Create/update contact](#createupdate-contact)
      * [Update contact credentials](#credentials-contact)
      * [Get contact by ID](#get-contact-by-id)
      * [Find contact by email](#find-contact-by-email)
      * [Add tag to contact](#add-tag-to-contact)
      * [Remove tag from contact](#remove-tag-from-contact)
      * [Course completions](#course-completions)
      * [Issue points](#issue-points)
   * [Lists](#lists)
      * [Get lists](#get-lists)
      * [Subscribe to list](#subscribe-to-list)
     *  [Bulk subscribe to list](#bulk-subscribe-to-list)
      * [Unsubscribe from a list](#unsubscribe-from-a-list)
      * [Find subscription by email](#find-subscription-by-email)
      * [List subscriptions](#list-subscriptions)
   * [Products](#products)
      * [Get products](#get-products)
      * [Get product by ID](#get-product-by-id)
      * [Create free purchase](#create-free-purchase)
      * [Find purchase by email](#find-purchase-by-email)
      * [Find purchase by ID or token](#find-purchase-by-id-or-token)
      * [Search purchases](#search-purchases)
   * [Tags](#tags)
      * [Get tags](#get-tags)
      * [Get tag by ID](#get-tag-by-id)
      * [Tag a contact using a tag ID](#tag-a-contact-using-a-tag-id)
      * [Untag a contact using a tag ID](#untag-a-contact-using-a-tag-id)
   * [Custom domains](#custom-domains)
   * [Broadcasts](#broadcasts)
      * [Get broadcasts](#get-broadcasts)
      * [Get broadcast by ID](#get-broadcast-by-id)
      * [Create broadcast](#create-broadcast)
      * [Send test broadcast](#send-test-broadcast)
      * [Get broadcast activity](#get-broadcast-activity)
   * [Invoices](#invoices)
   * [Affiliates](#affiliates)
   * [Point types](#point-types)
   * [Administrators](#administrators)
      * [Get administrators](#get-administrators)
      * [Get administrator by ID](#get-administrator-by-id)
      * [Get administrator by email](#find-administratorship-by-email)
      * [Create or update administrator](#createupdate-administrator)
      * [Remove administrator](#remove-administrator)
      * [Get admin roles](#admin-roles)
   * [Automations](#automations)
      * [Get automations](#get-automations)
      * [Start automation for a contact](#start-automation-for-a-contact)
   * [Segments](#segments)
   * [Assets](#assets)
   * [Account and Zapier helper endpoints](#account-and-zapier-helper-endpoints)
      * [Get account custom fields](#get-account-custom-fields)
      * [Who am I for Zapier](#who-am-i-for-zapier)
      * [Get Zapier customer action fields](#get-zapier-customer-action-fields)
      * [Poll sample customers for new subscriptions](#poll-sample-customers-for-new-subscriptions)
      * [Poll sample customers for deleted subscriptions](#poll-sample-customers-for-deleted-subscriptions)
      * [Poll sample purchases for new purchases](#poll-sample-purchases-for-new-purchases)
      * [Poll sample purchases for canceled purchases](#poll-sample-purchases-for-canceled-purchases)
      * [Poll sample customers for new taggings](#poll-sample-customers-for-new-taggings)
      * [Poll sample customers for deleted taggings](#poll-sample-customers-for-deleted-taggings)
   * [Zapier subscriptions](#zapier-subscriptions)
   * [Asynchronous requests](#asynchronous-requests)
   * [Webhook endpoint](#webhook-endpoint)


Making a request
================

All API request URLs start with `https://simplero.com/api/v1/`.

All requests have to be over HTTPS.


Authentication
--------------

Requests are authenticated using HTTP Basic Auth, with the API key as the username, password left empty.

You can also provide the API key in the `X-API-Key` request header instead of HTTP Basic Auth.

Failure to provide a valid API key will result in a status 401 (unauthorized) with a JSON body of:

```json
{ "error": "Bad API key" }
```


Identify your application in the User-Agent string
--------------------------------------------------

Include the name of your application and a contact email in your User-Agent string like this:

* `Some app (email@example.test)`

Failure to do so will result in a status 400 (bad request) with a JSON body of:

```json
{ "error": "Incorrect User-Agent" }
```


Example request
---------------

Here's what a complete request looks like:

```shell
curl -u "API_KEY:" -H 'User-Agent: Some app (test@example.test)' https://simplero.com/api/v1/lists.json
```

Replace `API_KEY` with your actual API key that you get from the Simplero Admin interface under Settings > Integrations. Note the colon after your API key. That's important.

To add or update something, include the JSON data with a `Content-Type` header (assuming your API key is "f37dTH32fP" - in reality, yours is going to be longer):

```shell
curl -u "f37dTH32fP:" -H 'Content-Type: application/json' -H 'User-Agent: Some app (email@example.test)' \
  -d '{ "first_name": "New Subscriber", "email": "someemail@example.test" }' \
  https://simplero.com/api/v1/lists/1/subscribe.json
```


Custom contact fields
---------------------

Custom contact fields (added under Contacts > Fields in your admin interface) can be set using their internal names, which look like `field_ID_SUBFIELD`.

The `ID` part is the internal ID, and the `SUBFIELD` part depends on the type of field. When _reading_ the value of custom contact fields, you will get all subfields, and you will also get a combined field that includes a string representation of the entire value of the field, keyed by the label of the field. You can use whichever one you prefer.

**Address fields**

An address field will have subfields `address`, `address_2`, `postal_code`, `city`, `region_code`, and `country_code`.

In all, the field names to update an address field could look like this:

* `field_123_address`
* `field_123_address_2`
* `field_123_postal_code`
* `field_123_city`
* `field_123_region_code`
* `field_123_country_code`

**Phone fields**

A phone field will have subfields:

* `field_123_country_code` - the 2 letter country code, ex: "UK"
* `field_123_number` - the phone number, without the country calling code, ex: "55555555555"

You can alternatively use `field_123_all` to set a complete number with the country calling code: "4455555555555".


Need anything?
--------------

Need help using the API? Contact our support team using the HELP link inside your Simplero admin interface.

Have a feature request for the API? Submit it [here](https://simplero.community/ideas).



Contacts
========

List contacts
--------------

`GET /customers.json` will return a paginated list of contacts.

**Parameters:**

- `page` — 0-indexed page number (default 0)
- `per_page` — results per page, 1–100 (default 20)
- `from` / `to` — filter by created_at (ISO 8601)
- `updated_from` / `updated_to` — filter by updated_at (ISO 8601)
- `tag_id` — filter contacts that have this tag

**Response:**

```json
[
  {
    "id": 1046901344,
    "token": "abc123",
    "name": "Calvin Correli",
    "email": "calvin@simplero.com"
  }
]
```


Create/update contact
--------------

`POST /customers.json` will create a new contact, or update an existing contact with the same email

**POST request body:**

```json
{
  "email":                   "calvin@simplero.com",
  "first_name":              "Calvin",
  "last_name":               "Correli",
  "ip_address":              "77.66.17.105",
  "referrer":                "http://google.com/search?q=foo",
  "ref":                     123,
  "track":                   "api",
  "first_activated_at":      "2012-03-22T17:35:50-05:00",
  "auto_responder_start_at": "2012-03-22T17:35:50-05:00",
  "landing_page_id":         123,
  "tags":                    "tag1,tag2",
  "note":                    "A new note",
  "phone":                   "15551231234",
  "field_123_value":         "calvincorreli",
  "gdpr_consent":            1,
  "gdpr_consent_text":       "Subscribe to our newsletter"
}
```

The only required argument is `email`.

`tags` will add these as additional tags. You cannot remove tags with this API call.

`note` will add a new note to the customer. You cannot remove notes this way.

`phone` is a sanitized phone number with country code first, eg. '15551231234'.

Pass `gdpr_consent` with a value of `1` to indicate GDPR consent was request and granted, `0` to indicate GDPR consent was requested but not granted, or omit if GDPR consent was not requested. `gdpr_consent_text` should be the text of the request (e.g. the label on the checkbox).

By default we will not override any existing information, only add new. Set `override` = `yes` to override existing values, except `email` which cannot be changed this way.

**Response:**

```json
{
  "id":                                       1046901344,
  "email":                                    "calvin@simplero.com",
  "created_at":                               "2016-01-21T09:07:28.471-05:00",
  "updated_at":                               "2016-01-21T09:07:28.471-05:00",
  "ref":                                      123,
  "track":                                    "api",
  "affiliate_id":                             157416808,
  "lifetime_value_cents_excl_tax":            0,
  "lead_acquisition_cost_cents_excl_tax":     0,
  "customer_acquisition_cost_cents_excl_tax": 0,
  "first_names":                              "Calvin",
  "last_name":                                "Correli",
  "do_not_contact":                           false,
  "ip_address":                               "192.168.1.2",
  "referrer":                                 "http://google.com/search?q=foo",
  "name":                                     "Calvin Correli",
  "simplero_id":                              null,
  "contact_since":                            "less than a minute",
  "tag_names":                                "foo,bar",
  "phone":                                    "15551231234",
  "field_123_value":                          "calvincorreli"
}
```

Also included will be any custom contact fields.

**Error response:**

```json
{ "error": "Comma-separated, error-messages" }
```

This will be sent using HTTP status code 422 Unprocessable Entity.


Credentials contact
--------------

`POST /customers/update_credentials.json` will update a contact's login credentials.

Note: Use this API endpoint only with the consent of the contact. Be mindful of its use.

**POST request body:**

```json
{
  "email":                   "calvin@simplero.com",
  "first_name":              "Calvin",
  "last_name":               "Correli",
  "update_login_email":      true
}
```

For identification, you can pass a `id`, `token` or `email` field.

**Response:**

```json
{
  "id":                                       1046901344,
  "email":                                    "calvin@simplero.com",
  "created_at":                               "2016-01-21T09:07:28.471-05:00",
  "updated_at":                               "2016-01-21T09:07:28.471-05:00",
  "ref":                                      123,
  "track":                                    "api",
  "affiliate_id":                             157416808,
  "lifetime_value_cents_excl_tax":            0,
  "lead_acquisition_cost_cents_excl_tax":     0,
  "customer_acquisition_cost_cents_excl_tax": 0,
  "first_names":                              "Calvin",
  "last_name":                                "Correli",
  "do_not_contact":                           false,
  "ip_address":                               "192.168.1.2",
  "referrer":                                 "http://google.com/search?q=foo",
  "name":                                     "Calvin Correli",
  "simplero_id":                              null,
  "contact_since":                            "less than a minute",
  "tag_names":                                "foo,bar",
  "phone":                                    "15551231234",
  "field_123_value":                          "calvincorreli"
}
```

Also included will be any custom contact fields.

**Error response:**

```json
{ "error": "Comma-separated, error-messages" }
```

This will be sent using HTTP status code 422 Unprocessable Entity.
Unexpected errors will have HTTP status 500


Get contact by ID
-----------------

`GET /customers/123.json` will get a JSON representation of a contact, looked up by the internal ID.

Replace `123` with the ID of the contact you're interested, which you will have gotten from a previous call to the API.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.



Find contact by email
---------------------

`POST /customers/find.json` will get a JSON representation of a contact, looked up by email.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

You can also look up the contact by passing `id` or `token` instead of `email`.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.



Add tag to contact
------------------

`POST /customers/add_tag.json` will add a tag to the contact with the given email

**POST request body:**

```json
{
  "email": "calvin@simplero.com",
  "tag":   "new-tag-to-add"
}
```

You can also identify the contact with `id` or `token` instead of `email`.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.


Remove tag from contact
--------------

`POST /customers/remove_tag.json` will remove a tag from the contact with the given email

**POST request body:**

```json
{
  "email": "calvin@simplero.com",
  "tag":   "tag-to-remove"
}
```

You can also identify the contact with `id` or `token` instead of `email`.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.


Course completions
--------------

`POST /customers/course_completions.json` will provide a broad view of the course progress across all sites and courses for a particular user

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

Response codes:

- `401` if API key is invalid
- `404` if contact is not found
- `200` if contact is found

200 response structure:

```json
{
  "contact": {
    "name": "Calvin Correli",
    "email": "calvin@simplero.com",
    "courses": [
      {
        "title": "My course",
        "modules": [
          {
            "title": "My module",
            "position": 1,
            "completion": "20%",
            "lessons": [
              {
                "title": "My lesson",
                "position": 1,
                "completion": "Not started" | "Completed" | "Viewed",
                "views": 0
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Issue points
------------

`POST /customers/issue_points.json` will issue points to a contact.

**POST request body:**

```json
{
  "email": "calvin@simplero.com",
  "point_type_id": 12,
  "amount": 50,
  "note": "Bonus points for attending the workshop live"
}
```

You can identify the contact with `email`, `id`, or `token`.

`point_type_id` and `amount` are required. `amount` must be greater than `0`.

Use [`GET /point_types.json`](#point-types) to discover valid point types for the account.

**Response:**

```json
{
  "success": true
}
```

**Error response:**

```json
{
  "errors": [
    "Point type is not a valid point type"
  ]
}
```

This will be sent using HTTP status code 422 Unprocessable Entity.


Lists
=====


Get lists
---------

`GET /lists.json` will return all the account's lists

**Response:**

```json
[
  {
    "id":1942123930,
    "name":"Marianne's Universe",
    "internal_name":"Main list",
    "confirm_pending_url":null,
    "created_at":"2013-10-23T16:45:14+02:00",
    "email_field_name":"89aet",
    "reply_to":null,
    "require_confirmation":false,
    "return_to":null,
    "sender_email":"marianne@mariannesclub.com",
    "sender_name":"Marianne",
    "unsubscribed_url":null
  },
  {
    "id":7253003287,
    "name":"Free tips",
    "internal_name":"E-bomb offer 1",
    "confirm_pending_url":null,
    "created_at":"2013-10-23T16:45:14+02:00",
    "email_field_name":"atn23nh2e",
    "reply_to":null,
    "require_confirmation":false,
    "return_to":null,
    "sender_email":"marianne@mariannesclub.com",
    "sender_name":"Marianne",
    "unsubscribed_url":null
  }
]
```

`id` is the id you'd need to use in the URL when subscribing or unsubscribing contacts.

Subscribe to list
-----------------

`POST /lists/1/subscribe.json` will add a new subscriber to the list

Replace `1` with the id of the list to subscribe to.

**POST request body:**

```json
{
  "first_name":              "Calvin",
  "last_name":               "Correli",
  "email":                   "calvin@simplero.com",
  "ip_address":              "77.66.17.105",
  "referrer":                "http://google.com/search?q=foo",
  "ref":                     123,
  "track":                   "api",
  "first_activated_at":      "2012-03-22T17:35:50-05:00",
  "auto_responder_start_at": "2012-03-22T17:35:50-05:00",
  "landing_page_id":         123,
  "tags":                    "tag1,tag2",
  "phone":                   "15551231234",
  "gdpr_consent":            true
}
```

Everything except `email` is optional.

`ip_address` should be the actual IP address that the actual customer used to subscribe to the list.
It should NOT be a private IP address, nor should it be faked or manipulated in any way.

`landing_page_id` is the landing page (signup form) within Simplero that this conversion should be counted for.
This is only relevant when using Simplero's conversion tracking featuer.

`ref` is the affiliate ref that this conversion should be attributed to.

`referrer` should be the last external URL visited before the subscribe form.

`track` is any keyword that you wish to associate with this opt-in, so you can track where people came from.

`tags` is a comma-separated string of tags to add to this contact. Individual tags will be trimmed of whitespace at the beginning and end. Everything else will be preserved as-is.

`phone` is a sanitized phone number with country code first, eg. '15551231234'.

**Response:**

```json
{
  "id": 1073281992,
  "list_id": 51750419,
  "customer_id": 1046904945,
  "return_to": "",
  "active": false,
  "confirmed": false,
  "confirmed_at": null,
  "unsubscribed_at": null,
  "created_at": "2016-01-21T09:07:28.471-05:00",
  "updated_at": "2016-01-21T09:07:28.471-05:00",
  "affiliate_id": 157416808,
  "ref": null,
  "track": "api",
  "first_activated_at": null,
  "landing_page_id": 1067655620,
  "confirmation_request_sent_at": "2016-01-21T09:07:28.471-05:00",
  "confirmation_request_sent_count": 1,
  "auto_responder_start_at": null,
  "skip_auto_responses": false,
  "ip_address": "192.168.1.2",
  "referrer": "http://google.com/search?q=foo",
  "auto_responder_paused_at": null,
  "suspended_at": null,
  "customer": {
    "email": "calvin@correli.me",
    "first_names": "Calvin",
    "last_name": "Correli",
    "id": 1046904945,
    "created_at": "2016-01-21T09:07:28.471-05:00",
    "updated_at": "2016-01-21T09:07:28.471-05:00",
    "ref": null,
    "track": null,
    "affiliate_id": null,
    "lifetime_value_cents_excl_tax": 0,
    "lead_acquisition_cost_cents_excl_tax": 0,
    "customer_acquisition_cost_cents_excl_tax": 0,
    "do_not_contact": false,
    "ip_address": null,
    "referrer": null,
    "name": "Calvin Correli",
    "simplero_id": null,
    "contact_since": "less than a minute",
    "tag_names": "foo,bar",
    "phone": "15551231234"
  }
}
```


Bulk subscribe to list
-----------------

`POST /lists/1/bulk_subscribe.json` will multiple subscribers to the list.

Replace `1` with the id of the list to subscribe to.

**POST request body:**

```json
{
  "subscriber_data": [
    {
      "first_name": "Calvin",
      "last_name": "Correli",
      "email": "calvin@simplero.com"
    },
    {
      "first_name": "Scalvin",
      "last_name": "Scorreli",
       "email": "scalvin@simplero.com"
    }
  ]
}
```

The format of each entry under `subscriber_data` is the same as for the "Subscribe to list" endpoint. A maximum of 1,000
entries may be included in a single request.

**Response:**

```json
{
     "token": "ABC123"
}
```

This token may be used to [check the status of the request](#asynchronous-requests).

When the request is completed, The `result` attribute of the request will contain an array where the format of each
entry is the same as the response of the "Subscribe to list" endpoint.

Unsubscribe from a list
-----------------------

`POST /lists/1/unsubscribe.json` will unsubscribe a customer from the list

Replace `1` with the id of the list to unsubscribe from.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

`email` is the email address of the contact to unsubscribe from the list.

**Response:**

```json
{ success: true }
```

When `success` is true it means the contact was successfully unsubscribed.

When `success` is false, it means we couldn't find any contact with this email. The response will be returned using HTTP status code 400 Bad Request.

Find subscription by email
-----------------------

`POST /lists/1/subscriptions/find.json` will get a JSON representation of an array of subscriptions, looked up by email.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

Responds with the array of subscription object. Will respond with empty array if no such subscription exists.
Will respond with 404 if no such list exists.


List subscriptions
-----------------------

`GET /subscriptions.json` will return a paginated list of subscriptions.

**Parameters:**

- `page` — 0-indexed page number (default 0)
- `per_page` — results per page, 1–100 (default 20)
- `list_id` — filter by list
- `status` — `active`, `unsubscribed`, or `suspended`
- `from` / `to` — filter by created_at (ISO 8601)

**Response:**

```json
[
  {
    "id": 1073281992,
    "list_id": 51750419,
    "customer_id": 1046904945,
    "active": true,
    "confirmed": true,
    "confirmed_at": "2016-01-21T09:07:28.471-05:00",
    "first_activated_at": "2016-01-21T09:07:28.471-05:00",
    "unsubscribed_at": null,
    "suspended_at": null,
    "created_at": "2016-01-21T09:07:28.471-05:00",
    "status": "active"
  }
]
```


Products
========

Get products
------------

`GET /products.json` will list all the account's products.

**Parameters:**

- `page` — 0-indexed page number (default 0)
- `per_page` — results per page, 1–100 (default 20)

**Response:**
```JSON
[
  {
    "id": 123,
    "name": "My product",
    "handle": "my-product" ,
    "sales_page_url": null,
    "received_price_cents": 3000,
    "incoming_price_cents": 0,
    "sales_price_cents": 3000
  }
]
```

Get product by ID
-----------------

`GET /products/123.json` will get product info.

**Response:**
```JSON
{
  "id": 123,
  "name": "My product",
  "handle": "my-product" ,
  "sales_page_url": null,
  "received_price_cents": 3000,
  "incoming_price_cents": 0,
  "sales_price_cents": 3000
}
```

Create free purchase
--------------------

`POST /products/1/purchases.json` will create a free purchase for the product.

Replace `1` with the id of the product.

This endpoint only works on active, non-trial accounts.

**POST request body:**

```json
{
  "email": "calvin@simplero.com",
  "first_name": "Calvin",
  "last_name": "Correli",
  "skip_contract": false
}
```

`email` is required.

You may also pass `first_names`, `name`, and `last_name` to help identify or create the billing contact.

Set `skip_contract` to `true` if you do not want Simplero to generate the purchase contract.

**Response:**

Returns HTTP status code 201 Created with a Zapier-style purchase payload. The response includes the purchase data, the billing contact, and any entrants attached to the purchase.

```json
{
  "purchase_id": 10000001,
  "product_id": 200000,
  "product_name": "Sample Product Name",
  "state": "paid",
  "name": "Calvin Correli",
  "email": "calvin@simplero.com",
  "currency": "USD",
  "billing_contact": {
    "id": 1046901344,
    "name": "Calvin Correli",
    "email": "calvin@simplero.com",
    "token": "CONTACT_TOKEN"
  },
  "entrants": []
}
```

If the account is not active, the endpoint returns HTTP status 400 with:

```json
{
  "error": "Purchases can only be performed on active, non-trial accounts."
}
```

Find purchase by email
-----------------------

`POST /products/1/purchases/find.json` will get a JSON representation of an array of purchases, looked up by email.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

Responds with the array of purchase object. Will respond with empty array if no such purchase exists.
Will respond with 404 if no such product exists.


Find purchase by ID or token
-----------------------

`POST /products/1/purchases/find.json` will get a JSON representation of an array of purchases, looked up by ID or token.

You can pass them on the URL as a query parameter or using the POST body.

- `POST /products/1/purchases/find.json?id=<ID>`
- `POST /products/1/purchases/find.json?token=<token>`

**POST request body:**

```json
{
  "id": "<ID>",
  "token": "<token>"
}
```

Responds with a single purchase object directly on the root element.
Will respond with 404 if product or purchase does not exist.

Example successul 200 OK response:

```json
{
  "id": 10000001,
  "account_id": 100000,
  "product_id": 200000,
  "customer_id": 30000001,
  "affiliate_id": null,
  "email_verified_at": null,
  "state": "paid",
  "ref": null,
  "track": null,
  "company_name": null,
  "vat_no": null,
  "address": null,
  "postal_code": null,
  "city": null,
  "email": "user@example.com",
  "purchased_at": "2026-01-28T12:41:47.000-05:00",
  "refunded_at": null,
  "welcome_email_sent_at": null,
  "created_at": "2026-01-28T12:41:44.000-05:00",
  "updated_at": "2026-01-28T12:41:48.000-05:00",
  "token": "PURCHASE_TOKEN",
  "first_failed_at": null,
  "failure_count": 0,
  "wants_marketing": true,
  "wants_notifications": true,
  "next_attempt_at": null,
  "country_code": "US",
  "phone_number": null,
  "received_price_cents": 2700,
  "received_shipping_cents": 0,
  "received_tax_cents": 0,
  "received_total_cents": 2700,
  "tax_percent": 0.0,
  "first_names": "FirstName",
  "last_name": "LastName",
  "freebie": false,
  "phone_country_code": "US",
  "active": true,
  "incoming_price_cents": 0,
  "incoming_total_cents": 0,
  "tax_region_id": null,
  "quantity": 1,
  "address_2": null,
  "canceled_at": null,
  "marked_failed_at": null,
  "landing_page_id": null,
  "price_id": 400000,
  "price_period_id": 500000,
  "price_period_rep_no": 1,
  "committed_until": null,
  "auto_renew": false,
  "period_ends_at": null,
  "region_code": null,
  "coupon_id": null,
  "discount_type": "none",
  "discount_percent": null,
  "discount_cents": null,
  "discount_on_renewal": true,
  "ip_address": "0.0.0.0",
  "upsold": true,
  "offer_id": null,
  "upsell_id": null,
  "parent_purchase_id": null,
  "donation_cents": 0,
  "donation_on_renewal": false,
  "mailing_id": 600000,
  "affiliate_percentage": null,
  "discount_on_installments": false,
  "currency_code": "USD",
  "renewed_at": null,
  "renewal_warning_sent_at": null,
  "discount_expires_at": null,
  "refund_right_waived_by_id": null,
  "refund_right_waived_at": null,
  "refund_right_waived_ip": null,
  "gdpr_consent_at": null,
  "gdpr_consent_text": null,
  "utm_adgroup": "",
  "utm_campaign": "sample-campaign",
  "utm_content": "",
  "utm_keyword": "",
  "utm_medium": "email",
  "utm_source": "email-list",
  "utm_term": "",
  "vat_no_validated_at": null,
  "vat_no_validation_result": null,
  "cancellation_reason": null,
  "replaces_purchase_id": null,
  "trial_ending_warning_sent_at": null,
  "time_since_last_charge_reminder_sent_at": null,
  "checkout_started_at": "2026-01-28T12:41:44.000-05:00",
  "payment_method_trial_ending_warning_sent_at": null,
  "donation_strategy": "add",
  "affiliate_fixed_amount": null,
  "banked_days": null,
  "banked_previous_plan": null,
  "order_id": 700000,
  "replace_purchase_after_expiry": false,
  "replacement_purchase_price_id": null,
  "review_required_at": null,
  "checkout_from_iframe": false,
  "funnel_step_id": null,
  "paused_at": null,
  "latest_flexpay_transaction_id": null,
  "event_occurrence_id": null,
  "sales_rep_id": null,
  "royalty_purchase": false,
  "subscription_change_period_ends_at": null,
  "event_occurrence_participant_id": null,
  "purchase_url": "https://example.com/purchases/PURCHASE_TOKEN",
  "edit_purchase_url": "https://example.com/purchases/PURCHASE_TOKEN/edit",
  "payment_processor_name": "Stripe",
  "total_payment_processor_fee_cents": 108,
  "last_charge_paid": {
    "paid_at": "2026-01-28T12:41:47.000-05:00",
    "our_price_cents": 2700,
    "our_tax_cents": 0,
    "our_total_cents": 2700
  },
  "product": {
    "name": "Sample Product Name",
    "manual_account_number": null
  },
  "payment_method": {
    "cardtype": "visa"
  },
  "transactions": [
    {
      "id": 800000,
      "transaction_id": "PAYMENT_INTENT_ID",
      "state": "succeeded",
      "transaction_type": "charge",
      "currency_code": "USD",
      "created_at": "2026-01-28T12:41:44.000-05:00",
      "amount": 27,
      "fee": 1.08,
      "tax": 0.0
    }
  ],
  "sales_rep_email": null,
  "purchase": {
    "purchase_id": 10000001,
    "product_id": 200000,
    "product_name": "Sample Product Name",
    "created_at": "2026-01-28 12:41 -0500",
    "purchased_at": "2026-01-28 12:41",
    "state": "paid",
    "name": "FirstName LastName",
    "email": "user@example.com",
    "company": null,
    "vat_no": null,
    "address": null,
    "address_2": null,
    "postal_code": null,
    "city": null,
    "region_code": null,
    "region": null,
    "country_code": "US",
    "country": "United States",
    "phone_country_code": "US",
    "phone_number": null,
    "vat_percent": 0.0,
    "price_id": 400000,
    "price_name": "Simple price",
    "quantity": 1,
    "received_amount": 27,
    "received_shipping": 0.0,
    "received_tax": 0.0,
    "received_total": 27,
    "incoming_amount_excl_tax_and_shipping": 0.0,
    "payment_method_type": "credit_card",
    "card_type": "visa",
    "track": null,
    "ref": null,
    "affiliate_id": null,
    "affiliate_name": null,
    "affiliate_email": null,
    "wants_marketing_emails": true,
    "wants_any_emails": true,
    "last_charge_at": "2026-01-28",
    "next_charge_at": null,
    "period_ends_at": null,
    "currency": "USD",
    "entrants": [
      {
        "id": 900000,
        "created": "2026-01-28 12:41 -0500",
        "updated": "2026-01-28 12:41 -0500",
        "canceled_at": null,
        "wants_notifications": true,
        "first_name": "FirstName",
        "last_name": "LastName",
        "email": "user@example.com",
        "simplero_id": "user_id",
        "contact": {
          "id": 30000001,
          "email": "user@example.com",
          "token": "CONTACT_TOKEN",
          "name": "FirstName LastName",
          "simplero_id": "user_id",
          "contact_since": "about X months",
          "tag_names": "sample-tag-1,sample-tag-2",
          "phone": null
        }
      }
    ]
  },
  "children": []
}
```

Search purchases
----------------

`GET /purchases/search.json` will search purchases across the whole account.

**Parameters:**

- `page` — 1-indexed page number (default 1)
- `per_page` — results per page, 1–100 (default 20)
- `filters[product_id]` — filter by product ID
- `filters[state]` — filter by purchase state
- `filters[created][start_at]` / `filters[created][end_at]` — filter by purchase creation time (ISO 8601)
- `filters[first_successful_charge][start_at]` / `filters[first_successful_charge][end_at]` — filter by the date of the first paid charge (ISO 8601)

Results are ordered by `created_at` descending.

**Response:**

Returns an array of purchase objects in the same format as the purchase object returned by [Find purchase by ID or token](#find-purchase-by-id-or-token).


Tags
====

Get tags
--------

`GET /tags.json` will return a paginated list of tags.

**Parameters:**

- `page` — 0-indexed page number (default 0)
- `per_page` — results per page, 1–100 (default 20)

**Response:**

```json
[
  {
    "id": 123,
    "name": "vip-customer"
  }
]
```

Get tag by ID
-------------

`GET /tags/123.json` will get a tag by its ID.

**Response:**

```json
{
  "id": 123,
  "name": "vip-customer"
}
```

Tag a contact using a tag ID
----------------------------

`POST /tags/123/tag.json` will add an existing tag to a contact.

Replace `123` with the id of the tag.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

`email` is required.

If the email does not already belong to a contact in the account, Simplero will create that contact first and then apply the tag.

**Response:**

```json
{
  "success": true
}
```

Untag a contact using a tag ID
------------------------------

`POST /tags/123/untag.json` will remove an existing tag from a contact.

Replace `123` with the id of the tag.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

**Response:**

```json
{
  "success": true,
  "tag_found": true
}
```

`tag_found` is `false` when the contact exists but does not currently have that tag.


Custom domains
==============

Check whether a domain is configured
------------------------------------

`GET /domain_configured_in_simplero.json?domain=example.com` will check whether `www.example.com` is configured in an active Simplero account.

This endpoint does not require API authentication.

**Successful response:**

```json
{
  "message": "domain is configured"
}
```

**Not found response:**

```json
{
  "message": "domain is not configured"
}
```

The endpoint returns HTTP status 200 when the domain is configured and 404 when it is not.


Broadcasts
==========

Get broadcasts
--------------

`GET /broadcasts.json` will return a paginated list of broadcasts.

**Parameters:**

- `page` — 0-indexed page number (default 0)
- `per_page` — results per page, 1–100 (default 20)
- `from` / `to` — filter by deliver_at (ISO 8601)

**Response:**

```json
[
  {
    "id": 123,
    "subject": "Weekly newsletter",
    "name": "Newsletter #42",
    "state": "delivered",
    "delivery_type": "immediately",
    "deliver_at": "2026-01-15T10:00:00.000-05:00",
    "delivered_at": "2026-01-15T10:01:23.000-05:00",
    "sender_name": "Calvin",
    "sender_email": "calvin@simplero.com",
    "reply_to": null,
    "recipients_count": 1000,
    "sent_count": 998,
    "delivered_count": 995,
    "opens_count": 400,
    "unique_opens_count": 350,
    "clicks_count": 100,
    "unique_clicks_count": 80,
    "bounces_count": 3,
    "unsubscribes_count": 2,
    "spamreports_count": 0,
    "open_rate": 35.18,
    "click_rate": 8.04,
    "bounce_rate": 0.3,
    "unsubscribe_rate": 0.2,
    "created_at": "2026-01-14T15:30:00.000-05:00",
    "updated_at": "2026-01-15T10:01:23.000-05:00"
  }
]
```

Get broadcast by ID
-------------------

`GET /broadcasts/123.json` will get a broadcast by its ID or token.

Responds with the broadcast object, like above. Will respond with 404 if no such broadcast exists.


Create broadcast
----------------

`POST /broadcasts.json` will create a new broadcast.

**POST request body:**

```json
{
  "subject": "Weekly newsletter",
  "body": "<h1>Hello!</h1><p>Here is your weekly update.</p>",
  "sender_name": "Calvin",
  "sender_email": "calvin@simplero.com",
  "reply_to": "replies@simplero.com",
  "email_template_id": 456,
  "list_ids": [1, 2],
  "segment_ids": [3],
  "delivery_type": "later",
  "deliver_at": "2026-01-20T10:00:00-05:00"
}
```

`subject` is required. All other fields are optional.

`delivery_type` can be `now` (send immediately), `later` (schedule for `deliver_at`), or omitted to save as a draft. When `delivery_type` is `later`, `deliver_at` is required (ISO 8601).

Returns 201 with the broadcast object on success. Returns 422 with `{ "error": "..." }` on validation errors. Returns 403 if email sending is not enabled for the account.


Send test broadcast
-------------------

`POST /broadcasts/123/send_test.json` will send a test email for the broadcast.

**POST request body:**

```json
{
  "email": "test@example.com"
}
```

`email` is required.

**Response:**

```json
{
  "success": true,
  "message": "Test email sent to test@example.com"
}
```

Returns 422 with `{ "error": "..." }` if email is missing or sending fails.


Get broadcast activity
----------------------

`GET /broadcasts/123/activity.json` will return engagement stats for the broadcast.

**Response:**

```json
{
  "opened": [
    { "id": 1046901344, "date": "2026-01-15T10:05:00.000-05:00" }
  ],
  "clicked": [
    { "id": 1046901344, "date": "2026-01-15T10:06:30.000-05:00" }
  ],
  "bounced": [],
  "unsubscribed": [],
  "spamreported": []
}
```

Each action type contains an array of contacts who performed that action, with their contact ID and the date of the action.


Invoices
========

`GET /invoices.json` will return invoice info. You can use the following parameters to filter:

- `created_at_from` - ISO-8601 date/time, return invoices created at or after this time
- `created_at_to` - ISO-8601 date/time, return invoices created before this time
- `paid_at_from` - ISO-8601 date/time, return invoices paid at or after this time
- `paid_at_to` - ISO-8601 date/time, return invoices paid before this time
- `invoice_number_from` - String, return invoices with an invoice number alphabetically equal to or greater than this number
- `invoice_number_to` - String, return invoices with an invoice number alphabetically less than this number
- `dir` - String, case-insensitive: Sort order. 'asc' or 'desc'. Default 'asc'.
- `page` - 1-indexed page number. 20 invoices are returned at a time.

The invoices will be ordered by invoice_number in the order described by 'dir', default ascending.

20 invoices will be returned at a time. You can paginate using the `page` parameter.

NOTE: Prior to August 15, 2022, this call would also include unpaid charges, and would order randomly, or ascending if you provided the `order_by_invoice_number` parameter.
Affiliates
==========

`GET /affiliates.json` will return all affiliates for the account.

**Response:**

```json
[
  {
    "id": 123,
    "name": "Jane Affiliate",
    "ref": "jane-affiliate"
  }
]
```


Point types
===========

`GET /point_types.json` will return all point types available on the account. These IDs are what you use with [`POST /customers/issue_points.json`](#issue-points).

**Response:**

```json
[
  {
    "id": 12,
    "name": "Reward points"
  }
]
```


Administrators
========
Get administrators
---------
`GET /administratorships.json` will return all the account's administratorships
**Response:**

```json
[
  {
    "id": 79565,
    "user_id": 289386,
    "account_id": 67112,
    "created_at": "2022-08-27T14:52:34.000-04:00",
    "updated_at": "2023-09-04T13:32:48.000-04:00",
    "num_logins": 3,
    "last_login_at": "2022-09-28T16:11:05.000-04:00",
    "second_to_last_login_at": "2022-09-27T11:14:11.000-04:00",
    "ticket_assignee": true,
    "ticket_sms_alerts": false,
    "show_on_ticket": false,
    "notifications_seen": [],
    "admin_role_id": 9,
    "accessible_object_ids": {
      "site": "all",
      "worksheet": "all",
      "course": "all",
      "affiliate_program": "all",
      "landing_page": "all",
      "product": "all"
    },
    "invitation_id": null,
    "enabled": true,
    "can_self_grant_super_admin": false,
    "admin_role": {
      "name": "Support specialist"
    }
  },
    {
      "id": 79567,
      "user_id": 2063600,
      "account_id": 67112,
      "created_at": "2022-08-27T17:17:24.000-04:00",
      "updated_at": "2023-09-05T09:49:12.000-04:00",
      "num_logins": 104,
      "last_login_at": "2023-09-05T09:49:12.000-04:00",
      "second_to_last_login_at": "2023-09-04T13:19:32.000-04:00",
      "ticket_assignee": true,
      "ticket_sms_alerts": false,
      "show_on_ticket": false,
      "notifications_seen": [],
      "admin_role_id": 1,
      "accessible_object_ids": {
          "site": "all",
          "worksheet": "all",
          "course": "all",
          "affiliate_program": "all",
          "landing_page": "all",
          "product": "all"
      },
      "invitation_id": null,
      "enabled": true,
      "can_self_grant_super_admin": false,
      "admin_role": {
          "name": "Co-Owner"
      }
    }
]
```

Get administrator by ID
-----------------

`GET /administratorships/123.json` will get a JSON representation of an administratorship, looked up by the internal ID.

Replace `123` with the ID of the administratorship you're interested, which you will have gotten from a previous call to the API.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.

Find administrator by email
---------------------

`POST /administratorships/find.json` will get a JSON representation of an administratorship, looked up by email.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

Responds with the administratorship object, like above. Will respond with 404 if no such contact exists.

Create/update administrator
--------------

`POST /administratorships.json` will create a new administratorship, or update an existing administratorship with the same email.

**POST request body:**

```json
{
  "email":           "calvin@simplero.com",
  "admin_role_id":   9,
  "ticket_assignee": true,
  "show_on_ticket":  true,
  "autogenerate":    true,
  "invitee_name":    "Calvin Correli",
  "inviter_email":   "angel@simplero.com",
  "message":         "Welcome Calvin"
}
```

The only required arguments are `email` and `admin_role_id`.

`admin_role_id` can be one of the shared system roles, listed below, or one of the account's saved custom roles.

System roles with their ID:
```
"Co-Owner" => 1
"Admin" => 2
"Basic admin" => 3
"Assistant" => 8
"Support specialist" => 9
"Affiliate manager" => 10
```

`autogenerate` if you want to auto-generate a username and password for the admin.

`invitee_name` name of the new user if `autogenerate` is true

`ìnviter_email` email of one of the account administrators. If not provided, the inviter will be the account owner.

`message` optional message for the invitation.

`ticket_assignee` and `show_on_ticket`  enable whether the admin can be assigned tickets and listed as a support team member in the tickets system (if you have Support tickets enabled in your account).

When updating an existing administratorship, the relevant arguments are `email`, `admin_role_id`, `ticket_assignee`, and `show_on_ticket`.

Remove administrator
--------------
`DELETE /administratorships/123.json` will delete an administratorship with the given ID.


Replace `123` with the ID of the administratorship you want to remove, which you will have gotten from a previous call to the API.

Admin Roles
--------------
`GET /admin_roles.json` will return all the account's available admin roles, including saved custom roles (only available on the Skyrocket plan).
```json
[
  {
    "id": 1,
    "name": "Co-Owner"
  },
  {
    "id": 2,
    "name": "Admin"
  },
  {
    "id": 3,
    "name": "Basic admin"
  },
  {
    "id": 8,
    "name": "Assistant"
  },
  {
    "id": 9,
    "name": "Support specialist"
  },
  {
    "id": 10,
    "name": "Affiliate manager"
  },
  {
    "id": 67,
    "name": "Custom Saved Role"
  },
  {
    "id": 77,
    "name": "Developer Role"
  }
]
```
Automations
===========

Get automations
---------------

`GET /automations.json` will return the account's active automations.

**Parameters:**

- `page` — 0-indexed page number (default 0)

20 automations are returned per page.

**Response:**

```json
[
  {
    "id": 123,
    "name": "Welcome sequence"
  }
]
```

Start automation for a contact
------------------------------

`POST /automations/123/start.json` will start the automation for a contact.

Replace `123` with the id of the automation.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

`email` is the recommended way to identify the contact. If the contact does not exist yet, Simplero will create it first.

**Response:**

```json
{
  "success": true
}
```

If neither a contact email nor a valid contact identifier is provided, the endpoint returns HTTP status 400 with:

```json
{
  "success": false,
  "reason": "customer_not_found"
}
```


Segments
========

`GET /segments.json` will return the account's saved segments.

**Response:**

```json
[
  {
    "id": 123,
    "name": "Customers with active subscriptions",
    "members_count": 432,
    "calculated_at": "2026-01-15T10:01:23.000-05:00"
  }
]
```


Assets
======

`POST /assets.json` will upload a file into the account's assets library.

Send this request as `multipart/form-data`, with the file in the `file` parameter.

**Example request:**

```shell
curl -u "API_KEY:" \
  -H 'User-Agent: Some app (test@example.test)' \
  -F "file=@/path/to/image.png" \
  https://simplero.com/api/v1/assets.json
```

**Response:**

```json
{
  "id": 123,
  "filename": "image.png",
  "content_type": "image/png",
  "size": 123456,
  "width": 1200,
  "height": 628,
  "url": "https://assets.simplero.com/...",
  "image_url": "https://assets.simplero.com/..."
}
```

`image_url` is present for ready image assets. For non-image assets it will be `null`.

If `file` is missing, the endpoint returns HTTP status 422 with:

```json
{
  "error": "File is required"
}
```


Account and Zapier helper endpoints
===================================

These endpoints are mainly used when connecting Simplero to Zapier or when building tooling that needs account-specific field metadata and sample payloads.

Get account custom fields
-------------------------

`GET /account/fields.json` will return the account's contact form fields.

**Response:**

```json
[
  {
    "label": "Instagram handle",
    "internal_name": "instagram_handle",
    "method_name": "field_123_value",
    "field_type": "Text field",
    "subfields": [
      {
        "name": "value",
        "method_name": "field_123_value"
      }
    ]
  }
]
```

Who am I for Zapier
-------------------

`GET /account/zapier_who_am_i.json` will return base account and user information for a Zapier connection test.

**Response:**

```json
[
  {
    "id": 12345,
    "account": {
      "name": "My Simplero account",
      "host": "myaccount.simplero.com"
    },
    "user": {
      "name": "Calvin Correli",
      "username": "calvin"
    }
  }
]
```

Get Zapier customer action fields
---------------------------------

`GET /account/zapier_customer_action_fields.json` will return the contact custom fields in Zapier's action-field format.

**Response:**

```json
[
  {
    "type": "unicode",
    "key": "field_123_value",
    "label": "Instagram handle",
    "required": false
  }
]
```

Complex field types may produce multiple entries, for example address and phone subfields.

Poll sample customers for new subscriptions
-------------------------------------------

`GET /account/zapier_new_subscription_poll.json` will return sample customer payloads for the "new subscription" Zapier trigger.

Optionally pass `list_id` to bias the sample data toward a specific list.

**Response:**

```json
{
  "results": [
    {
      "id": 42,
      "note": null,
      "email": "john@doe.com",
      "name": "John Doe",
      "tag_names": ""
    }
  ]
}
```

Poll sample customers for deleted subscriptions
-----------------------------------------------

`GET /account/zapier_delete_subscription_poll.json` returns the same payload shape as the new subscription poll endpoint.

Poll sample purchases for new purchases
---------------------------------------

`GET /account/zapier_new_purchase_poll.json` will return sample purchase payloads for the "new purchase" Zapier trigger.

**Response:**

```json
{
  "results": [
    {
      "purchase_id": 10000001,
      "product_id": 200000,
      "product_name": "Sample Product Name",
      "state": "paid",
      "email": "user@example.com"
    }
  ]
}
```

Poll sample purchases for canceled purchases
--------------------------------------------

`GET /account/zapier_cancel_purchase_poll.json` returns the same payload shape as the new purchase poll endpoint.

Poll sample customers for new taggings
--------------------------------------

`GET /account/zapier_new_tagging_poll.json` will return sample customer payloads for the "new tagging" Zapier trigger.

Optionally pass `tag_id` to bias the sample data toward a specific tag.

The response shape is the same as [`GET /account/zapier_new_subscription_poll.json`](#poll-sample-customers-for-new-subscriptions).

Poll sample customers for deleted taggings
------------------------------------------

`GET /account/zapier_delete_tagging_poll.json` returns the same payload shape as the new subscription poll endpoint.


Zapier subscriptions
====================

These endpoints are used to create and remove webhook subscriptions for the public Simplero Zapier app.

Create or update a Zapier subscription
--------------------------------------

`POST /zapier_subscriptions.json` will create a new Zapier subscription or update the existing one for the same `target_url`.

**POST request body:**

```json
{
  "event": "new_subscription",
  "target_id": 123,
  "target_url": "https://hooks.zapier.com/hooks/catch/123456/abcdef/"
}
```

Supported `event` values are:

- `new_subscription`
- `delete_subscription`
- `new_purchase`
- `cancel_purchase`
- `new_tagging`
- `delete_tagging`

For subscription events, `target_id` should be the list ID.

For tagging events, `target_id` should be the tag ID.

For purchase events, subscriptions are account-wide and `target_id` is not used.

**Response:**

```json
{
  "id": 98765,
  "event": "new_subscription",
  "target_id": 123,
  "target_type": "List"
}
```

Destroy a Zapier subscription
-----------------------------

`DELETE /zapier_subscriptions/98765.json` will remove a Zapier subscription.

The `id` can be the current Simplero subscription ID. Legacy numeric IDs from older Zapier subscriptions are still accepted as well.

**Response:**

```json
{
  "id": 98765,
  "event": "new_subscription",
  "target_id": 123,
  "target_type": "List",
  "target_url": "https://hooks.zapier.com/hooks/catch/123456/abcdef/"
}
```


Asynchronous requests
--------------

Some API requests are asynchronous and do not complete immediately. You can use the requests endpoint to check the
status of these requests.

`GET /requests/ABC123.json` will provide status on the request. Replace `ABC123` with the token you got from the
original request.

```json
{
     "token": "ABC123",
     "action": "lists/bulk_subscribe",
     "status": "success",
     "enqueued_at": "2023-09-05T09:49:12.000-04:00",
     "processing_started_at": "2023-09-05T09:49:12.000-04:00",
     "completed_at": "2023-09-05T09:49:12.000-04:00",
     "result": [
          /* ... */
     ]
```

`status` can be:
- `pending` - request has not started
- `running` - request is being processed
- `success` - request completed successfully
- `failure` - request failed

`result` will be populated with action-specific data.

Webhook endpoint
================

If you're using some other system for selling, and want to hook it up so you can automatically add people to your products in Simplero, which will give them access to the space, this is what you need.

The URL is `POST https://simplero.com/webhook/products/1/purchase`

Replace the number 1 with the ID of the product you want to add people to. You can find that in the admin URL.

You must include the following POST parameters:

* `api_key` - the API key found in your admin interface under Settings > API keys
* `first_names` - first name
* `last_name` - last name
* `email` - email address

We'll take it from there, hook people up with a Simplero ID, email them the auto-responders you've set up, and do anything else that needs to happen.

**Please note:** purchases, as represent a given user's access to Simplero, cannot be created on trial accounts. You must activate the account before purchases can be made.
