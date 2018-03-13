# Creating a Comunica project configuration

Before a Comunica instance can be created,
a configuration file is needed.
Such a file defines all the modules that should be included in the instance,
and how they should be linked to one another.

Here we will cover the requirements for such a configuration and how to build one.
Examples can be found in the currently existing `init` actors,
such as the
[configurations](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql/config)
of `actor-init-sparql`.

## Format
Although we call them configuration files, they are actually just a collection of triples.
This means that you can also use a turtle file or an API that returns a set of ntriples.
All our examples will be in JSON-LD, but can easily be replaced by other formats.

## Runner
Every Comunica instance starts from a
[Runner](https://github.com/comunica/comunica/tree/master/packages/runner).
The Runner module is responsible for loading up the given modules and setting everything up.
It is also responsible for sending the input actions to the actors listening to its Bus,
which is the `init` Bus by default. All other actors are linked to the Runner as follows:

```json
{
  "@context": [ 
    "https://linkedsoftwaredependencies.org/contexts/comunica-runner.jsonld",
    "https://linkedsoftwaredependencies.org/contexts/comunica-core.jsonld",
    ...
  ],
  "@graph": [
    {
      "@id": "urn:comunica:my",
      "@type": "Runner",
      "actors": [ ... ]
}
```

All other actors are then contained in the `actors` array.
At least one of these actors should be listening to the `init` bus,
to receive the input actions and handle those.

The context files allow for more readable configuration,
such as being able to write `actors` instead of 
`https://linkedsoftwaredependencies.org/bundles/npm/@comunica/runner/actor`.

## Defining an Actor
The minimum requirement for having an Actor loaded by the Runner,
is to define an object of the given type, e.g.:
```json
    {
      "@id": "ex:mySparqlParser",
      "@type": "ActorSparqlParseAlgebra"
    }
```
It is a good idea to also add an `@id` since otherwise it would be hard to reference that Actor,
but this is actually not always required, 
e.g., if there are no direct references at all.

In this case an `ActorSparqlParseAlgebra` Actor would be created,
with all the default settings of that class.

## Setting Actor parameters
All default, and undefined, parameters can be changed in the config
Let's take an example from the default `actor-init-sparql`
[config](https://github.com/comunica/comunica/blob/master/packages/actor-init-sparql/config/config-default.json).

```json
  {
    "@id": "urn:comunica:sparqlinit",
    "@type": "ActorInitSparql",
    "mediatorQueryOperation": {
      "@id": "ex:mediatorQueryOperation",
      "@type": "MediatorRace",
      "cc:Mediator/bus": { "@id": "cbqo:Bus/QueryOperation" }
    }, ...
  }
```

Here you can see how the `mediatorQueryOperation` parameter
of the `ActorInitSparql` Actor gets defined,
which is again a new type, `MediatorRace`, with its own parameters.
Note that we can use `mediatorQueryOperation` instead of the full URI due to the context files again.
Since this is JSON-LD, other actors that would need the same mediator
can just reference to it by ID:
```json
  {
    "@id": "ex:myUnionQueryOperator",
    "@type": "ActorQueryOperationUnion",
    "cbqo:mediatorQueryOperation": { "@id": "ex:mediatorQueryOperation" }
  }
```
Which references the same mediator `ex:mediatorQueryOperation`.

Many Busses have already been pre-defined in the `bus-...` packages,
such as the `cbqo:Bus/QueryOperation` Bus here,
which was defined in the `bus-query-operation` package
and is the default bus of all query operation Actors.

## But which actors do I put in the configuration file now?
Well that is the beauty of Comunica:
it is completely up to you!
Besides the Runner, all other Actors are optional.
So you should include all Actors that are necessary to solve your problem.

The components file of every individual Actor will always indicate
which requirements it has so can be used to see which other Actors need to be added.
Besides that Comunica will also throw an error if it has an action
for which it finds no Actor that can solve it,
which can also be used to find out what more would need to be added.

The easiest is always to start from existing configuration files and adapt those,
so we recommend starting there.
