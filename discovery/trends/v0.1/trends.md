# "Fediscovery": Trends

## Overview

Specification identifier: `trends`

The `trends` capability allows fediverse servers to query FASP for
content, hashtags and links that are currently trending.

## Trends and Discovery

A huge part of discovery in social media is for users to be able to see
what other people are currently posting, including but not limited to:

* A post "going viral", meaning it is being replied to, liked and shared
  a lot
* A hashtag being used unusually often within a given timeframe
* A link being included in a lot in different posts

Many fediverse software products already include ways to compute and
display these trends. But a single fediverse server is always limited to
the content it knows about. A discovery FASP can potentially see more
fediverse-wide activity and thus provide fediverse servers with more
interesting trends data.

This specification defines an API for fediverse servers to query FASP
for this trends data.

It does not specify how exactly trends are computed. FASP MAY use all
the data that is available to them, including the data obtained via
[data sharing](../../data_sharing/v0.1/data_sharing.md), to compute
trends.

Different implementations of this specifications MAY even compete on the
actual algorithm(s) they use.

FASP SHOULD document how they compute trends so that fediverse server
administrators can make an informed decision when choosing between
different discovery FASP.

## Representation of Content, Hashtags and Links

Content refers to ActivityPub objects that have a unique identifier,
their URI.

Uniquely identifying hashtags and links is more complex.

Hashtags are represented by their name, a UTF-8 encoded string beginning
with `#`. Different hashtag names might refer to the same concept,
especially if they only differ in their use of upper and lower case
characters. For example `#activitypub` and `#ActivityPub` can be
considered to be the same.

Links are HTTP(S) URLs. Syntactically different URLs can refer to the same
resource. A simple example is again case-sensitivity as the domain part
of an HTTP(S) URL is case-insensitive. So `https://example.com/test` and
`https://EXAMPLE.com/test` both refer to the same resource.

Fediverse servers already have to deal with this problem and will in
most cases have some concept of "normalization" for both hashtags and
links. This "normalization" is not part of any standard and different
fediverse software can have different and even incompatible
implementations of this.

That is why when sharing trending hashtags and links with fediverse
servers, FASP do not need to take special care how to represent them and
can freely choose the representation (e.g. the first one they
encountered or a normalized version). Fediverse servers SHOULD handle
them just like they handle these representations in other contexts.

When computing trends however, FASP SHOULD consider normalization to
minimize the risk of obvious duplicates and individual spellings of the
same hashtag not meeting the threshold of what constitutes a trend,
while the aggregated numbers clearly would.

For hashtags possible strategies for this besides case-insensitive
handling of names include "ASCII folding" and using existing character
transliteration rules. As all of these approaches have their downsides
and can lead to two different words being erroneously treated as the
same it is up to FASP to decide which strategy to use.

