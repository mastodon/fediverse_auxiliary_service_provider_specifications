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

### Subscribing To And Receiving New Content

#### Managing Subscriptions (FASP => Fediverse Server)

In order to subscribe to new content, FASPS can subscribe to events
by making a `POST` call to the `/discovery/event_subscriptions`
endpoint on the fediverse server.

Example call:

```http
POST /discovery/event_subscriptions
```

The request body MUST contain a JSON object defining what events to
subscribe to. This object MUST contain the keys `object_type` and
`event_type`. It MAY contain the keys `max_batch_size` and/or
`threshold`. The keys are defined as follows:

* `object_type`: One of `Account`, `Post`. This is the type of content
  the FASP would like to receive.
* `event_type`: One of `new`, `update`, `delete`, `trending`. `new`
  means the FASP would like to receive all new objects of the given
  type. `update` means the FASP would like to be notified about updates
  to objects of the given type. `delete` means the FASP would like to be
  notified when an object of the given type is deleted. `trending` means
  that the FASP would like to be notified of objects that had recent
  interactions.
* `max_batch_size`: The maximum number of events the FASP would like to
  receive in a single request. This is optional.
* `threshold`: If `event_type` is `trending` then this object can be used to
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
  "object_type": "Post",
  "event_type": "trending",
  "max_batch_size": 10,
  "threshold": {
    "timeframe": 15,
    "shares": 3
  }
}
```

The response from the fediverse server MUST include an HTTP status code
`201` (Created) and a JSON object including the `id` of the generated
subscription. The latter can be used to unsubscribe later.

Example response object:

```json
{
  "id": 3446
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

#### Sending Events (Fediverse Server => FASP)

When an event occurs that a FASP has subscribed to, the fediverse server
MUST make an HTTP `POST` call to the FASP's `/discovery/events`
endpoint.

Example call:

```http
POST /discovery/events
```

The request body MUST include a JSON object including the keys
`event_type`, `object_type` and `object_uris`.

`event_type` and `object_type` MUST mirror the original subscription.
`object_uris` is an array of URIs representing the objects.

Requests MUST include at least one object URI, but the fediverse server
MAY save requests and batch events together. In that case it MUST
respect the `max_batch_size` it received with the subscription.

Example payload:

```json
{
  "event_type": "new",
  "object_type": "Post",
  "object_uris": [
    "https://fediverse-server.example.com/@example/2342",
    "https://fediverse-server.example.com/@other/8726"
  ]
}
```

### Requesting Historic Content (FASP => Fediverse Server)

To request historic content to index a FASP can make an HTTP `GET` call
to the `/discovery/posts` and `/discovery/accounts` endpoints.

Both behave exactly the same except that the former is for retrieving
posts and the latter for retrieving accounts.

Results MUST be paginated in order to not overload both parties. Both
endpoints MUST support a `per_page` query parameter to control the
maximum number of objects returned per call.

Example query:

```http
GET /posts?per_page=20
```

The response MUST include an HTTP status code `200` (OK) and at most the
number of objects specified in the `per_page` parameter. If more objects
would be available the response MUST include an HTTP `Link` header that
specifies the URI of the next "page" of objects.

Example header:

```http
Link:
<https://fediverse-server.example.com/aux/discovery/posts?per_page=20&cursor=68642346437>;
rel="next"
```

The response payload contains a JSON object with a single key,
`object_uris`, that contains an array of object URIs.

Example:

```json
{
  "object_uris": [
    "https://fediverse-server.example.com/@example/2342",
    "https://fediverse-server.example.com/@other/8726"
  ]
}
```

---

Next: [03: Trends](03_trends.md)
