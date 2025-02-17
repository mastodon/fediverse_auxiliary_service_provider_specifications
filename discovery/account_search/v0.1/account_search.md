# "Fediscovery": Account Search Specification

## Overview

Specification identifier: `account_search`

An important part of discovery is finding people to follow. This
capability offers a very simple API to search for accounts that have
opted-in to being disvovered.

## Performing Full-Fext Searches for Accounts

To perform a full-text search for accounts FASP allow fediverse servers
to make an HTTP `GET` call to the `/account_search/v0/search` endpoint.

The following parameters can be used:

* `term`: This parameter MUST be present and include the text that is to
  be searched for.
* `limit`: This parameter MAY optionally be present to give a positive
  integer number representing the maximum nmber of results FASP should
  return. If omitted this value defaults to `20`.

Example call to search for "teapot" and request at most 10 results:

```http
GET /account_search/v0/search?term=teapot&limit=10
```

If a `term` was present in the request, the response MUST include an
HTTP status code `200` (OK) and a JSON array that includes the URIs
(IDs) of the ActivityPub actor belonging to accounts matching the
requests. These results MUST be sorted by relevance in descending order.

If the `term` is missing from the request, the response MUST include an
HTTP status code `422` (Unprocessable Content).

Example response array:

```json
[
  "https://fedi.example.com/actor/23",
  "https://other.example.com/user/245/actor"
]
```

In case there are more search results than specified by `limit` FASP MAY
allow to paginate more results. To signal that there are more results
FASP MUST include an HTTP `Link` header as described in
[RFC-5988](https://tools.ietf.org/html/rfc5988.html). This header MUST
include the URL of the next page of results with a relation type (`rel`)
of `next`.
