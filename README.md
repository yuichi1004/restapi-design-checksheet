# REST API Design Check Sheet 

This sheet is a collection of REST API best practices.
Each section has references to series of REST API design guidelines, RFCs, and/or articles which support the practice.
Please refer to the reference for ther further detail.

## TOC

- [Resource](#resource)
  - [Resource and Collection Resource](#resource-and-collection-resource)
- [Methods](#methods)
  - [Standard HTTP Methods](#standard-http-methods)
  - [POST](#post)
  - [PATCH](#patch)
- [Consistency](#consistensy)
  - [Etag](#etag)
  - [Optimistic Locking with Etag](#optimistic-locking-with-etag)
- [Cache](#cache)
  - [Cache Control](#cache-control)
  - [Cache Validation](#cache-validation)
- [Content Negotiation](#content-negotiation)
  - [Content Localization](#content-localization)

## Resource

- REST API should be resource oriented API described below.

### Resource and Collection Resource

- URI identifies a resource.
- A resource can be nested by collection resource.

```
GET /users/{user-id}
```

- A resource ID is prefered to be meaningful string.

```
GET /books/history-of-computer-science -- Prefered
GET /books/978-4532401234              -- OK
```

- A collection resouce ID SHOULD be plural.

```
GET /users/{user-id} -- Good
GET /user/{user-id}  -- Bad
```

- Use lower chars and hyphen to avoid URI encoding.

```
GET /purchase-order/1234
```

- Use persistent resouce ID.
- Avoid extra nested URLs if possible.

```
GET /skus/{sku-id}                        -- Prefered
GET /products/{product-id}/skus/{sku-id}  -- Use if {sku-id} can be unique with {product-id}
```

- References
  - [Google Cloud API - Resource Names](https://cloud.google.com/apis/design/resource_names)
  - [Microsoft API Design - Organize the API around resources](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design#organize-the-api-around-resources)
  - [Zalando RESTful API and Event Scheme Guidelines - Resources](https://opensource.zalando.com/restful-api-guidelines/#resources)

[TOC](#toc)

## Methods

### Standard HTTP Methods

- Use common HTTP method as verb
  - GET
  - POST
  - PUT
  - PATCH
  - DELETE
- Avoid using verb on URI as much as possible. Use it only if standard HTTP method does not cover.

```
POST /users        -- Good
POST /users/create -- Avoid
POST /users/login  -- Acceptable
```

[TOC](#toc)

### POST

- Use POST to create new entity
  - Use `201 Created` or `202 Accepted` instead of `200 OK`.

```
# Use 201 if object is created synchronously.
HTTP/1.1 201 Created

{"id": "1234", "name": abc"}
```
```
# Use 202 if the request is processed asynchronously.
HTTP/1.1 202 Accepted

{"jobId": "1234"}
```

[TOC](#toc)

### PATCH

- Use JSON Patch for PATCH request

```
PATCH /users/{user-id}

[
  { "op": "replace", "path": "/email", "value": "new-mail@mail.com" }
]
```

- References
  - [JavaScript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902)
  - [これからの Microservices (JP)](https://www.slideshare.net/zigorou/microservices-57643957/24)

[TOC](#toc)

## Consistency

### Etag

- Resource response should include `Etag` header.

```
HTTP/1.1 200 OK
Etag: w/"1234-5678-9012"

{ "id": "user.id", "name": "user name", "email": "email@address.com" }
```
```
GET /users/{user-id}
If-None-Match: w/"1234-5678-9012"
```

- References
  - [Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests](https://tools.ietf.org/html/rfc7232)
  - [Google Cloud API - Design Patterns](https://cloud.google.com/apis/design/design_patterns#etags)

[TOC](#toc)

### Optimistic Locking with Etag

- Use `If-Match` header to update entity with optimistic locking

```
Request:
PUT /books/art-of-computer-science
If-Match: w/"1234-5678-90"

{ "name": "art-of-computer-science-continued" }

Response 1 (Success):
HTTP/1.1 200 OK
Etag: w/"1234-5678-91"

{ ... }

Response 2 (Other client updated between the request):
HTTP/1.1 412 Precondition Failed

```

- References
  - [Zalando RESTful API and Event Scheme Guidelines - Optimistic Locking in RESTful APIs](https://opensource.zalando.com/restful-api-guidelines/#optimistic-locking)
  - [これからのMicroservices](https://www.slideshare.net/zigorou/microservices-57643957/36)

[TOC](#toc)

## Cache

### Cache Control 

- Use `Cache-Control` header.
- API SHOULD return `Cache-Control: no-cache` response header if does not support cache.

```
# use no-cache unless cache is supported
GET /books/art-of-computer-science
Cache-Control: no-cache, no-store

{ ... }
```

- Use `Vary` header if cache depends on request headers.

```
GET /books/art-of-computer-science
Accept-Language: en
```
```
HTTP 200 OK
Content-Language: en
Vary: Accept-Language

{...}
```

- Use `max-age` directive to specify how long the client keep the cache.
- Cache server MUST return `Age` response header with `max-age` directive.

```
HTTP/1.1 200 OK
Cache-Control: max-age=1800
Age: 623

{ ... }
```

- References
  - [Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234)
  - [HOW-TO: HTTP CACHING FOR RESTFUL & HYPERMEDIA APIS](http://www.apiacademy.co/how-to-http-caching-for-restful-hypermedia-apis/)

[TOC](#toc)

### Cache Validation

- GET resquest should support `If-None-Match` header for cache validation.
  - Cache validation can be used either:
    - `Cache-Control: no-cache` request is specified.
    - Reuse stale cached entity after `Age` exceeds `max-age`.

```
# Request: use currently cached entity's etag for If-None-Match header.
GET /books/art-of-computer-science
If-None-Match: w/"1234-5678-9012"
```
```
# Response 1: 302 not modified will be returned if the entity not changed.
HTTP/1.1 302 Not Modified
```
```
# Response 2: 200 OK will be returned if the entity is changed.
HTTP/1.1 302 Not Modified

{...}
```

- References
  - [Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234)
  - [HOW-TO: HTTP CACHING FOR RESTFUL & HYPERMEDIA APIS](http://www.apiacademy.co/how-to-http-caching-for-restful-hypermedia-apis/)

[TOC](#toc)

## Content Negotiation

### Content Localization

- Use `Accept-Language` to get localized resource.

```
GET /books/art-of-computer-science
Accept-Language: fr

{ "name": "art de l'informatique", ... }
```

- References
  - [Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231#section-5.3.5)
  - [これからの Microservices (JP)](https://www.slideshare.net/zigorou/microservices-57643957/30)

[TOC](#toc)
