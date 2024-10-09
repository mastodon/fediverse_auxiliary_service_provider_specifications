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
intended to be relative to a given base URL. FASPs announce their base
URL in their "registration token" and fediverse servers call back with their
respective base URLs (see section [03: Registration](registration.md)
for details). For the sake of brevity all examples assume the base URL
to not contain any paths. Implementations MUST consider the base URL
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

Registration tokens are JSON Web Tokens (JWT) as defined in
[RFC-7519](https://datatracker.ietf.org/doc/html/rfc7519).

Custom API calls are HTTPS calls sending, if necessary, JSON data
(`Content-Type: application/json`) and receiving JSON data.

### Authentication

As described in [03: Registration](registration.md) both FASP and
fediverse server exchange client IDs and secret keys. API requests are
being authenticated by HTTP Message Signatures as defined in [RFC-9421](https://tools.ietf.org/html/rfc9421.html).

Signatures are HMAC using SHA-256 and cover signature parameters and
the following derived components:

* `@method`
* `@target-uri`

The `keyid` parameter MUST include the client ID and the secret key is
used to generate the signature.

The required signature parameters are `created` and `keyid`.

Example headers:

```http
Signature-Input: sig1=("@method" "@target-uri"); created=1728467285;
keyid="b2ks6vm8p23w"
Signature: sig1=bfa93d587d952c44d16ffaaf4ad6a321acf72fe4b6104493455c2349d3da56db
```

The signature MUST be verified by the receiving party. It SHOULD be
verified that the `created` timestamp is within an acceptable range,
allowing for time drift between servers.

If this validation fails the response MUST use the HTTP status code
`401` (Unauthorized).

### Rate Limiting

FASPs MAY impose rate limiting on API endpoints, e.g. if the requested capability is
resource intensive. These rate limits are not part of a FASP
specification as they may differ between implementations or even
between installations.

If rate limits are imposed, FASP MUST return the HTTP status code `429` (Too Many
Requests) on rate-limited responses and provide a `Retry-After` HTTP
header with the number of seconds the fediverse server needs to wait before it
can retry this particular request.

Fediverse server software SHOULD be able to handle these responses and
respect the `Retry-After` header.

---

Next: [03: Registration](registration.md)
