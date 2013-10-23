Simplero API Documentation
==========================

The API is REST, using JSON for serialization.

Base end-point for API requests are:

https://simplero.com/api/v1/...

All requests must be over SSL.

We do not include a root element in our JSON responses.



Lists
=====

Get lists
---------

* `GET /lists.json` will return all the account's lists

```json
[
  {
    "id": 605816632,
    "name": "BCX",
    "description": "The Next Generation",
    "updated_at": "2012-03-23T13:55:43-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/605816632-bcx.json",
    "archived": false,
    "starred": true
  },
  {
    "id": 684146117,
    "name": "Nothing here!",
    "description": null,
    "updated_at": "2012-03-22T16:56:51-05:00",
    "url": "https://basecamp.com/999999999/api/v1/projects/684146117-nothing-here.json",
    "archived": false,
    "starred": false
  }
]
```
