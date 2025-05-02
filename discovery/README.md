# Fediscovery: Fediverse Discovery Providers

This directory contains the basic specifications for discovery
providers, FASPs that facilitate better search and discovery for the
fediverse.

## Motivation

Each server that participates in the fediverse is largely independent
when it comes to discovery. A real user search, discovery of content,
trends and recommendations are not easily possible across the network.

More often than not, the search and discovery experience is limited
to content from the instance a user is on. This poses a problem,
especially on small instances.

See [the website](https://www.fediscovery.org) for details.

## Specifications 

* [Data Sharing](data_sharing/v0.1/)

  All discovery providers need a way to request content from fediverse
  servers. This specification describes both an API to subscribe to new
  content as well as an API to retrieve existing content.

* [Trends](trends/v0.1/)

  This describes the `trends` capability for providers that help
  discovering content that is currently trending.

* [Account Search](account_search/v0.1/)

  This describes the `account_search` capability for providers that
  offer search for fediverse accounts.

* [Follow Recommendation](follow_recommendation/v0.1/)

  This describes the `follow_recommendation` capability for
  providers that can recommend accounts to follow.

* Status/Post Search (Tentatively planned) 
