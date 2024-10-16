# Fediscovery: Fediverse Discovery Providers

## 01: Introduction

This document describes a set of FASP capabilities to facilitate search
and discovery on the fediverse.

### Motivation

Each server that participates in the fediverse is largely independent
when it comes to discovery. A real user search, discovery of content,
trends and recommendations are not easily possible across the network.

More often than not, the search and discovery experience is limited
to content from the instance a user is on. This poses a problem,
especially on small instances.

### Search and Discovery Capabilities

This document introduces the following four capabilities:

* `trends`
* `account_search`
* `account_recommendation`
* `post_search`

`trends` allows querying for trending posts, accounts, hashtags etc.

`account_search` allows searching for accounts from all over the
fediverse.

`account_recommendation` helps with discovering interesting accounts to
follow.

`post_search` offers the ability to search for posts.

A FASP can decide to implement one, several or all of these
capabilities.

### Common Concerns

In order to be able to present search results the FASP has to ingest
content from across the fediverse. This includes subscribing to get new
content from fediverse servers as it is being created as well as
requesting older content.

This is something that all four capabilities have in common. The next
section describes a common API that all FASPs that implement at least
one discovery capability and all fediverse servers that want to make use
of it need to implement.

---

Next: [02: Content Ingestion](02_content_ingestion.md)
