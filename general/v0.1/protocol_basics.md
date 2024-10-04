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

For authentication and authorization of API calls, FASP and fediverse server
use the OAuth 2.0 protocol as defined in
[RFC-6749](https://tool.ietf.org/html/rfc6749.html).

Custom API calls are HTTPS calls sending, if necessary, JSON data
(`Content-Type: application/json`) and receiving JSON data.

### OAuth2, Authentication and Authorization

As described in [03: Registration](registration.md) both FASP and
fediverse server use OAuth 2.0 to authorize API calls. Both MUST obtain a valid
access token and send this as a "bearer token" in the `Authorization`
HTTP header with every API call.

Example header:

```http
Authorization: Bearer SpBr6rheOp891mwWOfT6Pb"
```

Both FASP and fediverse server MUST expire access tokens, forcing the other
side to periodically request a new one.

The OAuth 2.0 endpoint to request an access token MUST reside at the
path `/oauth/token` that is relative to the base URL as described
above. Existing fediverse software that already uses
OAuth 2.0 and wants to add FASP support cannot re-use existing
routes. This simplifies FASP implementation and
enables fediverse software implementers to separate their existing OAuth
2.0 implementation for regular API clients from the FASP API if so
desired.

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
