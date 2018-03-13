# Creating your own module

Here we will provide an example of a Comunica module,
showing all the components necessary so a module can be used.
We will create a simple HTTP module, which takes an URL and optional metadata as input,
and returns a response stream together with the resulting metadata.

If a module ever wants to do an HTTP call, it can't call our HTTP module directly,
it has to call a Mediator that has a Bus containing one or more HTTP Actors.
Since these would all be contained in the same Bus,
it is important that they accept the same input and produce the same kind of output.
To this end, before actually implementing an HTTP module,
we first have to define what an HTTP module would look like,
so all implementations can make use of that interface.

For comunica, the `bus-http` contains the relevant interfaces (and a default HTTP bus),
while `actor-http-node-fetch` contains an implementation of an HTTP actor,
making use of the `node-fetch` library.
Some of the code in this tutorial will be simplified.

## bus-http
All code for this module can be found
[here](https://github.com/comunica/comunica/tree/master/packages/bus-http).
This is the module that will define what all HTTP actors should accept as input/output.
Should you want to write a module that differs from the interfaces that already exist,
you would first have to write such a bus module defining your new interfaces.

The following are the TypeScript interfaces found in this module:

### Input interface
```typescript
export interface IActionHttp extends IAction {
  url: string;
  method?: string;
  headers?: string[];
  // ...
}
```
The `IActionHttp` interface defines what the input for HTTP modules should look like,
which, as mentioned before, consists of the URL and some metadata.
All input interfaces should extend the (empty) `IAction` interface.

### Output interface
```typescript
export interface IActorHttpOutput extends IActorOutput, IBody {
  url: string;
  status: number;
  // ...
}

export interface IBody {
  body: NodeJS.ReadableStream;
  // ...
}
```
Similarly, the IActorHttpOutput interface defines what the output should look like.
In this case, it is an extension of both `IActorOutput`,
which is required for all output interfaces,
and the `IBody` interface (which is specific for the HTTP case).

### Abstract class
```typescript
export abstract class ActorHttp extends Actor<IActionHttp, IActorTest, IActorHttpOutput> {
  constructor(args: IActorArgs<IActionHttp, IActorTest, IActorHttpOutput>) {
    super(args);
  }
}
```
Those two interfaces defined before allow us to define the abstract `ActorHttp` class.
Note that the more specific generics are used now to extend the `Actor` base class.
The original `IActorTest` is still used though, allowing specific implementations
to decide independently which cost interfaces they can support.
(TODO: link to cost interfaces explanation).
This is the class that defines what all HTTP modules will look like:
if someone writes an HTTP module they should inherit from this class
(or write a new abstract class and accept that it will be incompatible with other HTTP modules).

### Bus
A default Bus is defined, making it easier to configure all HTTP interfaces
to subscribe to the same bus.
This Bus can be found in `components/Bus/Http.jsonld`.

```json
{
  "@context": "https://linkedsoftwaredependencies.org/contexts/comunica-bus-http.jsonld",
  "@id": "cbh:Bus/Http",
  "@type": "cc:Bus",
  "comment": "A bus for HTTP request events"
}
```

Note that defining a Bus is quite easy due to its lack of requirements.

### Actor configuration
There is also a config file defining the abstract `ActorHttp` class,
allowing it to be used by the `components.js` library.
```json
{
  "@context": [
    "https://linkedsoftwaredependencies.org/contexts/comunica-bus-http.jsonld",
    "https://linkedsoftwaredependencies.org/contexts/comunica-core.jsonld"
  ],
  "@id": "npmd:@comunica/bus-http",
  "components": [
    {
      "@id": "cbh:Actor/Http",
      "@type": "AbstractClass",
      "extends": "Actor",
      "requireElement": "ActorHttp",
      "comment": "A base actor for handling HTTP events.",
      "parameters": [
        {
          "@id": "cc:Actor/bus",
          "defaultScoped": {
            "defaultScope": "cbh:Actor/Http",
            "defaultScopedValue": { "@id": "cbh:Bus/Http" }
          }
        }
      ],
      "constructorArguments": [
        {
          "@id": "cbh:Actor/Http/constructorArgumentsObject",
          "extends": "cc:Actor/constructorArgumentsObject"
        }
      ]
    }
  ]
}
```

An in-depth description of these configuration files can be found in the components.js
[documentation](http://componentsjs.readthedocs.io/en/latest/).
This file defines what could already be seen in the TypeScript example above,
while the `requireElement` value specifically points to the TypeScript file itself.
Additionally, the HTTP Bus defined above is set as a default value for the bus parameter,
in case no other Bus gets assigned.

## mediatortype-time
Remember that an Actor has to return metadata when its *test* function gets called.
In this case, our implementation returns an estimate of how long it will take (in milliseconds).
To this end, we need a [Mediator Type](../core.md) that defines *time*.
The `mediatortype-time` library contains a single file defining that type:
```typescript
import {IActorTest} from "@comunica/core";
export interface IMediatorTypeTime extends IActorTest {
  time?: number;
}
```
It's that easy!

## actor-http-node-fetch

The actual implementation of the HTTP actor is quite simple:

```typescript
import {ActorHttp, IActionHttp, IActorHttpOutput} from "@comunica/bus-http";
import {IActorArgs} from "@comunica/core";
import {IMediatorTypeTime} from "@comunica/mediatortype-time";
import fetch from "node-fetch";

export class ActorHttpNodeFetch extends ActorHttp {

  constructor(args: IActorArgs<IActionHttp, IMediatorTypeTime, IActorHttpOutput>) {
    super(args);
  }

  public async test(action: IActionHttp): Promise<IMediatorTypeTime> {
    return { time: Infinity };
  }

  public run(action: IActionHttp): Promise<IActorHttpOutput> {
    return fetch(action.url, action);
  }

}
```

All required functions have been overloaded and the `IMediatorTypeTime` is used as Mediator type.
The *test* function returns the expected type of metadata
(although in this case the estimate is not that useful),
while the *run* function returns the result of `node-fetch`,
which itself has an output that matches what is expected.

The class also has to be defined in the `components.js` format
so it can be integrated with the rest of the system.
This configuration is quite similar to the others:

```json
{
  "@context": [
    "https://linkedsoftwaredependencies.org/contexts/comunica-actor-http-node-fetch.jsonld",
    "https://linkedsoftwaredependencies.org/contexts/comunica-bus-http.jsonld"
  ],
  "@id": "npmd:@comunica/actor-http-node-fetch",
  "components": [
    {
      "@id": "chf:Actor/Http/NodeFetch",
      "@type": "Class",
      "extends": "cbh:Actor/Http",
      "requireElement": "ActorHttpNodeFetch",
      "comment": "A node-fetch actor that listens on the 'http' bus.",
      "constructorArguments": [
        {
          "extends": "cbh:Actor/Http/constructorArgumentsObject"
        }
      ]
    }
  ]
}
```

The actual implementation doesn't differ much from the abstract class,
meaning that the interface requires few changes.

## Summary
Writing a new module for Comunica is quite easy if there already is an existing interface:
only the implementation file has to be written, based on the interfaces given by the abstract class.
On the other hand, it is also possible to completely define a new type of module by first creating 
the generic interface for that type of modules.
Additionally, `component.js` config files also have to be written for the implementations.