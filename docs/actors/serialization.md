# Serialization actors
The final step when generating results is presenting them to the user.
There are actors available for many of the popular formats,
allowing the option to choose based on the user's preference.

Supported parsing formats:

 * [JSON-LD](https://github.com/comunica/comunica/tree/master/packages/actor-rdf-parse-jsonld)
 * [N3](https://github.com/comunica/comunica/tree/master/packages/actor-rdf-parse-n3)

Supported bindings serialization formats:

 * [JSON](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-json)
 * [Simple](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-simple)
 * [SPARQL JSON](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-sparql-json)
 * [SPARQL XML](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-sparql-xml)
 * [Table](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-table)

Additional serializers:

 * [Stats](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-stats) serializer for testing and debugging.
 * [RDF](https://github.com/comunica/comunica/tree/master/packages/actor-sparql-serialize-rdf) serializer for triple results (e.g., when running a CONSTRUCT query).

All these implementations start from the [`@comunica/actor-abstract-mediatyped`](https://github.com/comunica/comunica/tree/master/packages/actor-abstract-mediatyped)
package, which provides functionality for all classes that have functionality based on media types.
The supported media types are defined in the constructor, as shown in the config file for N3
[here](https://github.com/comunica/comunica/blob/master/packages/actor-rdf-parse-n3/components/Actor/RdfParse/N3.jsonld).
It also takes care of testing if the input actions have the correct media types and
provides support for returning all applicable media types.
The only thing subclasses have to implement is the `testHandleChecked` function to do any remaining tests after checking media types,
and the `runHandle` function that does the actual conversion (either parsing or serializing).

## Parsing and Serializing
The default classes and bus for parsing can be found in [`@comunica/bus-rdf-parse`](https://github.com/comunica/comunica/tree/master/packages/bus-rdf-parse),
while those for serializing can be found in [`@comunica/bus-rdf-serialize`](https://github.com/comunica/comunica/tree/master/packages/bus-rdf-serialize).
Both packages provide abstract classes that already implement `testHandleChecked` by simply returning true,
but it can still be overridden of course.