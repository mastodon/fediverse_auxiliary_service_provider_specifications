# Fediverse Auxiliary Service Provider Specifications

This repo collects drafts of specifications for "Fediverse Auxiliary Service Providers". It also serves as a hub for discussions around these specifications.

## What are "Fediverse Auxiliary Service Providers"?

Fediverse Auxiliary Service Providers are independent services that can interact with Fediverse instances to provide services that are either not at the core of the Fediverse instance's software or cannot be achieved by a single instance.

Some examples for these services would be:

* Search and Discovery
* SPAM detection
* Blocklist synchronization
* Link preview generation
* Helpdesk integration

![Fediverse instances using difference auxiliary service providers](images/instances_using_providers.svg)

Read a more thorough explanation in the [Provider Instance Interaction Specification introduction](general/v0.1/introduction.md).

## List of Specifications

### General

[Provider Instance Interaction](general/v0.1/) specifies the initial setup of the instance and provider relationship and their general interaction. This should be common to all providers and forms the basis for the other specifications below.

### Dummy

The specification of a dummy provider to help with developing provider integration into existing fediverse software.

### Search and Discovery

To learn more about the initial plans around search and discovery please visit the [website of the "Fediverse Discovery Provider" project](https://fediscovery.org).

Specifications:

* Coming soon 

## Contributing

We welcome contributions from all Fediverse software implementers.

If you find a major problem with one of the specifications, please open an issue.

If you would like to correct spelling, grammar etc., please create a pull request.

If you have questions, plan a larger PR (maybe your own provider specification?) or simply want to discuss anything related to auxiliary service providers, please open a discussion.