For links [RFC-3986](https://tools.ietf.org/html/rfc3986.html) describes
different normalization techniques in section 6.

## Availability of Content for Hashtags and Links

When a fediverse server requests trending hashtags or links it will in
most cases want to display them to users and have a way for users to see
how and where those have been used. Since FASP have a wider view of the
fediverse it might be the case that the fediverse server has no content
at all that references a trending hashtag or link. Or it might not have
recent content that does so.

To help fediverse servers in this situation discovery FASP include a
couple of example URIs of content with every result of a query for
hashtags and links trends. This allows fediverse servers to quickly fill
their local caches with some content for the received trends. See the
sections "Requesting Trending Hashtags" and "Requesting Trending Links"
below for details.

## Requesting Trends

### Common Request Options

All requests for trends from a fediverse server to FASP are HTTP `GET`
requests that allow - with one exception - using the same optional
parameters:

* `withinLastHours`: This MUST be a positive integer specifying the
  number of hours up to the current time for which to compute trends.
  The minimum value of `1` means records trending within the last hour.
  FASP MUST support values up to `168` (i.e. one week) but MAY allow
  larger values. If omitted defaults to `24`.
* `maxCount`: This MUST be a positive integer specifying the maximum
  number of results that should be returned. If omitted defaults to
  `20`.
* `language`: A [BCP47](https://tools.ietf.org/html/bcp47) language tag
  to only receive results in or relevant for the specific language. If
  omitted results can be in or relevant for any language. FASP MUST
  perform "basic filtering" as described by
  [RFC-4647](https://tools.ietf.org/html/rfc4647.html) to determine
  matching languages.

### Common Response Attribute

All responses include a list of results. Individual result objects
share the following common key and value:

* `rank`: A positive integer less than or equal `100` representing the
  rank of this result. `100` is the highest rank, meaning "most trending".
  Calculation of the rank is not part of this specification, but within
  the same FASP software, ranks MUST be comparable. If a fediverse
  server uses several FASP to query for trends that all run the same
  software, it MUST be possible to merge results according to rank to
  get the correct order.

### Requesting Trending Content

To request trending content fediverse servers can make an HTTP `GET`
request to the `/trends/v0/content` endpoint on the FASP.

Example call:

```http
GET /trends/v0/content
```

Optionally all the parameters from the previous section "Common Request
Options" can be used.

Another example, limiting the results to be trending in the last two
hours and to no more than 10 results:

```http
GET /trends/v0/content?withinLastHours=2&maxCount=10
```

FASP MUST respond with an HTTP status code `200` (OK) and a JSON object
that contains a single key, `content`. The value of that key is an array
of objects.

These objects include the following keys:

* `uri`: The URI of the content object.
* `rank`: See previous section "Common Response Attributes".

These objects MUST be sorted by `rank` in descending order.

Example response for a request for trending content from the last three
hours:

```json
{
  "content": [
    {
      "uri": "https://fedi1.example.com/status/23",
      "rank": 100
    },
    {
      "uri": "https://fedi3.example.com/posts/17",
      "rank": 74
    },
    {
      "uri": "https://fedi2.example.com/users/1/posts/56",
      "rank": 55
    }
  ]
}
```

### Requesting Trending Hashtags

To request trending hashtags fediverse servers can make an HTTP `GET`
request to the `/trends/v0/hashtags` endpoint on the FASP.

Example call:

```http
GET /trends/v0/hashtags
```

Optionally all the parameters from the previous section "Common Request
Options" can be used.

Another example, limiting the results to be relevant to the french
language:

```http
GET /trends/v0/hashtags?language=fr
```

FASP MUST respond with an HTTP status code `200` (OK) and a JSON object
that contains a single key, `hashtags`. The value of that key is an array
of objects.

These objects include the following keys:

* `name`: The name of the hashtag.
* `rank`: See previous section "Common Response Attributes".
* `examples`: An array of URIs of content that uses the hashtag. See
  section "Availability of Content for Hashtags and Links" above for a
  rationale.

These objects MUST be sorted by `rank` in descending order.

Example response for a request for trending hashtags from the last three
hours:

```json
{
  "hashtags": [
    {
      "name": "#fediscovery",
      "rank": 100,
      "examples": [
        "https://fedi1.example.com/status/23",
        "https://fedi3.example.com/posts/17",
        "https://fedi2.example.com/users/1/posts/56"
      ]
    },
    {
      "name": "#cats",
      "rank": 72,
      "examples": [
        "https://fedi3.example.com/posts/89",
        "https://fedi1.example.com/status/976",
        "https://fedi2.example.com/users/83/posts/26"
      ]
    }
  ]
}
```

### Requesting Trending Links

To request trending links fediverse servers can make an HTTP `GET`
request to the `/trends/v0/links` endpoint on the FASP.

Example call:

```http
GET /trends/v0/links
```

Optionally all the parameters from the previous section "Common Request
Options" can be used.

Another example, requesting at most the top 5 links from the last week: 

```http
GET /trends/v0/links?maxCount=5&withinLastHours=168
```

FASP MUST respond with an HTTP status code `200` (OK) and a JSON object
that contains a single key, `links`. The value of that key is an array
of objects.

These objects include the following keys:

* `url`: The URL of the link. 
* `rank`: See previous section "Common Response Attributes".
* `examples`: An array of URIs of content that includes the link. See
  section "Availability of Content for Hashtags and Links" above for a
  rationale.

These objects MUST be sorted by `rank` in descending order.

Example response for a request for trending links from the last three
hours:

```json
{
  "links": [
    {
      "url": "https://blog.example.com/posts/23",
      "rank": 100,
      "examples": [
        "https://fedi1.example.com/status/23",
        "https://fedi3.example.com/posts/17",
        "https://fedi2.example.com/users/1/posts/56"
      ]
    },
    {
      "url": "https://news.example.com/articles/45",
      "rank": 72,
      "examples": [
        "https://fedi3.example.com/posts/89",
        "https://fedi1.example.com/status/976",
        "https://fedi2.example.com/users/83/posts/26"
      ]
    }
  ]
}
```

## Privacy Policy Information

With the APIs specified above fediverse servers do not share any personally
identifiable or otherwise sensitive information with FASP.
