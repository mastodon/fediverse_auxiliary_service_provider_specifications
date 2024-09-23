# Debug Provider Specification

## Overview

The debug provider is a
[Fediverse Auxiliary Service Provider](../../general/v0.1/)
that only has a single capability, that includes one API endpoint and
the ability to call one API endpoint in registered instances.

Both the provider and the instance SHOULD log calls to these APIs and
present this information to the instance administrator.

It is meant to aid in developing provider integrations into existing
fediverse software and to debug issues with provider setup in general.

### Capabilities

This provider offers a single capability.

Identifier: `debug`

Description:

> The `debug` capability offers a single API endpoint in the provider.
> When called it makes the provider call an API on the instance in
> return. The calls are logged to help debug issues with the
> provider/instance setup.

### Provider API Endpoints

The provider allows an instance to make a HTTP `POST` call to `/debug/log`.

OAuth 2.0 scope: `debug`

Example call:

```http
POST /debug/log
```

The request body MAY be empty or contain a single JSON object.

The provider MUST log:

* that the request has been made,
* at what time it was made,
* the IP address of the instance that made the request and
* if present the JSON object from the request body 

The provider must then make the API call described in the next section
to the instance.

### Instance API Endpoints

The instance allows the provider to make a HTTP `POST` call to
`/debug/callback`.

OAuth 2.0 scope: `aux:debug`

Example call:

```http
POST /debug/callback
```

The request body MAY be empty, unless the instance included a JSON
object in its call to the provider. In this case the provider MUST
include this object in the request body unchanged.

The instance SHOULD log:

* that the request has been made,
* at what time it was made,
* the IP address of the provider that made the request and
* if present the JSON object from the request body 

This is a strong "SHOULD" but not a "MUST" because skipping this step
might make the implementation inside the instance's software easier
while still providing some value.

### User Interface Requirements

The provider MUST offer the instance admin a user interface that
displays the logged information.

TODO Mockup

The instance MUST provide a button or link to initiate the API call to
the provider. It SHOULD provide a way for the instance admin to see the
logged callbacks from the provider.

TODO Mockup

### Data Protection

This provider does not receive any personally identifiable or otherwise
sensitive information.
