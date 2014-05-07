---
layout: page
title: "Format"
---

{% include status.md %}

## Introduction <a href="#introduction" id="introduction" class="headerlink">¶</a>

JSON API is a media type ([`application/vnd.api+json`](http://www.iana.org/assignments/media-types/application/vnd.api+json)) that specifies how a client should request that resources be fetched or modified and how a server should respond to those requests. JavaScript Object Notation (JSON) [[RFC4627](http://tools.ietf.org/html/rfc4627)] is used for the exchange of data.

JSON API is designed to minimize both the number of requests and the amount of data transmitted between clients and servers. Many of the optional features in JSON API are aimed at furthering this goal.

A JSON API server supports fetching of resources through the HTTP method GET. In order to support creating, updating and deleting resources, it must support use of the HTTP methods POST, PUT and DELETE, respectively. 

A JSON API server may also optionally support modification of resources with the HTTP PATCH method [[RFC5789](http://tools.ietf.org/html/rfc5789)] and the JSON Patch format [[RFC6902](http://tools.ietf.org/html/rfc6902)]. JSON Patch support is possible because, conceptually, JSON API represents all of a domain's resources as a single JSON document that can act as the target for operations. Resources are grouped at the top level of this document according to their type. Each resource can be identified at a unique path within this document.

## Conventions <a href="#conventions" id="conventions" class="headerlink">¶</a>

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [[RFC2119](http://tools.ietf.org/html/rfc2119)].

## Document Structure <a href="#document-structure" id="document-structure" class="headerlink">¶</a>

Different JSON documents are referenced throughout this specification:

* "Resource documents" are JSON representations of resources and their associations. "Primary documents" represent the primary resource(s) specified by a request, while "linked documents" represent resources associated with the primary resource(s).

* "Link documents" are JSON representations of links between resources.

* The "response document" is the entire JSON document returned as the body of a response.

* The "request document" is the entire JSON document sent as the body of a request.

### Resource Documents <a href="#resource-documents" id="resource-documents" class="headerlink">¶</a>

Resource documents have the same internal structure, regardless of whether they represent primary or linked resources and whether they are contained in request or response documents. Some attributes are only applicable for particular contexts, as explained below. 

Here's how a "post" resource might appear in a response document:

```javascript
{
  "posts": [{
    "id": "1",
    "title": "Rails is Omakase"
  }]
}
```

In the example above, the resource document is simply:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase"
  }
//...
```

This section will focus exclusively on resource documents, outside of the context of request and response documents.

#### Resource Attributes <a href="#resource-document-attributes" id="resource-document-attributes" class="headerlink">¶</a>

There are four reserved keys in resource documents:

* `"id"`
* `"type"`
* `"href"`
* `"links"`

Every other key in a resource document represents an "attribute". An attribute's value may
be any JSON value.

#### Resource IDs <a href="#resource-document-ids" id="resource-document-ids" class="headerlink">¶</a>

Each resource document **SHOULD** contain an `"id"` key.

The `"id"` key in a document represents a unique identifier for the underlying
resource, scoped to its type. It **MUST** be a string which **SHOULD** only
contain alphanumeric characters, dashes and underscores. It can be used with URL
templates to fetch related resources, as described below.

In scenarios where uniquely identifying information between client and server
is unnecessary (e.g., read-only, transient entities), JSON API allows for
omitting the `"id"` key.

NOTE: While an implementation could use the values of `"id"` keys as URLs
(which are unique string identifiers, after all), it is not generally
recommended. URLs can change, so they are unreliable for mapping a document to
any client-side models that represent the same resource. It is recommended that
URL values be left to the task of linking documents while `"id"` values remain
opaque to solely provide a unique identity within some type.

#### Resource Types

The type of each resource document can usually be determined from the context of the response or request in which it is contained.

Each resource document **MAY** contain a `"type"` key that designates its type. 

The `"type"` key is **REQUIRED** when the type of a resource can not be specified or inferred otherwise.

For example, here's an array of resource documents that might be part of a has-many polymorphic relationship:

```javascript
//...
  [{
    "id": "9",
    "type": "people",
    "name": "@tenderlove"
  }, {
    "id": "1",
    "type": "cats",
    "name": "@gorbypuff"
  }]
//...
```

#### Resource URLs (Responses Only)

The URL of each resource in a response document may be specified with the `"href"` key.

```javascript
//...
  [{
    "id": "1",
    "href": "http://example.com/comments/1",
    "body": "Mmmmmakase"
  }, {
    "id": "2",
    "href": "http://example.com/comments/2",
    "body": "I prefer unagi"
  }]
//...
```

A server **MUST** respond to a `GET` request to the specified URL with a response that includes the resource represented as a single primary document.

It is generally more efficient to specify URL templates at the root level of a response document rather than to specify individual URLs per resource.

#### Resource Relationships <a href="#resource-document-relationships" id="resource-document-relationships" class="headerlink">¶</a>

The value of the `"links"` key is a JSON object that represents linked resources, keyed by the name of each association.

For example, the following post is associated with a single `author` and a collection of `comments`:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "author": "9",
      "comments": [ "5", "12", "17", "20" ]
    }
  }
//...
```

The allowed values for each "link" depend upon whether it represents a to-one or to-many relationship, as discussed below.

##### To-One Relationships <a href="#resource-document-relationships-to-one-relationships" id="resource-document-relationships-to-one-relationships" class="headerlink">¶</a>

A to-one relationship **MAY** be represented as a string value that corresponds to the ID of a linked resource.

For example, the following post is associated with a single author:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "author": "17"
    }
  }
//...
```

A to-one relationship **MAY** alternatively be represented with a "to-one link document" (see below).

For example, the link document below provides details about the linked author:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "author": {
        "href": "http://example.com/people/17",
        "id": "17",
        "type": "people"
      }
    }
  }
//...
```

##### To-Many Relationships <a href="#resource-document-relationships-to-many-relationships" id="resource-document-relationships-to-many-relationships" class="headerlink">¶</a>

A to-many relationship **MAY** be represented as an array of strings corresponding to IDs of linked resources.

For example, the following post is associated with several comments:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "comments": [ "5", "12", "17", "20" ]
    }
  }
//...
```

A to-many relationship **MAY** alternatively be represented with a "to-many link document" (see below).

For example, the link document below provides details about the linked comments:

```javascript
//...
  {
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "comments": {
        "href": "http://example.com/comments/5,12,17,20",
        "ids": [ "5", "12", "17", "20" ],
        "type": "comments"
      }
    }
  }
