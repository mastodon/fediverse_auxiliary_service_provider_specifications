# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

This directory contains the basic specifications for the interaction
between a "Fediverse Auxiliary Service Provider" and a Fediverse
software instance.

## Table of Contents

* [01: Introduction](introduction.md)

  Introducing the concept of "Fediverse Auxiliary Service Providers" and
  the basics for this specification.

* [02: Protocol Basics](protocol_basics.md)

  Describes the basic building blocks of the protocol and HTTP
  interactions between providers and instances that are common to all
  types of providers.

* [03: Registration](registration.md)

  Describes the process of registering an instance with a provider,
  covering both protocol and UX aspects.

* [04: Provider info](provider_info.md)

  Describes the basics of an API that providers must offer to query
  important metadata about their service.

* [05: Provider Specification](provider_specifications.md)

  Describes the common requirements for all provider specifications.

## OpenAPI

There is an experimental
[OpenAPI specification file](provider_openapi.yml) included with this
specification.
