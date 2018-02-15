This document describes the architecture of this project.

# Architecture

In general, this architecture allows certain tasks to be performed by one or more actors in a certain bus, and where a mediator can choose one or more appropriate actors to perform the actual task.

The main modules we have are the following:
* Bus: Accepts _messages_ and sends those to different _actors_.
* Actor: Performs a certain _action_, based on a _message_ from a _bus_. Can also send messages to other buses.
* Mediator: Wraps over a _bus_, to detect the best _actor_ for a certain task based on some conditions. For example, a certain mediator could find the fastest HTTP actor.
* Mediator type: A certain category of _mediators_. For example, different mediation implementations could exist for finding such a fastest HTTP actor.

A _message_ can either can either be a 'can'-message, or an 'action'-message.
A 'can'-message is sent to ask actors if they _can_ solve the task, while the 'action'-message tells them to actually perform the task.

# Modularity

Comunica-core only contains the implementation of the core classes of this architecture.
Each actor, mediator and mediator type must be a separate module.

We use the following naming convenient for (NPM) modules:
* Bus types: `@comunica/bus-[name-of-bus-type]`
* Mediator types: `@comunica/mediatortype-[name-of-mediator-type]`
* Mediators: `@comunica/mediator-[name-of-mediator]`
* Actors: `@comunica/actor-[name-of-bus-type]-[name-of-actor]`

# Technology

## Bluebird

Bus events are promise-based, so they happen asynchronously.
The Bluebird Promise implementation is used to make Promises _cancelable_,
which is useful when multiple actor tasks were started,
but only the result of the first one is needed.

## TypeScript

Easier development because of [static typing](https://www.npmjs.com/package/typescript).

## Components.js

[Modular component injection](https://www.npmjs.com/package/componentsjs),
with a semantic config file describing how each component should be wired.