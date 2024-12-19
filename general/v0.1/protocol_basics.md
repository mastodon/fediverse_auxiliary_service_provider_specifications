# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

## 02: Protocol Basics

This document introduces some basic API endpoints FASPs and
fediverser servers need to implement. All other endpoints depend on what services
the FASP offers and are documented in the respective
FASP specifications.

There are some common aspects that are mandatory for all FASPs or
shared between many.

### Definition of Terms and Conventions

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in
this document are to be interpreted as described in
[RFC-2119](https://tools.ietf.org/html/rfc2119.html).

All examples and descriptions of API calls include path names that are
intended to be relative to a given base URL.  Base URLs are discovered /
exchanged during [registration](registration.md).  For the sake of
brevity all examples here and in provider specifications assume the base
URL to not contain any paths. Implementations MUST consider the base URL
and prefix all API paths accordingly if the base URL contains any path
segments.

### General Approach and Building Blocks

This specification aims to define as little new protocol as possible.
Instead, existing protocols and technologies that many fediverse
software projects already use should be re-used wherever
possible. This approach SHOULD be taken by all FASP specifications
that build on this one.

Communication between fediverse server and FASP MUST use HTTPS in production
or production-like settings. This requirement MAY be relaxed in
development environments.

Custom API calls are HTTPS calls sending, if necessary, JSON data
(`Content-Type: application/json`) and receiving JSON data.

### Base URL

As mentioned above both FASP and fediverse software MUST implement the
API endpoints specified here relative to a base URL of their own
choosing. This allows existing fediverse software and existing software
projects that want to add the ability to act as FASP to implement this
without confliciting with their existing API endpoints.

To make the initial registrations of a FASP with a fediverse server
easier, fediverse software MUST include their base URL as part of the
`metadata` of their `nodeinfo` accessible via the `.well-known/nodeinfo`
endpoint.


Example `nodeinfo`:

```json
{
  "version": "2.0",
  "software": {
    "name": "fediexample",
    "version": "6.2.7"
  },
  "protocols": [
    "activitypub"
  ],
  "services": {
    "outbound": [],
    "inbound": []
  },
  "openRegistrations": false,
  "metadata": {
    "nodeName": "fedi",
    "faspBaseUrl": "https://fedi.example.com/fasp"
  }
}
```

### Request Integrity

In order to allow both parties to verify the integrity of message
contents, all requests MUST contain a `content-digest` HTTP header as
defined by [RFC-9530](https://tools.ietf.org/html/rfc9530.html).

The hashing algorith used MUST be SHA-256.

### Authentication

As described in [03: Registration](registration.md) both FASP and
fediverse server generate a unique public/private keypair and exchange
the public keys and an associated identifier with each other.

API requests are being authenticated by HTTP Message Signatures as defined in
[RFC-9421](https://tools.ietf.org/html/rfc9421.html).

The signature algorithm used is "EdDSA Using Curve edwards25519".
Signatures cover signature parameters, the derived components `@method`
and `@target-uri` and the HTTP header `content-digest` (see previous
section).

When signing requests, the `keyid` parameter MUST be the identifier
exchanged during registration, so the other side can infer the
corresponding public key.

The required signature parameters are `created` and `keyid`.

Example headers:

```http
Content-Digest: sha-256=:RK/0qy18MlBSVnWgjwz6lZEWjP/lF5HF9bvEF8FabDg=:
Signature-Input: sig1=("@method" "@target-uri" "content-digest"); created=1728467285;
keyid="b2ks6vm8p23w"
Signature: sig1=:+CcncFjyE+JAuwJO8MOEhRdyfShQz59e9bWDYGN3hoBorVp69k4V2PvS7zJiAoX3QchMlc47sUF4DsptUN+rDQ==:
```

The signature MUST be verified by the receiving party using the public
key belonging to the transmitted identifier (`keyid` parameter).

To ensure the integrity of the request, the derived components `@method`
and `@target-uri` SHOULD be verified. Additionally the integrity of the
`content-digest` HTTP header and its validity SHOULD be checked.

It SHOULD be verified that the `created` timestamp is within an
acceptable range, allowing for time drift between servers.

If this validation fails the response MUST use the HTTP status code
`401` (Unauthorized).

Responses to API requests MUST be signed in the same way, except that
they MUST use the derived component `@status` and the `content-digest`
HTTP header to generate the signature base.

### Rate Limiting

FASPs MAY impose rate limiting on API endpoints, e.g. if the requested capability is
resource intensive. These rate limits are not part of a FASP
specification as they may differ between implementations or even
between installations.

If rate limits are imposed, FASP MUST return the HTTP status code `429`
(Too Many Requests) on rate-limited responses and provide a
`Retry-After` HTTP header (as defined by
[RFC-9110](https://tools.ietf.org/html/rfc9110.html)) with the number of
seconds the fediverse server needs to wait before it can retry this
particular request.

Fediverse server software SHOULD be able to handle these responses and
respect the `Retry-After` header.

---

Next: [03: Registration](registration.md)
