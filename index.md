---
layout: page
title: "A standard for building APIs in JSON."
show_masthead: true
---

If you've ever argued with your team about the way your JSON responses
should be formatted, JSON API is your anti-bikeshedding weapon.

By following shared conventions, you can increase productivity,
take advantage of generalized tooling, and focus on what
matters: your application.

Clients built around JSON API are able to take
advantage of its features around efficiently caching responses,
sometimes eliminating network requests entirely.

Here's an example response from a blog that implements JSON API:

```json
{
  "links": {
    "self": "http://example.com/posts",
    "next": "http://example.com/posts?page[offset]=2",
    "last": "http://example.com/posts?page[offset]=10"
  },
  "data": [{
    "type": "posts",
    "id": "1",
    "title": "JSON API paints my bikeshed!",
    "links": {
      "self": "http://example.com/posts/1",
      "author": {
        "self": "http://example.com/posts/1/links/author",
        "resource": "http://example.com/posts/1/author",
        "type": "people",
        "id": "9"
      },
      "comments": {
        "self": "http://example.com/posts/1/links/comments",
        "resource": "http://example.com/posts/1/comments",
        "type": "comments",
        "ids": ["5", "12"]
      }
    }
  }],
  "linked": [{
    "type": "people",
    "id": "9",
    "first-name": "Dan",
    "last-name": "Gebhardt",
    "twitter": "dgeb",
    "links": {
      "self": "http://example.com/people/9"
    }
  }, {
    "type": "comments",
    "id": "5",
    "body": "First!",
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "body": "I like XML better",
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
}
```

The response above contains the first in a collection of "posts", as well as
links to subsequent members in that collection. It also contains resources
linked to the post, including its author and comments. Last but not least,
links are provided that can be used to fetch or update any of these
resources.

JSON API covers creating and updating resources as well, not just responses.

{% include status.md %}

## MIME Types <a href="#mime-types" id="mime-types" class="headerlink"></a>

JSON API has been properly registered with the IANA. Its media
type designation is [`application/vnd.api+json`](http://www.iana.org/assignments/media-types/application/vnd.api+json).

## Format documentation <a href="#format-documentation" id="format-documentation" class="headerlink"></a>

To get started with JSON API, check out [documentation for the base
specification](/format).

## Extensions <a href="#extensions" id="extensions" class="headerlink"></a>

JSON API can be [extended in several ways](/extensions).

Official extensions are available for [bulk](/extensions/bulk/) and
[patch](/extensions/patch/) operations.

## Update history <a href="#update-history" id="update-history" class="headerlink"></a>

- 2013-05-03: Initial release of the draft.
- 2013-07-22: Media type registration completed with the IANA.

You can subscribe to an RSS feed of individual changes [here](https://github.com/json-api/json-api/commits.atom).
