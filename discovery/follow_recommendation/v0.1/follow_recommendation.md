# "Fediscovery": Follow Recommendation Specification

## Overview

Specification identifier: `follow_recommendation`

An important part of discovery is finding people and accounts to follow.
This capability offers an API that allows passing an existing account
and returns a list of accounts that the given user might be interested
in following.

## Query For Follow Recommendations 

To perform a query for follow recommendations, FASP allow fediverse servers
to make an HTTP `GET` call to the `/follow_recommendation/v0/accounts` endpoint.

The following parameters can be used:

* `accountUri`: This parameter MUST be present and include the URI (ID)
  of the account the server wants to have follow recommendations for. 
* `language`: An optional [BCP47](https://tools.ietf.org/html/bcp47)
  language tag to only receive recommendations of accounts posting in
  the specified language. FASP MUST perform "basic filtering" as
  described by
  [RFC-4647](https://tools.ietf.org/html/rfc4647.html) to determine
  matching languages.

Example call to get follow recommendations for
`https://fedi.example.com/users/1`:

```http
GET /follow_recommendation/v0/accounts?accountUri=https%3A%2F%2Ffedi.example.com%2Fusers%2F1
```

For a valid request the response MUST include an HTTP status code `200`
(OK) and a JSON array that includes the recommended accounts URIs (IDs). 

If the `accountUri` parameter is missing from the request, the response
MUST include an HTTP status code `422` (Unprocessable Content).

Example response array:

```json
[
  "https://fedi.example.com/actor/23",
  "https://other.example.com/user/245/actor"
]
```

## Fediverse Server Considerations

Results obtained from FASP might contain any of the following:

* Accounts the user is already following
* Accounts the user has blocked
* Accounts from a domain that is blocked either by the user or the
  server

It is the responsibility of the fediverse server to filter results
accordingly before presenting them to users.

## Privacy Policy Information

With the API specified above fediverse servers do not share any personally
identifiable or otherwise sensitive information with FASP per se.

But depending on the implementation in the fediverse server a request
could mean that the passed user / account has actively requested follow
recommendations. And depending on the publicly available data for the
account it might be personally identifiable.

Fediverse server administrators may add something along the lines of
this to their privacy policy:

> <server> sends requests for follow recommendations to a third-party
> discovery service, <fasp>. This service returns relevant data from the
> fediverse that <server> uses to improve follow recommendations. The
> user's identifier (URI) is passed to the service to personalize
> results. Depending on the publicly available data for the user the
> service may be able to personally identify the user and thus know who
> initiated this request. 
