The API is REST, using JSON for serialization, with no root element.

We also have one webhook endpoint available. See the bottom of this file.

   * [Making a request](#making-a-request)
      * [Authentication](#authentication)
      * [Identify your application in the User-Agent string](#identify-your-application-in-the-user-agent-string)
      * [Example request](#example-request)
      * [Custom contact fields](#custom-contact-fields)
      * [Need anything?](#need-anything)
   * [Contacts](#contacts)
      * [Create/update contact](#createupdate-contact)
      * [Get contact by ID](#get-contact-by-id)
      * [Find contact by email](#find-contact-by-email)
      * [Add tag to contact](#add-tag-to-contact)
      * [Remove tag from contact](#remove-tag-from-contact)
   * [Lists](#lists)
      * [Get lists](#get-lists)
      * [Subscribe to list](#subscribe-to-list)
      * [Unsubscribe from a list](#unsubscribe-from-a-list)
      * [Find subscription by email](#find-subscription-by-email)
   * [Products](#products)
      * [Get products](#get-products)
      * [Get product by ID](#get-product-by-id)
      * [Find purchase by email](#find-purchase-by-email)
   * [Invoices](#invoices)
   * [Administratorships](#administratorships)
      * [Get administratorships](#get-administratorships)
      * [Get administratorship by ID](#get-administratorship-by-id)
      * [Get administratorship by email](#find-administratorship-by-email)
      * [Create or update administratorship](#createupdate-administratorship)
      * [Remove administratorship](#remove-administratorship)
      * [Get admin roles](#admin-roles)
   * [Webhook endpoint](#webhook-endpoint)


Making a request
================

All API request URLs start with `https://simplero.com/api/v1/`. 

All requests have to be over HTTPS.


Authentication
--------------

Requests are authenticated using HTTP Basic Auth, with the API key as the username, password left empty.

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

Have a feature request for the API? Submit it [here](https://requests.simplero.com).



Contacts
========

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
  "field_123_value":         "calvincorreli"
}
```

The only required argument is `email`.

`tags` will add these as additional tags. You cannot remove tags with this API call.

`note` will add a new note to the customer. You cannot remove notes this way.

`phone` is a sanitized phone number with country code first, eg. '15551231234'.

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

Responds with the contact object, like above. Will respond with 404 if no such contact exists.




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


Products
========

Get products
------------

`GET /products.json` will list all the account's products.

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


Invoices
========

`GET /invoices.json` will return invoice info. You can use the following parameters to filter:

- `created_at_from` - ISO-8601 date/time, return invoices created at or after this time
- `created_at_to` - ISO-8601 date/time, return invoices created before this time
- `invoice_number_from` - String, return invoices with an invoice number alphabetically equal to or greater than this number
- `invoice_number_to` - String, return invoices with an invoice number alphabetically less than this number
- `dir` - String, case-insensitive: Sort order. 'asc' or 'desc'. Default 'asc'.

The invoices will be ordered by invoice_number in the order described by 'dir', default ascending.

20 invoices will be returned at a time. You can paginate using the `page` parameter.

NOTE: Prior to August 15, 2022, this call would also include unpaid charges, and would order randomly, or ascending if you provided the `order_by_invoice_number` parameter.

Administratorships
========
Get administratorships
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

Get administratorship by ID
-----------------

`GET /administratorships/123.json` will get a JSON representation of an administratorship, looked up by the internal ID. 

Replace `123` with the ID of the administratorship you're interested, which you will have gotten from a previous call to the API.

Responds with the contact object, like above. Will respond with 404 if no such contact exists.

Find administratorship by email
---------------------

`POST /administratorship/find.json` will get a JSON representation of an administratorship, looked up by email.

**POST request body:**

```json
{
  "email": "calvin@simplero.com"
}
```

Responds with the administratorship object, like above. Will respond with 404 if no such contact exists.

Create/update administratorship
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

`message` optional message for the invitation.

`ticket_assignee` and `show_on_ticket`  enable whether the admin can be assigned tickets and listed as a support team member in the tickets system (if you have Support tickets enabled in your account).

When updating an existing administratorship, the relevant arguments are `email`, `admin_role_id`, `ticket_assignee`, and `show_on_ticket`.

Remove administratorship
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
