# Actor subclasses
The [module tutorial](../tutorials/module.md) describes how to write an actor class in general.
But there are already several actor subclasses available,
which make it easier to write a new actor by inheriting there functionality.

In the following pages we will cover some of these subclasses.

 * [Algebra actors](algebra.md) resolve functions of a SPARQL algebra tree.
 * [Init actors](init.md) are the entrypoints into the Comunica framework.
 * [Join actors](join.md) can be used to join two or more streams.
 * [Metadata extractors](metadata.md) parse a stream of metadata triples to find specific entries.
 * [Quad pattern resolvers] return all bindings and metadata for a given pattern.
 * [Serialization actors] are responsible for transforming the results to a given output format.
