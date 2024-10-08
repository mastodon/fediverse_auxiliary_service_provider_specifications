# Fediverse Auxiliary Service Providers: Fediverse Server Interaction

This directory contains the basic specifications for the interaction
between a Fediverse Auxiliary Service Provider ("FASP") and a fediverse
server.

## Table of Contents

* [01: Introduction](introduction.md)

  Introducing the concept of Fediverse Auxiliary Service Providers and
  the basics for this specification.

* [02: Protocol Basics](protocol_basics.md)

  The basic building blocks of the protocol and HTTP
  interactions between FASPs and fediverse servers that are common to all
  types of FASPs.

* [03: Registration](registration.md)

  Registering a fediverse server with a FASP,
  covering protocol and UX aspects.

* [04: Provider info](provider_info.md)

  Important metadata that FASPs must offer via API to allow
  fediverse serervs to query their service.

* [05: Provider Specification](provider_specifications.md)

  Common requirements for all FASP specifications.

## OpenAPI

There is an experimental
[OpenAPI specification file](provider_openapi.yml) included with this
specification.
