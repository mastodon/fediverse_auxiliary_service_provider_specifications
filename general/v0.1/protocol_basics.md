# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

## 02: Protocol Basics

This document introduces some basic API endpoints providers and
instances need to implement. All other endpoints depend on what services
the provider actually offers and are documented in the respective
provider specifications.

There are some common aspects that are mandatory for all providers or
shared between many.

### Definition of Terms and Conventions

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT" and "MAY" in
this document are to be interpreted as described in
[RFC-2119](https://tools.ietf.org/html/rfc2119.html).

All examples and descriptions of API calls include path names that are
meant to be relative to a given base URL. Providers announce their base
URL in their "registration token" and instances call back with their
respective base URLs (see section [03: Registration](registration.md)
for details). For the sake of brevity all examples assume the base URL
to not contain any paths. Implementations MUST consider the base URL
and prefix all API paths accordingly if the base URL contains any path
segments.

### General Approach and Building Blocks

This specification aims to define as little new protocol as possible.
Instead, existing protocols and technologies that many fediverse
software projects already use anyway should be re-used wherever
possible. This approach SHOULD be taken by all provider specifications
that build on this one.

Communication between instance and provider MUST use HTTPS in production
or production-like settings. This requirement MAY only be relaxed in
development environments.

Registration tokens are JSON Web Tokens (JWT) as defined in
[RFC-7519](https://datatracker.ietf.org/doc/html/rfc7519).

For authentication and authorization of API calls, provider and instance
use the OAuth 2.0 protocol as defined in
[RFC-6749](https://tool.ietf.org/html/rfc6749.html).

Custom API calls are HTTPS calls sending, if necessary, JSON data
(`Content-Type: application/json`) and receiving JSON data.

### OAuth2, Authentication and Authorization

As described in [03: Registration](registration.md) both provider and
instance use OAuth 2.0 to authorize API calls. Both MUST obtain a valid
access token and send this as a "bearer token" in the `Authorization`
HTTP header with every API call.

Example header:

```http
Authorization: Bearer SpBr6rheOp891mwWOfT6Pb"
```

Both instance and provider MUST expire access tokens, forcing the other
side to periodically request a new one.

The OAuth 2.0 endpoint to request an access token MUST reside at the
path `/oauth/token` that is relative to the base URL as described
above. This means that existing fediverse software that already uses
OAuth 2.0 and that wants to add provider support cannot re-use existing
routes. But this does simplify implementation of providers a lot and
enables fediverse software implementers to separate their existing OAuth
2.0 implementation for regular API clients from the provider API if so
desired.

### Rate Limiting

Providers MAY impose rate limiting on some API endpoints if they are for
example resource intensive. These rate limits are not part of a provider
specification as they might differ between implementations or even
between installations.

If they do, they MUST return the HTTP status code `429` (Too Many
Requests) on rate-limited responses and provide a `Retry-After` HTTP
header with the number of seconds the instance needs to wait before it
can try this particular request again.

Fediverse instance software SHOULD be able to handle these responses and
respect the `Retry-After` header.

---

Next: [03: Registration](registration.md)
