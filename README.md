Simplero API Documentation
==========================

The API is REST, using JSON for serialization, with no root element.

You'll notice right away that this API is woefully incomplete. If you need anything, please ask.
I'm Calvin Conaway (calvin@conaway.com), the founder, CEO, and creator of zenbilling, and I'm always open
to your input, and we're flexible about adding additional API calls.

The hostname to use for API calls is simplero.com, not zenbilling.com.

zenbilling will be rebranding as Simplero shortly, and we might as well be forward-looking with the API.


Making a request
----------------

All API request URLs start with `https://simplero.com/api/v1/`. 

All requests have to be over HTTPS.


Authentication
--------------

Requests are authenticated using HTTP Basic Auth, with the API key as the username, password left empty.

Failure to provide a valid API key will result in a status 401 (unauthorized) with a JSON body of:

```json
{ "error": "Bad API key" }
```


Identify your appliaction in the User-Agent string
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
curl -u "api_key:" -H 'User-Agent: Some app (test@example.test)' https://simplero.com/api/v1/lists.json
```

To add or update something, include the JSON data with a `Content-Type` header:

```shell
curl -u "api_key:" -H 'Content-Type: application/json' -H 'User-Agent: Some app (email@example.test)' \
  -d '{ "first_name": "New Subscriber", "email": "someemail@example.test" }' \
  https://simplero.com/api/v1/lists/1/subscribe.json
```



Lists
=====

Get lists
---------

* `GET /lists.json` will return all the account's lists

```json
[
  {
    "id":1942123930,
    "name":"Marianne's Universe",
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


Subscribe to list
-----------------

* `POST /lists/1/subscribe.json` will add a new subscriber to the list

```json
{
  "first_name":              "Calvin", 
  "last_name":               "Conaway",
  "email":                   "calvin@conaway.com", 
  "ip_address":              "77.66.17.105",
  "referrer":                "http://google.com/search?q=foo",
  "ref":                     123,
  "track":                   "api",
  "first_activated_at":      "2012-03-22T17:35:50-05:00",
  "auto_responder_start_at": "2012-03-22T17:35:50-05:00",
  "landing_page_id":         123
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

