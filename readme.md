> Shared vocabularies provide a powerful extensibility point for OData. <small>[from the OData spec][1]</small>

# Shared vocabularies

One of the many extensibility points in the OData protocol is the concept of vocabularies. A vocabulary is a collection of terms. Annotations are used to apply terms to OData elements.

Using the same vocabulary for a multitude of different services allows common interpretation of the content regardless of the specific structure and property names used. In concept this is quite similar to [schema.org][2].

## Central storage

This repository is a central clearinghouse for the OData community to work on vocabularies that are intended to be shared widely. The vocabularies are licensed under the MIT license, however, we encourage the community to make every effort to iterate on the shared vocabularies rather than forking them (as this defeats the purpose of a shared vocabulary).

## Current vocabularies

Currently the following vocabularies are shared:

* [Display][3]

[1]: http://docs.oasis-open.org/odata/odata/v4.0/errata02/os/complete/part1-protocol/odata-v4.0-errata02-os-part1-protocol-complete.html#_Toc406398214
[2]: http://schema.org/
[3]: https://github.com/OData/vocabularies/blob/master/OData.Community.Display.V1.md
