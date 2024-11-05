# Fediscovery: Fediverse Discovery Providers

## 02: Content Ingestion

In order to be able to present results, discovery providers need to be
able to request content from connected fediverse servers. Two different
concerns need to be distinguished: Retrieving new content as it is
created and retrieving historic content to backfill search indexes.

### Restrictions On Content Being Shared

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
public. In the case of account and post data it MUST make sure to only
share content where the creator has opted in to discovery.

The FASP in turn MUST also ensure that creators have opted in to
discovery before storing and indexing content.

Both parties MUST use the mechanism described in
[FEP-5feb](https://codeberg.org/fediverse/fep/src/branch/main/fep/5feb/fep-5feb.md)
to determine if a creator has opted in to discovery.

### Subscribing To, Requesting And Receiving Content

#### Managing Subscriptions (FASP => Fediverse Server)

In order to subscribe to new content, FASPs can subscribe to events
by making a `POST` call to the `/discovery/event_subscriptions`
endpoint on the fediverse server.

Example call:

```http
POST /discovery/event_subscriptions
```

The request body MUST contain a JSON object defining what events to
subscribe to. This object MUST contain the keys `objectType` and
`eventType`. It MAY contain the keys `maxBatchSize` and/or
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
* `threshold`: If `eventType` is `trending` then this object can be used to
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
  "subscriptionId": 3446
}
```

To unsubscribe the FASP can make an HTTP `DELETE` call to the
`/discovery/event_subscriptions/:id` endpoint, where `:id` is replaced
with the ID received when the original subscription was created.

Example call:

```http
DELETE /discovery/event_subscriptions/3446
```

The response MUST be an HTTP status code `200` (OK).

#### Requesting Historic Content / Backfilling (FASP => Fediverse Server)

To request historic content to index a FASP can make an HTTP `POST` call
to the `/discovery/backfill_requests` endpoint.

Example call:

```http
POST /discovery/backfill_requests
```

The request body MUST contain a JSON object with the following keys:

* `category`: One of `content`, `account`. This is the category
  of objects the FASP is interested in.
* `maxCount`: An integer representing the maximum number of objects the
  FASP would like to receive.
* `cursor`: Optional. An opaque token received from the fediverse server
  in a previous request. MUST be used by the fediverse server to
  determine results that have not already been sent to the FASP.

Example object:

```json
{
  "category": "Note",
  "maxCount": 100,
  "cursor": "1541815103606536472"
}
```

The response from the fediverse server MUST include an HTTP status code
`201` (Created) and a JSON object including the `id` of the generated
backfill request.

Example response object:

```json
{
  "backfillRequestId": 672
}
```

#### Sending Content (Fediverse Server => FASP)

Both the occurrence of an event that a FASP has subscribed to and the
fulfillment of a backfill request, MUST be announced to the FASP by the
fediverse server with an HTTP `POST` call to the FASP's
`/discovery/announcements` endpoint.

Example call:

```http
POST /discovery/announcements
```

The request body MUST include a JSON object including the keys
`source`, `objectType` and `objectUris`.

* `source` lets the FASP know in reply to which of its request this
  announcement is sent. It MUST include an object with either a
  `subscriptionId` or a `backfillRequestId`.
* `category` MUST mirror the category given in the original
  subscription or backfill request.
* `objectUris` is an array of URIs representing the objects.
* `eventType` MUST be present for events and be one of `new`, `update`,
  `delete` or `trending`. The first three are for subscriptions to the
  `lifecycle` type and the latter for the `trends` type.
  This key MUST NOT be present for responses to backfill requests.
* `cursor` MAY optionally be present on responses to backfill requests.
  If it is string its value can be included in a future backfill request
  to make sure the fediverse server does not send the same object uris
  again.  If this is a partial announcement that does not conclude an
  open backfill request this key MUST be absent.
* `moreObjectsAvailable` MAY optionally be present on responses to
  backfill requests. If set to `false` it means that no more objects of
  this category are available and no future backfill requests should be
  made.

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
    "subscriptionId": 58152
  },
  "eventType": "new",
  "objectType": "Note",
  "objectUris": [
    "https://fediverse-server.example.com/@example/2342",
    "https://fediverse-server.example.com/@other/8726"
  ]
}
```

#### Retrieving Content From Its Origin (FASP => Wider Fediverse)

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
  "id": "<Base URL>/actor",
  "type": "Application",
  "publicKey": {
    "id": "<Base URL>/actor#main-key",
    "owner": "<Base URL>/actor",
    "publicKeyPem": "<Public key PEM>"
  }
}
```

`<Base URL>` MUST be replaced with the base URL of the FASP, `<Public
key PEM` with its public key used for signing.

FASP MUST include the content shown above in their actor JSON but MAY
add further information.

Note that the JSON shown above is not a valid ActivityPub actor because
it is missing an "inbox" and an "outbox". The information contained is enough
to verify signatures though and it seems preferrable to not confuse
anyone with "fake" in- and outboxes. But this is subject to change if it
turns out that fediverse software exists that only accepts fetch
requests from actors that do have an inbox and an outbox.

---

Next: [03: Trends](03_trends.md)