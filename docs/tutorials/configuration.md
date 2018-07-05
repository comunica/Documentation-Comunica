# Creating a Comunica project configuration

The Comunica engine is always instantiated using a _configuration file_.
This files describes the components that should be loaded into the engine, and how they are wired.

In this tutorial, we cover the requirements for such a configuration and how to create one yourself.
Examples can be found in the currently existing `init` actors,
such as the [configurations of `actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql/config).

!!! note
    Although we call them configuration files, they are actually just a collection of RDF triples.
    This means that you can for example also use a Turtle file or an API that returns a set of N-Triples.
    All our examples will be in JSON-LD, as JSON is easier to use for non-RDF experts, but they can easily be replaced by other formats.

!!! note
    Throughout this document, we reuse the [terminology of Components.js](http://componentsjs.readthedocs.io/en/latest/getting_started/basics/terminology/)
    where a *component* is a class that can be instantiated, and a *module* is a collection of components.

!!! note
    These configuration files are Components.js configuration files.
    As such, the more advanced features from [Components.js](http://componentsjs.readthedocs.io/en/latest/configuration/configurations/semantic/) can be used here as well.

## Runner
Every Comunica engine is started from a
[Runner](https://github.com/comunica/comunica/tree/master/packages/runner).
The Runner module is responsible for loading the modules from a configuration and wiring them.
It is also responsible for sending the input actions to the actors listening to its Bus,
which is the `init` Bus by default. All other actors are linked to the Runner as follows:

```json
{
  "@context": [ 
    "https://linkedsoftwaredependencies.org/bundles/npm/@comunica/runner/^1.0.0/components/context.jsonld"
    "https://linkedsoftwaredependencies.org/bundles/npm/@comunica/core/^1.0.0/components/context.jsonld"
    ...
  ],
  "@id": "urn:comunica:my",
  "@type": "Runner",
  "actors": [
      {
        "@id": "ex:myInitActor",
        "@type": "ActorThatInitializes"
      },
      {
        "@id": "ex:myRdfParserN3",
        "@type": "ActorRdfParseN3"
      },
      {
        "@id": "ex:myRdfParserJsonLd",
        "@type": "ActorRdfParseJsonLd",
        "priorityScale": 0.9
      },
      ...
}
```

All other actors must be contained in the `actors` array.
At least one of these actors should be listening to the `init` bus,
to receive the input actions and handle those.

!!! note
    The context files are required to abbreviate the full URIs of components.
    For example, the first context can be used to write `actors` instead of
    `https://linkedsoftwaredependencies.org/bundles/npm/@comunica/runner/actor`.

## Defining an Actor
The minimum requirement for having an Actor loaded by the Runner,
is to define a component of a certain type, e.g.:
```json
    {
      "@id": "ex:mySparqlParser",
      "@type": "ActorSparqlParseAlgebra"
    }
```
In this case an `ActorSparqlParseAlgebra` instance would be created,
with the default settings of that class.

!!! note
    While it is not required, the convention is to always add an `@id` to identify each Actor.
    This will make it easier to debug Runner loading failures or other errors that reference components.

## Configuring an Actor

Next to just including including components, the configuration file can also be used to configure the *parameters* of an actor.

The following example configures the JSON serialization indentation of an `ActorRdfSerializeJsonLd` using the `jsonStringifyIndentSpaces` parameter.
```json
    {
      "@id": "ex:myRdfSerializeJsonLd",
      "@type": "ActorRdfSerializeJsonLd",
      "jsonStringifyIndentSpaces": 2
    },
```

Let's take a more complex example from the default `actor-init-sparql`
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
of the `ActorInitSparql` Actor is defined,
which is again a new type, `MediatorRace`, with its own parameters.
Note that we can use `mediatorQueryOperation` instead of the full URI due to the context files again.
Since this is JSON-LD, other actors that would need the same mediator
can just reference it by its `@id`:
```json
  {
    "@id": "ex:myUnionQueryOperator",
    "@type": "ActorQueryOperationUnion",
    "cbqo:mediatorQueryOperation": { "@id": "ex:mediatorQueryOperation" }
  }
```
Which references the same mediator `ex:mediatorQueryOperation`.

## Configuring buses

Buses typically don't require a separate implementation.
They can therefore directly be instantiated by the default Bus implementation.

Many Buses have already been pre-defined, such as the `cbqo:Bus/QueryOperation` Bus:
```json
{
  "@context": ""https://linkedsoftwaredependencies.org/bundles/npm/@comunica/bus-query-operation/^1.0.0/components/context.jsonld"",
  "@id": "cbqo:Bus/QueryOperation",
  "@type": "cc:Bus",
  "comment": "A comunica bus for query-operation events."
}
```

## But which actors do I put in the configuration file now?
Well that is the beauty of Comunica:
it is completely up to you!
Besides the Runner, all other Actors are optional.
So you should include all Actors that are necessary to solve your problem.

The easiest way is to start from existing configuration files and adapt those.

!!! note
    The applicable buses will typically require at least one registered actor.
    If no actor can be found to solve a certain action, Comunica will also throw an error at runtime.
