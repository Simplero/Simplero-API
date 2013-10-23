Simplero API Documentation
==========================

The API is REST, using JSON for serialization.

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
curl -u "api_key:" -H 'User-Agent: Some app (test@example.test)' https://zenbilling.com/api/v1/lists.json
```

To add or update something, include the JSON data with a `Content-Type` header:

```shell
curl -u "api_key:" -H 'Content-Type: application/json' -H 'User-Agent: Some app (email@example.test)' \
  -d '{ "first_name": "New Subscriber", "email": "someemail@example.test" }' \
  https://zenbilling.com/api/v1/lists/1/subscribe.json
```


Questions? Improvements?
------------------------

Get in touch with zenbilling's creator, Calvin Conaway at calvin@conaway.com.



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
  "email":                   "calvin@conaway.com", 
  "ip_address":              "192.168.1.2",
  "referrer":                "http://google.com/search?q=foo",
  "ref":                     123,
  "track":                   "api",
  "first_activated_at":      "2012-03-22T17:35:50-05:00",
  "auto_responder_start_at": "2012-03-22T17:35:50-05:00",
  "landing_page_id":         123
}
```



