# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

## 03: Common HTTP Interactions

This document introduces some basic API endpoints providers and
instances need to implement. All other endpoints depend on what services
the provider actually offers and are documented in the respective
provider specifications.

There are some common aspects that are mandatory for all providers or
shared between many.

### Authentication and Authorization

As described in [02: Registration](registration.md) both provider and
instance use OAuth 2.0 to authorize API calls. Both MUST obtain a valid
access token and send this as a "bearer token" in the `Authorization`
HTTP header with every API call.

Example header:

```http
Authorization: Bearer SpBr6rheOp891mwWOfT6Pb"
```

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

TODO: Should this be a MUST? I worry this might make implementing this a
lot harder before there even is an example of a provider actually
imposing any limits.

---

Next: [04: Provider Info](provider_info.md)