//...
```

As another alternative, a to-many relationship **MAY** be represented as an array of "to-one link documents" (see below). Given its verbosity, this format should be used sparingly, but is helpful when linked resources may vary in `"type"`.

```javascript
//...
  {
    "id": "1",
    "title": "One Type Purr Author",
    "links": {
      "authors": [{
        "href": "http://example.com/people/9",
        "id": "9",
        "type": "people"
      },
      {
        "href": "http://example.com/cats/1",
        "id": "1",
        "type": "cats"
      }]
    }
  }
//...
```

### Link Documents

Link documents **MAY** be used within a resource document's `links` key to represent to-one or to-many relationships.

#### To-One Link Documents

A "to-one link document" contains one or more of the attributes: 

* `"id"` - the ID of the linked resource.
* `"type"` - the resource **type**.
* `"href"` - the URL of the linked resource (only applicable to response documents).

A server that provides a to-one relationship as a URL (via `"href"`) **MUST** respond to a `GET` request to the specified URL with a response that includes the resource represented as a single primary document.

#### To-Many Link Documents

A "to-many link document" contains one or more of the attributes: 

* `"ids"` - an array of IDs for the linked resources.
* `"type"` - the resource **type**.
* `"href"` - the URL of the linked resources (only applicable to response documents). 

A server that provides a to-many relationship as a URL (via `"href"`) **MUST** respond to a `GET` request to the specified URL with a response that includes the resource represented as a collection of primary documents.


### Response Documents <a href="#response-documents" id="response-documents" class="headerlink">¶</a>

The top-level of a response document **SHOULD** contain a collection of primary documents. This collection **SHOULD** be keyed either by the plural form of the resource type or the generic key `"data"`. 

A single primary document **SHOULD** be represented as a member of an array:

```javascript
{
  "posts": [{
    "id": 1
    // an individual post document
  }]
}
```

Multiple primary documents **SHOULD** also be represented as an array:

```javascript
{
  "posts": [{
    "id": 1
    // an individual post document
  }, {
    "id": 2
    // an individual post document
  }]
}
```

Heterogenous collections **MUST** be keyed by `"data"`. Any collection keyed by `"data"` instead of by its type **MUST** specify `"type"` for each contained resource.

```javascript
{
  "data": [{
    "id": "9",
    "type": "people",
    "name": "@tenderlove"
  }, {
    "id": "1",
    "type": "cats",
    "name": "@gorbypuff"
  }]
}
```

The top-level of a response document **MAY** also have the following keys:

* `"meta"`: meta-information about a resource, such as pagination.
* `"links"`: URL templates to be used for expanding resources' relationships
  URLs.
* `"linked"`: a collection of resource documents, grouped by type, that are linked to
  the primary document(s) and/or each other.

No other keys should be present at the top level of the response document.

#### URL Templates <a href="#response-documents-url-templates" id="response-documents-url-templates" class="headerlink">¶</a>

A top-level `"links"` object **MAY** be used to specify URL templates that can be used to formulate URLs for resources according to their type.

Example:

```javascript
{
  "links": {
    "posts.comments": "http://example.com/posts/{posts.id}/comments"
  },
  "posts": [{
    "id": "1",
    "title": "Rails is Omakase"
  }, {
    "id": "2",
    "title": "The Parley Letter"
  }]
}
```

In this example, fetching `http://example.com/posts/1/comments` will fetch
the comments for `"Rails is Omakase"` and fetching `http://example.com/posts/2/comments`
will fetch the comments for `"The Parley Letter"`.

