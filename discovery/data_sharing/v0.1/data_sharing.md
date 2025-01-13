# "Fediscovery": Data Sharing Specification

## Overview

Specification identifier: `data_sharing`

In order to be able to present results, discovery providers need to be
able to request content from connected fediverse servers. Two different
concerns need to be distinguished: Retrieving new content as it is
created and retrieving historic content to backfill search indexes.

## Restrictions On Content Being Shared

A single FASP will never be used by all fediverse servers. To ensure it
still gets a broad view not limited to a small number of servers,
fediverse servers MUST share not only local but also remote content with
the FASP.

Since this may result in multiple servers sharing the same content with
a single FASP, FASPs MUST deduplicate content.

As a single server is not authoritative for all the content shared and
to save bandwidth only URIs of content are actually exchanged with the
FASP. The FASP is then responsible for retrieving the content and
working with it. The exact details of this differ between capabilties
and are described in the respective sections.

A fediverse server MUST make sure it only shares content that it is
actually allowed to share. It MUST NOT share any content that is not
public.

Some fediverse software projects (including Mastodon) have more
differentiated visibility settings and offer public visibility of posts
that is somewhat restricted. In the case of Mastodon this is called
"quiet public" or "unlisted". Posts with this visibility are publicly
available but should not be included in search. If a fediverse server
supports this kind of restricted public visibility it MUST NOT share
this kind of content with FASP.

FASP MUST check the `to:` property on content it retrieves to make sure
it is really meant for public consumption.

In the case of account and post data, a fediverse server MUST make sure
to only share content where the creator has opted in to discovery.

The FASP in turn MUST also ensure that creators have opted in to
discovery before storing and indexing content.

