# Actor subclasses
The [module tutorial](../tutorials/module.md) describes how to write an actor class in general.
But there are already several actor subclasses available,
which make it easier to write a new actor by inheriting there functionality.

In the following pages we will cover some of these subclasses.

 * [Algebra actors](algebra.md) resolve functions of a SPARQL algebra tree.
 * [Init actors](init.md) are the entrypoints into the Comunica framework.
 * [Join actors](join.md) can be used to join two or more streams.
 * [Metadata actors](metadata.md) parse a stream of triples to find metadata.
 * [Quad pattern resolvers](quads.md) return all bindings and metadata for a given pattern.
 * [Serialization actors](serialization.md) are responsible for parsing and transforming the results to a given output format.

!!! note
    It is not required to to make use of these interfaces when creating an actor fulfilling a similar function.
    These packages are just here to make implementation easier.