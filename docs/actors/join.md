# Join actors
Join actors can be used to join bindings streams.
Depending on the specific join actor used,
this can be any number of streams.
Although current existing actors only support the joining of 2 streams.

The bus and interface can be found in `@comunica/bus-rdf-join`.
The `ActorRdfJoin` abstract class requires a single constructor argument, `maxEntries`,
which defines the maximum amount of streams a join actor can combine.

The [abstract implementation](https://github.com/comunica/comunica/blob/master/packages/bus-rdf-join/lib/ActorRdfJoin.ts)
takes care of all the trivial cases,
combining 0 or 1 streams,
and also tests for valid input and generates the metadata.
Implementations should only implement a `getOutput` function that returns the joined result,
and a `getIterations` function that estimates the complexity of the execution.

Besides that the abstract class also provides multiple helper functions to aid in the joining of Bindings.
