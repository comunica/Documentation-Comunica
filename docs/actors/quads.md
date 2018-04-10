# Quad pattern resolvers
The quad pattern actors return a stream of Bindings that correspond to all results matching a single triple pattern,
from one or more sources.
This is especially important when using the TPF algorithm since it is entirely based on combining single pattern results.
The default bus and interfaces can be found in [`@comunica/bus-rdf-resolve-quad-pattern`](https://github.com/comunica/comunica/tree/master/packages/bus-rdf-resolve-quad-pattern).


Actors extending `ActorRdfResolveQuadPatternSource` should implement a `test` function as usual,
and besides that `getSource` and `getOutput`.
The source function can be used to prepare a source should that be required,
while the output function is then used to extract a stream of triples from the given source object.