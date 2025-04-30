# Fediverse Auxiliary Service Provider Specifications

This repo collects drafts of specifications for "Fediverse Auxiliary Service Providers". It also serves as a hub for discussions around these specifications.

## What are "Fediverse Auxiliary Service Providers"?

Fediverse Auxiliary Service Providers (FASPs) are independent services that can interact with Fediverse server software to provide services that augment capabilities that are in the server software, are not available in the server software, or cannot be achieved by a single server.

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

[Provider Instance Interaction](general/v0.1/) specifies the initial setup of the fediverse server and FASP relationship and their general interactions. This should be common to all FASPs and forms the basis for the other specifications below.

### Debug

The specification of a very basic debug FASP to help with developing FASP integration into existing fediverse software.

### Search and Discovery

To learn more about the initial plans for search and discovery please visit the [website of the "Fediverse Discovery Provider" project](https://fediscovery.org).

Specifications:

* [Data Sharing](discovery/data_sharing/v0.1/data_sharing.md) 

## Contributing

We welcome contributions from all fediverse software implementers.

If you find a major problem with one of the specifications, please open an issue.

If you would like to correct spelling, grammar etc., please create a pull request.

If you have questions, plan a larger PR (maybe your own FASP specification?) or simply want to discuss anything related to auxiliary service providers, please open a discussion.

## License

This work is marked with CC0 1.0 Universal