```javascript
{
  "links": {
    "posts.comments": "http://example.com/comments/{posts.comments}"
  },
  "posts": [{
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "comments": [ "1", "2", "3", "4" ]
    }
  }]
}
```

In this example, the `comments` variable is expanded by
"exploding" the array specified in the `"links"` section of each post.
The URL template specification [[RFC6570](https://tools.ietf.org/html/rfc6570)]
specifies that the default explosion is to percent encode the array members 
(e.g. via `encodeURIComponent()` in JavaScript) and join them by a comma. In 
this example, fetching `http://example.com/comments/1,2,3,4` will return a list 
of all comments.

The top-level `"links"` object has the following behavior:

* Each key is a dot-separated path that points at a repeated relationship. Paths start
  with a particular resource type and can traverse related resources. 
  For example `"posts.comments"` points at the `"comments"` relationship in each 
  document of type `"posts"`.
* The value of each key is interpreted as a URL template.
* For each document that the path points to, act as if it specified a
  relationship formed by expanding the URL template with the non-URL value
  actually specified.

Here is another example that uses a has-one relationship:

```javascript
{
  "links": {
    "posts.author": "http://example.com/people/{posts.author}"
  },
  "posts": [{
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "author": "12"
    }
  }, {
    "id": "2",
    "title": "The Parley Letter",
    "links": {
      "author": "12"
    }
  }, {
    "id": "3",
    "title": "Dependency Injection is Not a Virtue",
    "links": {
      "author": "12"
    }
  }]
}
```

In this example, the URL for the author of all three posts is
`http://example.com/people/12`.

Top-level URL templates allow you to specify relationships as IDs, but without
requiring that clients hard-code information about how to form the URLs.

NOTE: In case of conflict, an individual document's `links` object will take
precedence over a top-level `links` object.

#### Compound Documents <a href="#response-documents-compound-documents" id="response-documents-compound-documents" class="headerlink">¶</a>

To save HTTP requests, responses may optionally allow for the inclusion of linked documents along with the requested primary documents. Such response documents are called "compound documents".

In a compound document, linked documents **MUST** be included in a top level `"linked"` object, in which they are grouped together in arrays according to their type.

The type of each relationship **MAY** be specified in the `"links"` object with
the `"type"` key. This facilitates lookups of linked documents by the client.

```javascript
{
  "links": {
    "posts.author": {
      "href": "http://example.com/people/{posts.author}",
      "type": "people"
    },
    "posts.comments": {
      "href": "http://example.com/comments/{posts.comments}",
      "type": "comments"
    }
  },
  "posts": [{
    "id": "1",
    "title": "Rails is Omakase",
    "links": {
      "author": "9",
      "comments": [ "1", "2", "3" ]
    }}, {
    "id": "2",
    "title": "The Parley Letter",
    "links": {
      "author": "9",
      "comments": [ "4", "5" ]
   }}, {
    "id": "1",
    "title": "Dependency Injection is Not a Virtue",
    "links": {
      "author": "9",
      "comments": [ "6" ]
    }
  }],
  "linked": {
    "people": [{
      "id": "9",
      "name": "@d2h"
    }],
    "comments": [{
      "id": "1",
      "body": "Mmmmmakase"
    }, {
      "id": "2",
      "body": "I prefer unagi"
    }, {
      "id": "3",
      "body": "What's Omakase?"
    }, {
      "id": "4",
      "body": "Parley is a discussion, especially one between enemies"
    }, {
      "id": "5",
      "body": "The parsley letter"
    }, {
      "id": "6",
      "body": "Dependency Injection is Not a Vice"
    }]
  }
}
```

This approach ensures that a single canonical representation of each document
is returned with each response, even when the same document is referenced
multiple times (in this example, the author of the three posts). Along these
lines, if a primary document is linked to another primary or linked document,
it should not be duplicated within the `"linked"` object.

### Request Documents <a href="#request-documents" id="request-documents" class="headerlink">¶</a>

The request document structure varies according to the request method and is covered in detail below.

## URLs

The URL for a collection of resources **SHOULD** be formed from the plural form of the resource **type**.

For example, a collection of photos should be accessible at:

```text
/photos
```

The URL for an individual resource **SHOULD** be formed by appending the resource's ID to the collection URL.

For example, the photo with an `id` of `1` should be accessible at:

```text
/photos/1
```

The URL for multiple individual resources **SHOULD** be formed by appending a comma-separated list of resource IDs to the collection URL.

For example, the photos with `id`s of `1` or `2` or `3` should be accessible at:

```text
/photos/1,2,3
```

### Alternative URLs

Alternative URLs for resources **MAY** optionally be specified in responses with `"href"` attributes or URL templates.

### Relationship URLs

A resource's relationship **MAY** be accessible at a URL formed by appending `links/<relationship-name>` to the resource's URL. This relative path is consistent with the internal structure of a resource document.

For example, a photo's collection of linked comments should be accessible at:

```text
/photos/1/links/comments
```

A photo's reference to an individual linked photographer should be accessible at:

```text
/photos/1/links/photographer
```

Note that these URLs represent the **relationship** and not the linked resource. 

A server **MUST** represent "to-one" relationships as an ID or a "to-one link document".

"To-many" relationships **MUST** be represented as an array of IDs, a single "to-many link document" or an array of "to-one link documents".


## Fetching Resources <a href="#fetching" id="fetching" class="headerlink">¶</a>

A resource, or collection of resources, can be fetched by sending a `GET` request to the URL described above.

Responses can be further refined with the optional features described below. 

### Filtering <a href="#fetching-filtering" id="fetching-filtering" class="headerlink">¶</a>

A server **MAY** choose to support requests to filter resources according to specific criteria.

Filtering **SHOULD** be supported by appending parameters to the base URL for the collection of resources to be filtered.

For example, the following is a request for all comments associated with a particular post:

```text
GET /comments?posts=1
```

With this approach, multiple filters **MAY** be applied to a single request:

```text
GET /comments?posts=1&author=12
```

This specification only supports filtering based upon strict matching. Additional filtering allowed by an API should be specified in its profile (see [Extending](/extending)).

### Inclusion of Linked Documents <a href="#fetching-inclusion-of-linked-documents" id="fetching-inclusion-of-linked-documents" class="headerlink">¶</a>

A server **MAY** choose to support returning compound documents that include
both primary and linked documents.

An endpoint **MAY** return documents linked to the primary document(s) by
default.

An endpoint **MAY** also support custom inclusion of linked documents based
upon an `include` request parameter. This parameter should specify the path to
one or more documents relative to the primary document. If this parameter is
used, **ONLY** the requested linked documents should be returned alongside the
primary document(s).

For instance, comments could be requested with a post:

```text
GET /posts/1?include=comments
```

In order to request documents linked to other documents, the dot-separated path
of each document should be specified:

```text
GET /posts/1?include=comments.author
```

Note: a request for `comments.author` should not automatically also include
`comments` in the response (although comments will obviously need to be
queried in order to fulfill the request for their authors).

Multiple linked documents could be requested in a comma-separated list:

```text
GET /posts/1?include=author,comments,comments.author
```

### Sparse Fieldsets <a href="#fetching-sparse-fieldsets" id="fetching-sparse-fieldsets" class="headerlink">¶</a>

A server **MAY** choose to support requests to return only specific fields in 
resource documents.

An endpoint **MAY** support requests that specify fields for the primary document
type with a `fields` parameter.

```text
GET /people?fields=id,name,age
```

An endpoint **MAY** support requests that specify fields for any document type
with a `fields[DOCUMENT_TYPE]` parameter.

```text
GET /posts?include=author&fields[posts]=id,title&fields[people]=id,name
```

An endpoint SHOULD return a default set of fields in a document if no fields
have been specified for its type, or if the endpoint does not support use of
either `fields` or `fields[DOCUMENT_TYPE]`.

Note: `fields` and `fields[DOCUMENT_TYPE]` can not be mixed. If the latter
format is used, then it must be used for the primary document type as well.

### Sorting <a href="#fetching-sorting" id="fetching-sorting" class="headerlink">¶</a>

A server **MAY** choose to support requests to sort documents according to
one or more criteria.

An endpoint **MAY** support requests to sort the primary document type with a
`sort` parameter.

```text
GET /people?sort=age
```

An endpoint **MAY** support multiple sort criteria by allowing comma-separated
fields as the value for `sort`. Sort criteria should be applied in the order
specified.

```text
GET /people?sort=age,name
```

The default sort order **SHOULD** be ascending. A `-` prefix on any sort field
specifies a descending sort order.

```text
GET /posts?sort=-created,title
```

The above example should return the newest posts first. Any posts created on the
same date will then be sorted by their title in ascending alpabetical order.

An endpoint **MAY** support requests to sort any document type with a
`sort[DOCUMENT_TYPE]` parameter.

```text
GET /posts?include=author&sort[posts]=-created,title&sort[people]=name
```

If no sort order is specified, or if the endpoint does not support use of either
`sort` or `sort[DOCUMENT_TYPE]`, then the endpoint **SHOULD** return documents
sorted with a repeatable algorithm. In other words, documents **SHOULD** always
be returned in the same order, even if the sort criteria aren't specified.

Note: `sort` and `sort[DOCUMENT_TYPE]` can not be mixed. If the latter
format is used, then it **MUST** be used for the primary document type as well.

## Creating, Updating and Deleting Resources <a href="#creating-updating-deleting" id="creating-updating-deleting" class="headerlink">¶</a>

A server **MAY** allow resources that can be fetched to also be created, modified and deleted.

A server **MAY** allow multiple resources to be updated in a single request, as discussed below. Updates to multiple resources **MUST** completely succeed or fail. No partial updates are allowed.

Any requests that contain content **MUST** include a `Content-Type` header whose value is `application/vnd.api+json`.

### Creating Resources <a href="#updating-creating-resources" id="updating-creating-resources" class="headerlink">¶</a>

A server that supports creating resources **MUST** support creating individual resources and **MAY** optionally support creating multiple resources in a single request.

One or more resources can be *created* by making a `POST` request to the URL that
represents a collection of resources to which the new resource should belong.

#### Creating an Individual Resource <a href="#creating-a-resource" id="creating-a-resource" class="headerlink">¶</a>

To create an individual resource, its document **MUST** be sent as the request document.

For instance, a new photo might be created with the following request:

```text
POST /photos
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "title": "Ember Hamster",
  "src": "http://example.com/images/productivity.png"
}
```

#### Creating Multiple Resources <a href="#creating-multiple-resources" id="creating-multiple-resources" class="headerlink">¶</a>

To create multiple resources, an array of primary documents **MUST** be sent as the request document.

For instance, multiple photos might be created with the following request:

```text
POST /photos
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

[{
  "title": "Ember Hamster",
  "src": "http://example.com/images/productivity.png"
}, {
  "title": "Mustaches on a Stick",
  "src": "http://example.com/images/mustaches.png"
}]
```

#### Client-Side IDs <a href="#updating-creating-a-document-client-side-ids" id="updating-creating-a-document-client-side-ids" class="headerlink">¶</a>

A server **MAY** require a client to provide IDs generated on the
client. If a server wants to request client-generated IDs, it **MUST**
include a `meta` section in all of its responses with the key
`client-ids` and the value `true`:

```text
GET /photos

HTTP/1.1 200 OK
Content-Type: application/vnd.api+json

{
  "photos": [{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "title": "Mustaches on a Stick",
    "src": "http://example.com/images/mustaches.png"
  }],
  "meta": {
    "client-ids": true
  }
}
```

If the server requests client-generated IDs, the client **MUST** include
an `id` key in its `POST` request, and the value of the `id` key
**MUST** be a properly generated and formatted *UUID* provided as a JSON
string.

```text
POST /photos
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Ember Hamster",
  "src": "http://example.com/images/productivity.png"
}
```

#### Response <a href="#updating-creating-a-document-response" id="updating-creating-a-document-response" class="headerlink">¶</a>

A server **MUST** respond to a successful document creation request
according to [`HTTP semantics`](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-6.3)

The response **MUST** include a `Location` header identifying the location of the primary resource created by the request. If more than one resource is created, the `Location` URL must locate all created resources.

The response **SHOULD** also include a document that contains the primary resource(s) created. If absent, the client **SHOULD** treat the transmitted document as accepted without modification.

```text
HTTP/1.1 201 Created
Location: http://example.com/photos/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/vnd.api+json

{
  "photos": [{
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "href": "http://example.com/photos/550e8400-e29b-41d4-a716-446655440000",
    "title": "Ember Hamster",
    "src": "http://example.com/images/productivity.png"
  }]
}
```

##### Other Responses <a href="#updating-creating-a-document-response-other-responses" id="updating-creating-a-document-response-other-responses" class="headerlink">¶</a>

Servers **MAY** use other HTTP error codes to represent errors.  Clients
**MUST** interpret those errors in accordance with HTTP semantics.

### Updating Resources <a href="#updating" id="updating" class="headerlink">¶</a>

A server that supports updating resources **MUST** support updating individual resources and **MAY** optionally support updating multiple resources in a single request.

Resources can be updated by making a `PUT` request to the URL that represents either the individual or multiple individual resources.

#### Updating an Individual Resource

To update an individual resource, send a `PUT` request to the URL that represents the resource. The request **MUST** include a single resource document.

For example:

```text
PUT /articles/1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "title": "To TDD or Not"
}
```

#### Updating Multiple Resources

To update multiple resources, send a `PUT` request to the URL that represents the multiple individual resources (NOT the entire collection of resources). The request **MUST** include a collection of resource documents that each contain an `"id"`.

For example:

```text
PUT /articles/1,2
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

[{
  "id": "1",
  "title": "To TDD or Not"
}, {
  "id": "2",
  "title": "LOL Engineering"
}]
```

#### Updating Attributes

To update one or more attributes of a resource, the primary document should include only the attributes to be updated. Attributes ommitted from the document should not be updated.

For example, the following `PUT` request will only update the `title` and `text` properties of an article:

```text
PUT /articles/1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "title": "To TDD or Not",
  "text": "TLDR; It's complicated... but check your test coverage regardless."
}
```

#### Updating Relationships

##### Updating To-One Relationships

To-one relationships **MAY** be updated along with other attributes by including them in a `links` object within the document in a `PUT` request.

For instance, the following `PUT` request will update the `title` and `author` of an article:

```text
PUT /articles/1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "title": "Rails is a Melting Pot",
  "links": {
    "author": "1"
  }
}
```

In order to remove a to-one relationship, specify `null` as the value of the relationship.

Alternatively, a to-one relationship **MAY** optionally be accessible at its relationship URL (see above).

A to-one relationship **MAY** be added by sending a `POST` request to the URL of the relationship. The key should match the name of the relationship and the value should either be an ID or a "to-one link document" (see above). For example:

```text
POST /articles/1/links/author
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "author": "12"
}
```

A to-one relationship **MAY** be removed by sending a `DELETE` reqest to the URL of the relationship. For example:

```text
DELETE /articles/1/links/author
```

##### Updating To-Many Relationships

To-many relationships **MAY** optionally be updated with other attributes by
including them in a `links` object within the document in a `PUT` request.

For instance, the following `PUT` request performs a complete replacement of
the `tags` for an article:

```text
PUT /articles/1
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "title": "Rails is a Melting Pot",
  "links": {
    "tags": ["2", "3"]
  }
}
```

In order to remove every member of a to-many relationship, specify an empty array (`[]`) as the value of the relationship.

Replacing a complete set of data is not always appropriate in a
distributed system which may involve many editors.
An alternative is to allow relationships to be added and removed
individually.

To facilitate fine-grained access, a to-many relationship **MAY** optionally be accessible at its relationship URL (see above).

A to-many relationship **MAY** be added by sending a `POST` request to the URL of the relationship. The key should match the name of the relationship and the value should either be an array of IDs, a single "to-many link document", or an array of "to-one link documents". For example:

```text
POST /articles/1/links/comments
Content-Type: application/vnd.api+json
Accept: application/vnd.api+json

{
  "comments": ["1", "2"]
}
```

To-many relationships **MAY** be deleted individually by sending a `DELETE` request to the URL of the relationship:

```text
DELETE /articles/1/links/comments/1
```

Multiple to-many relationships **MAY** be deleted by sending a `DELETE` request to the URL of the relationships:

```text
DELETE /articles/1/links/comments/1,2
```

When deleting heterogenous relationships, it is necessary to represent the **type** of the linked resource along with its ID. This **SHOULD** be done by representing the ID as `<type>:<ID>`.

## Deleting Resources

An individual resource can be *deleted* by making a `DELETE` request to the resource's URL:

```text
DELETE /photos/1
```

A server **MAY** optionally allow multiple resources to be *deleted* with a `DELETE` request to their URL:

```text
DELETE /photos/1,2,3
```

### 204 Responses

If a server returns a `204 No Content` in response to a `DELETE`
request, it means that the deletion was successful.

### Other Responses

Servers **MAY** use other HTTP error codes to represent errors.  Clients
**MUST** interpret those errors in accordance with HTTP semantics.


## PATCH Support (Optional)

JSON API servers **MAY** opt to support HTTP `PATCH` requests that conform to the JSON Patch format [[RFC6902](http://tools.ietf.org/html/rfc6902)]. There are JSON Patch equivalant operations for the operations described above that use `POST`, `PUT` and `DELETE`. From here on, JSON Patch operations sent in a `PATCH` request will be referred to simply as "`PATCH` operations".

`PATCH` requests **MUST** specify a `Content-Type` header of `application/json-patch+json`.

Each `PATCH` operation is fine grained and updates one resource or attribute.

A server **MAY** support multiple `PATCH` operations by allowing more than one operation in the request document.

### Request URLs

The URL for each `PATCH` request **SHOULD** map to the resource(s) or relationship(s) to be updated.

The request URL and the `PATCH` operation's `"path"` are complementary and combine to target a particular resource, collection, attribute, or relationship. Therefore, every `"path"` within a `PATCH` operation should be relative to the request URL.

`PATCH` operations **MAY** also be allowed at the root URL of an API. In this case, every `"path"` within a `PATCH` operation must include the full resource URL. This allows for general "fire hose" updates to any resource represented by an API.

### Creating a Resource with PATCH

To create a resource, perform an `"add"` operation with a `"path"` that points to the end of the resource collection (`"/-"`). The `"value"` should contain a document representing the new resource.

For instance, a new photo might be created with the following request:

```text
PATCH /photos
Content-Type: application/json-patch+json
Accept: application/vnd.api+json

[
  { 
    "op": "add", 
    "path": "/-", 
    "value": {
      "title": "Ember Hamster",
      "src": "http://example.com/images/productivity.png"
    }
  }
]
```

### Updating Attributes with PATCH

To update an attribute, perform a `replace` operation with the attribute's name specified as the `"path"`.

For instance, the following request should update just the `src` property of the photo at `/photos/1`:

```text
PATCH /photos/1
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/src", "value": "http://example.com/hamster.png" }
]
```

### Updating Relationships with PATCH

Relationship updates are represented as `PATCH` operations on the `links` document.

#### Updating To-One Relationships with PATCH

To update a to-one relationship, perform a `replace` operation with `/links/<name>` specified as the `"path"`. The `"value"` should either be an ID or a "to-one link document" (see above). 

For instance, the following request should update the `author` of the article at `/articles/1`:

```text
PATCH /article/1
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/links/author", "value": "1" }
]
```

To remove a to-one relationship, perform a `remove` operation with the same `"path"`. For example:

```text
PATCH /article/1
Content-Type: application/json-patch+json

[
  { "op": "remove", "path": "/links/author" }
]
```

#### Updating To-Many Relationships with PATCH

While to-many relationships are represented as a JSON array in a `GET`
response, they are updated as if they were a set.

To add an element to a to-many relationship, perform a `add` operation with `/links/<name>/-` specified as the `"path"`. The `"value"` should either be an array of IDs, a single "to-many link document", or an array of "to-one link documents".

For example, for the following `GET` request:

```text
GET /photos/1
Content-Type: application/vnd.api+json

{
  "links": {
    "comments": "http://example.com/comments/{comments}"
  },
  "data": [{
    "id": "1",
    "href": "http://example.com/photos/1",
    "title": "Hamster",
    "src": "images/hamster.png",
    "links": {
      "comments": [ "1", "5", "12", "17" ]
    }
  }]
}
```

You could move comment 30 to this photo by issuing an `add` operation in
the `PATCH` request:

```text
PATCH /photos/1
Content-Type: application/json-patch+json

[
  { "op": "add", "path": "/links/comments/-", "value": "30" }
]
```

To remove a to-many relationship, perform a `remove` operation on `links/<name>/<id>`.

For example, to remove comment 5 from this photo, issue a `remove` operation:

```text
PATCH /photos/1
Content-Type: application/json-patch+json

[
  { "op": "remove", "path": "/links/comments/5" }
]
```

When removing heterogenous relationships, it is necessary to represent the **type** of the linked resource along with its ID. This **SHOULD** be done by representing the ID as `<type>:<ID>`.

### Deleting a Resource with PATCH

To deleting a resource, perform an `"remove"` operation with a `"path"` that points to the resource's root (`"/"`).

For instance, photo 1 might be deleted with the following request:

```text
PATCH /photos/1
Content-Type: application/json-patch+json
Accept: application/vnd.api+json

[
  { "op": "remove", "path": "/" }
]
```

### Responses to PATCH

#### 204 No Content <a href="#updating-a-document-204-no-content" id="updating-a-document-204-no-content" class="headerlink">¶</a>

If a server returns a `204 No Content` in response to a `PATCH` request,
it means that the update was successful, and that the client's current
attributes remain up to date.

#### 200 OK <a href="#updating-a-document-200-ok" id="updating-a-document-200-ok" class="headerlink">¶</a>

If the server accepts the update but also changes the document in other
ways than those specified by the `PATCH` request (for example, updating
the `updatedAt` attribute or a computed `sha`), it **MUST** return a
`200 OK` response.

The body of the response **MUST** be a valid JSON API response, as if a
`GET` request was made to the same URL.

#### Other Responses <a href="#updating-a-document-other-responses" id="updating-a-document-other-responses" class="headerlink">¶</a>

Servers **MAY** use other HTTP error codes to represent errors.  Clients
**MUST** interpret those errors in accordance with HTTP semantics.

## HTTP Caching <a href="#http-caching" id="http-caching" class="headerlink">¶</a>

Servers **MAY** use HTTP caching headers (`ETag`, `Last-Modified`) in
accordance with the semantics described in HTTP 1.1.

## Compound Responses <a href="#compound-responses" id="compound-responses" class="headerlink">¶</a>

Whenever a server returns a `200 OK` response in response to a creation,
update or deletion, it **MAY** include other documents in the JSON
document. The semantics of these documents are the same as when
additional documents are included in response to a `GET`.
