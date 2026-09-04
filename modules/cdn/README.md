---
description: >-
  This module brings file management and image manipulation to Nails. Can be
  configured to use the local filesystem, or remote services for truly
  distributed content.
---

# CDN

The CDN module provides a unified interface for storing user generated content (i.e files), known as [objects](./#objects). Objects are organised in [buckets](./#buckets).

Objects can be uploaded via the [CDN Service](cdn-service.md), or via the [API](api/).

## Objects

Objects are synonomous with files in a traditional file system. They represent binary data which is stored in the CDN. Objects have meta data attached to them such as a title, mime type, file size etc.

## Buckets

Buckets can be though of as simialr to a folder in a traditional file system, they contain objects. They can only be one level deep, however: buckets cannot be nested and have no hierarchy.

## Drivers

Data storage is controlled through the use of drivers. Drivers allow the developer to leverage local storage, as well as storage as a service systems like S3, or Google Cloud Storage

{% content-ref url="drivers/" %}
[drivers](drivers/)
{% endcontent-ref %}

{% hint style="info" %}
All official drivers allow the end URL to be specified so that edges like CloudFront can be used to serve assets,
{% endhint %}

## CDN Service

Upload, delete, and fetch CDN objects and buckets using the CDN Service.

{% content-ref url="cdn-service.md" %}
[cdn-service.md](cdn-service.md)
{% endcontent-ref %}

## API

The CDN module exposes various API endpoints for querying the CDN and for uploading files.

{% content-ref url="api/" %}
[api](api/)
{% endcontent-ref %}

## Image Transformation

Simple image transformations can be made via the CDN module.

{% content-ref url="image-transformation.md" %}
[image-transformation.md](image-transformation.md)
{% endcontent-ref %}

## Console

The CDN module provides some utility console commands for managing objects from the command line.

{% content-ref url="console.md" %}
[console.md](console.md)
{% endcontent-ref %}
