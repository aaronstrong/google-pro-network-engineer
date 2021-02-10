# CDN

1. [Overview](#Overview)
1. [Cache Keys](#Cache-Keys)


## Overview

![](https://cloud.google.com/cdn/images/cdn-response-flow.svg)

The purpose of the Content Delivery Network is to store a cached version of content closer to the user to accelerate your applications and websites.

CDN Requirements:

* Requires the use of Google Premium Network to provide the AnyCast IP Address
* A Global HTTP Load Balancer
* Edge locations for cache server

## Terms

![](https://cloud.google.com/cdn/images/cache-hit.svg)
Cache hit and Cache Miss

* Cache Hit

  If the GFE (Google Frontend) looks in the Cloud CDN cache and finds a cached response to the user's request, the GFE sends the cached response to the user. This is called a cache hit.

* Cache Miss

  The first time that a piece of content is requested, the GFE determines that it can't fulfill the request from the cache. This is called a cache miss.

![](https://cloud.google.com/cdn/images/cache-fill.svg)
Cache fill and cache egress

* Cache Fill

  Data transfer to a cache is called cache fill. 

* Cache Egress

  Data transfer from a cache to a client is called cache egress.

## [Cache Keys](https://cloud.google.com/cdn/docs/caching#cache-keys)

Each cache entry in a Cloud CDN cache is identified by a cache key.When a request comes into the cache, the cache converts the URI of the request into a cache key, and then compares it with keys of cached entries. If it finds a match, the cache returns the object associated with that key.


## Cache Expiration

## Cache Validation