Both parties MUST use the mechanism described in
[FEP-5feb](https://codeberg.org/fediverse/fep/src/branch/main/fep/5feb/fep-5feb.md)
to determine if a creator has opted in to discovery.

For account data, both parties MUST ensure the `discoverable` flag is
set to `true`.

## Subscribing To, Requesting And Receiving Content

### Managing Subscriptions (FASP => Fediverse Server)

In order to subscribe to new content, FASPs can subscribe to events
by making a `POST` call to the `/data_sharing/v0/event_subscriptions`
endpoint on the fediverse server.

Example call:

```http
POST /data_sharing/v0/event_subscriptions
```

The request body MUST contain a JSON object defining what events to
subscribe to. This object MUST contain the keys `category` and
`subscriptionType`. It MAY contain the keys `maxBatchSize` and/or
`threshold`. The keys are defined as follows:

* `category`: One of `content`, `account`. This is the category of
  objects the FASP is interested in.
* `subscriptionType`: One of `lifecycle`, `trends`. `lifecycle`
  means the FASP would like to be notified when new objects of the given
  type are created, when objects are updated and when objects are
  deleted. `trends` means that the FASP would like to be notified of
  objects that had recent interactions. `trends` is only applicable if
  the `category` given is `content`.
* `maxBatchSize`: The maximum number of events the FASP would like to
  receive in a single request. This is optional.
* `threshold`: If `subscriptionType` is `trends` then this object can be used to
  further specify when the event should be reported. Valid keys for this
  object are:
  * `timeframe`: Number of minutes in which interactions should fire an
    event, defaults to `15`
  * `shares`: Number of shares in the given timeframe, defaults to `3`
  * `likes`: Number of likes in the given timeframe, defaults to `3`
  * `replies`: Number of replies in the given timeframe, defaults to `3`

Example object:

```json
{
  "category": "content",
  "subscriptionType": "trends",
  "maxBatchSize": 10,
  "threshold": {
    "timeframe": 15,
    "shares": 3
  }
}
```

The fediverse server MUST validate the request. If it is invalid it MUST
return an HTTP status code `422` (Unprocessable Content).

If it is valid the response from the fediverse server MUST include an
HTTP status code `201` (Created) and a JSON object including the `id` of
the generated subscription. The latter can be used to unsubscribe later.

Example response object:

```json
{
  "subscription": {
    "id": "3446"
  }
}
```

To unsubscribe the FASP can make an HTTP `DELETE` call to the
`/data_sharing/v0/event_subscriptions/:id` endpoint, where `:id` is replaced
with the ID received when the original subscription was created.

Example call:

```http
DELETE /data_sharing/v0/event_subscriptions/3446
```

The response MUST be an HTTP status code `204` (No Content).

### Requesting Historic Content / Backfilling (FASP => Fediverse Server)

FASP can request historic content from fediverse servers. Individual
requests are always for a limited number of individual records and will
be fulfilled asynchronously using the same delivery mechanism as event
subscriptions. This way both parties have control over how to spread out
the additional load generated by backfill requests.

To request historic content to index a FASP can make an HTTP `POST` call
to the `/data_sharing/v0/backfill_requests` endpoint.

Example call:

```http
POST /data_sharing/v0/backfill_requests
```

The request body MUST contain a JSON object with the following keys:

* `category`: One of `content`, `account`. This is the category
  of objects the FASP is interested in.
* `maxCount`: An integer representing the maximum number of objects the
  FASP would like to receive.

Example object:

```json
{
  "category": "content",
  "maxCount": 100
}
```

The response from the fediverse server MUST include an HTTP status code
`201` (Created) and a JSON object including the `id` of the generated
backfill request.

Example response object:

```json
{
  "backfillRequest": {
    "id": "672"
  }
}
```

Data received as a result of a backfill request will indicate if more
content for this request is available (see the next section for
details). In this case FASP can make an HTTP `POST` request to the
`/data_sharing/v0/backfill_requests/{id}/continuation` endpoint, where
`{id}` is the identifier of the backfill request as received from the
server. The request body MUST be empty.

Example call:

```http
POST /data_sharing/v0/backfill_requests/672/continuation
```

The response from the fediverse server MUST include an HTTP status code
`204` (No Content) if more content for the backfill request is
available. If no more content is available or if the `id` is not known
it MUST instead use the status code `404` (Not Found).

FASP SHOULD NOT issue this request until they received content that
indicates that more is available for a given backfill request.

### Sending Content (Fediverse Server => FASP)

Both the occurrence of an event that a FASP has subscribed to and the
fulfillment of a backfill request, MUST be announced to the FASP by the
fediverse server with an HTTP `POST` call to the FASP's
`/data_sharing/v0/announcements` endpoint.

Example call:

```http
POST /data_sharing/v0/announcements
```

The request body MUST include a JSON object including the keys
`source`, `objectType` and `objectUris`.

* `source` lets the FASP know in reply to which of its request this
  announcement is sent. It MUST include an object with either the
  key `subscription` or `backfillRequest`. This in turn MUST include
  an object with the key `id` containing the identifier of the given
  source.
* `category` MUST mirror the category given in the original
  subscription or backfill request.
* `objectUris` is an array of URIs representing the objects.
* `eventType` MUST be present for events and be one of `new`, `update`,
  `delete` or `trending`. The first three are for subscriptions to the
  `lifecycle` type and the latter for the `trends` type.
  This key MUST NOT be present for responses to backfill requests.
* `moreObjectsAvailable` MUST be present on responses to backfill
  requests. If set to `false` it means that no more objects of this
  category are available and no future backfill requests should be made.

Requests MUST include at least one object URI, but the fediverse server
MAY save requests and batch events together. In that case it MUST
respect the `maxBatchSize` it received with the subscription.

In case of backfill requests the fediverse server can decide to send the
requested number of objects in a single request or split it up into
smaller requests to reduce the load. In that case the optional `cursor`
MUST only be included with the final request, but only if more data
would be available.

Example payload:

```json
{
  "source": {
    "subscription": {
      "id": "58152"
    }
  },
  "category": "content",
  "eventType": "new",
  "objectUris": [
    "https://fediverse-server.example.com/@example/2342",
    "https://fediverse-server.example.com/@other/8726"
  ]
}
```

The response MUST be an HTTP status code `204` (No Content).

## Retrieving Content From Its Origin (FASP => Wider Fediverse)

As displayed in the sections above, FASP will only receive object URIs
from connected fediverse servers. They will then need to fetch the
actual content themselves.

To do so, they need to send HTTP `GET` requests to the object URIs with
an `Accept` header with the `application/ld+json;
profile="https://www.w3.org/ns/activitystreams"` media type as specified
by [ActivityPub](https://www.w3.org/TR/activitypub/#retrieving-objects).

These requests MUST be signed. To achieve a maximum of compatibility
with existing fediverse software, FASP MUST support request signing with
both "HTTP Message Signatures" as defined by
[RFC-9421](https://tools.ietf.org/html/rfc9421.html) as well as the
earlier draft version "HTTP Signatures" as defined by
[cavage-12](https://datatracker.ietf.org/doc/html/draft-cavage-http-signatures).

To find out which version a given fediverse server supports, FASP should
implement "double-knocking": They should first attempt a request using
"HTTP Message Signatures" and if the fediverse server replies with an
HTTP status code of `401` or `403` make a second attempt with the older
draft version, "HTTP Signatures".

To save on future requests, FASP SHOULD implement a per fediverse server
cache to save the information which version of the standard was accepted
and use that one directly when accessing objects the next time. In case
a server supports the older version, "HTTP Signatures", the newer one
MUST be retried periodically, i.e. the cache should be invalidated.

More information about HTTP signatures and "double-knocking" can be
found in this report:
[ActivityPub and HTTP Signatures](https://swicg.github.io/activitypub-http-signature/).

In both protocol versions, FASP MUST act as an "server/instance actor".
This means that the `keyId` given MUST be a URI that can be used to
fetch the JSON-LD representation of an ActivityPub Actor that represents
the FASP. This actor information MUST include the public key that can
be used to verify the signature.

The URI's path MUST be `/actor` and a minimal JSON response might look
like this:

```json
{
  "@context": [
    "https://www.w3.org/ns/activitystreams",
    "https://w3id.org/security/v1",
  ],
  "id": "https://fasp.example.com/actor",
  "type": "Application",
  "inbox": "https://fasp.example.com/inbox",
  "outbox": "https://fasp.example.com/outbox",
  "preferredUsername": "fasp",
  "publicKey": {
    "id": "https://fasp.example.com/actor#main-key",
    "owner": "https://fasp.example.com/actor",
    "publicKeyPem": "<Public key PEM>"
  }
}
```

`https://fasp.example.com` MUST be replaced with an URL of the actual
FASP, `<Public key PEM>` with its public key used for signing. Note that
is is *not* one of the private keys used to authenticate with a
registered fediverse server as defined in [FASP/Fediverse Server
Interaction](../../general/v0.1/) but part of a separate key pair FASP
MUST generate for this purpose.

FASP MUST include the content shown above in their actor JSON but MAY
add further information.

ActivityPub mandates that an `inbox` and `outbox` URL are present. A
minimal implementation of an outbox could simply always return an empty
collection while a minimal inbox could receive but immediately discard
activities posted to it. Note that fediverse servers might post
activities to the inbox that could be useful for FASP. But if and how
these endpoints are implemented in FASP is out of scope for this
specification.

FASP MUST give a `preferredUsername` as some fediverse servers require
this. This also means they need to support "WebFinger" lookups for this
username as defined in
[RFC-7033](https://datatracker.ietf.org/doc/html/rfc7033).

In the example above this would mean that the FASP would need to respond
to an HTTP `GET` request to
`https://fasp.example.com/.well-known/webfinger?resource=acct:fasp@fasp.example.com`
accepting a `Content-Type` of `application/jrd+json` with something like
this:

```json
{
  "subject": "acct:fasp@fasp.example.com",
  "aliases": [
    "https://fasp.example.com/actor"
  ],
  "links": [
    {
      "rel": "self",
      "type": "application/activity+json",
      "href": "https://fasp.example.com/actor"
    }
  ]
}
```

Once indexed or persisted in any way, FASP MUST periodically re-check
both content and account data. At least once every week FASP MUST
revalidate that the content / account is still publicly available and
still allowed to be indexed. If the content has been changed these
changes MUST be applied in the FASPs data storage as well. If FASP have
been notified of changes through their subscriptions they MAY suspend
the periodical check for this object for the next week.
