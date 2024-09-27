# Fediverse Auxiliary Service Providers: Provider-Instance Interaction

## 01: Introduction

This document introduces the concept of "Fediverse Auxiliary Service
Providers", external services that fediverse server instances can use to
perform a variety of tasks. It defines the way such providers are set-up
within an instance's server software and how the two communicate.

### Fediverse Auxiliary Service Providers

The Fediverse is a decentralized network of different servers, often
called "instances", running different kinds of social media software
that can interoperate by using a shared protocol, ActivityPub.

![Fediverse instances communicating with each other](../../images/instances_federating.svg)

The different sofware offerings have different use cases, e.g.
micro-blogging or photo sharing, and requirements. Instances, even when
using the same software, can also differ in many ways (e.g. number of users,
funding, size of administrative / moderation staff etc.).

That is why some tasks that are useful to many instances regardless of
the software used, are either hard or even impossible for a single
instance to perform. Or every instance performing that task has serious
downsides.

Examples of such tasks are:

* Search and discovery: No single instance has a complete view of the
  full fediverse, and if it had, that would be very expensive and result
  in a lot of duplicated content overall.
* Link preview generation: All instances fetching a web page to generate
  a link preview once a single post with a link is being federated,
  means the web site gets hammered with a lot of requests in a short
  timeframe.
* SPAM detection: This is not an easy problem to solve and every single
  Fediverse software having to implement their own solution seems
  wasteful. Also, it really helps to fight this efficiently if you are able
  to work on data from multiple servers.

Fediverse auxiliary service providers are software services that can
assist instances in performing one or more of these tasks. An instance
administrator can decide to "plug-in" one or more providers. Either to
perform different tasks or to complement each other when performing the
same task.

![Fediverse instances using difference auxiliary service providers](../../images/instances_using_providers.svg)

To learn more about seach and discovery related providers, please visit
the [website of the "Fediverse Discovery Providers" project](https://fediscovery.org).

To learn more about trust and safety related use cases, please refer to
[this blog post](https://renchap.com/blog/post/evolving_mastodon_trust_and_safety/).

### Provider Instance Interaction

Regardless of their exact capabilities, *all* providers have a common
way of interacting with fediverse instances. This document aims to
specify these common interactions up to the point where specific
capabilities can be used. These specific capabilities (search, SPAM
detection etc.) are the subject of their own respective specifications
which all build on this one.

The common interactions are:

1. Registration with the provider
2. Setup of the provider within the fediverse software
3. Display and selection of provider capabilities
4. The ability of the fediverse instance to authenticate with the
provider and call its APIs
5. The ability of the provider to authenticate with the instance and
call its APIs

Please note that 4. and 5. are not the same. Some services might not
need both directions, but many do. Imagine a search provider, where the
instance can perform a search by calling into the provider (4.), but the
provider might also call into the instance to request historic data to
index (5.). Compute-intensive services might be triggerd by an instance
(4.) but only call-back later (5.) with the results once the computation
has finished.

---

Next: [02: Protocol Basics](protocol_basics.md)
