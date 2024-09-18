# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

This directory contains the basic specifications for the interaction
between a "Fediverse Auxiliary Service Provider" and a Fediverse
software instance.

## Table of Contents

* [01: Introduction](introduction.md)

  Introducing the concept of "Fediverse Auxiliary Service Providers" and
  the basics for this specification.

* [02: Registration](registration.md)

  Describes the process of registering an instance with a provider,
  covering both protocol and UX aspects.

* [03: Common HTTP interactions](common_http_interactions.md)

  Describes HTTP interactions between providers and instances that are
  common to all types of providers.

* [04: Provider info](provider_info.md)

  Describes the basics of an API that providers must offer to query
  important metadata about their service.

* [05: Provider Specification](provider_specification.md)

  Describes the common requirements for all provider specifications.

## OpenAPI

There is an experimental
[OpenAPI specification file](provider_openapi.yml) included with this
specification.
