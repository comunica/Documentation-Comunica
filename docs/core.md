# Core components

Here we will describe the 3 core components of Comunica.
The internal communication of Comunica works by combining those componens:

1. An Actor requests a response through a Mediator.
2. The Mediator *tests* all Actors in its Bus.
3. Every Actor responds with metadata describing the estimated costs for answering the request.
4. Based on this list of metadata, the Mediator picks the best Actor.
5. That Actor gets to execute the request.
6. The resulting response gets returned to the original request.

## Actor

Actors are the main components that will be executing the required actions.
Most programs will have several Actors executing their actions
and sending their results to each other through the Buses.
The following is a simplified view of the Actor interface.
For a full view we reference the JSDoc documentation (TODO: make that).

```typescript
abstract class Actor<I, T, O> {

  public name: string;
  public bus: Bus<Actor<I, T, O>, I, T, O>;

  constructor(args) {
    // assign parameters and subscribe to bus
  }
  public abstract async test(action: I): Promise<T>;
  public abstract async run(action: I): Promise<O>;
}
```

With I, T and O corresponding to the expected Input, TestResult and Output formats.
As can be seen, every Actor has a name (TODO: what did this correspond to again?),
and a bus it is subscribed to.
The interfaces for T and O are empty, meaning there are no restrictions on the format there.

Besides the constructor, there are 2 functions that every Actor has to inherit: *test* and *run*.

The *test* function is used by Mediators to determine which Actor to call.
As mentioned previously, Mediators have the job of choosing
which Actor will get to execute a given task.
For this they base themselves on metadata provided by the subscribed Actors,
which is provided through the test function.
The Mediator provides the input of the task to each Actor.
Each of those Actors than needs to respond with `null`
if they are unable to provide results for the given input,
or with a metadata object indicating the costs required to solve that task.
"Costs" is left vague on purpose, allowing different Mediators to focus on different costs,
such as total execution time, # http calls, etc.
It is the responsibility of the Actor to provide decent estimates here.

The *run* function corresponds to actually executing the task this Actor was made for.
Once a Mediator chooses an Actor, this function will be called with the same input as was provided
in the *test* call.

TODO: maybe also include corresponding jsonld here? or make separate components section?

## Bus

A Bus is an aggregation of Actors,
providing several helper functions.
Since the tasks of a Bus are quite simple,
it is usually not needed to extend its functionality
and can be instantiated directly if a new Bus is required.

```typescript
class Bus<A, I, T, O> {

  public name: string;
  protected actors: A[] = [];

  constructor(args) {
    // assign parameters
  }

  public subscribe(actor: A) {
    // subscribe actor
  }

  public unsubscribe(actor: A) {
    // unsubscribe actor
  }

  public publish(action: I): IActorReply<A, I, T, O>[] {
    // test all actors
  }
}

interface IActorReply<A, I, T, O> {
  actor: A;
  reply: Promise<T>;
}
```

The *subscribe*/*unsubscribe* functions are used to add/remove Actors.
The main function for mediators is *publish*,
which requests the test results from all Actors based on the given output.
The result is an array of objects containing the results an corresponding actors.

## Mediator

Mediators have the responsibility of determining which Actors need to be executed
in case multiple options are possible.
First the test metadata gets requested from all Actors,
and based on that the Mediator makes a decision on which Actor to execute.

```typescript
abstract class Mediator<A, I, T, O> {

  public name: string;
  public bus: Bus<A, I, T, O>;

  constructor(args) {
      // assign parameters
  }

  public publish(action: I): IActorReply<A, I, T, O>[] {
    // test all actors
  }

  public async mediateActor(action: I): Promise<A> {
    // test all actors and return the best one
  }

  public async mediate(action: I): Promise<O> {
    // find the best actor and run it
  }

  protected abstract async mediateWith(action: I, testResults: IActorReply<A, I, T, O>[]): Promise<A>;
}
```

When creating a Mediator, only the *mediateWith* function needs to be overloaded.
This function determines what the *best* Actor is, based on the given metadata.
Besides that there are no restrictions on how to actually choose this,
that is completely up to the implementation.