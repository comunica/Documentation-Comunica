# Including Comunica in your project

There are 2 options to include Comunica in your project as a dependency:

1. Use one of the pre-composed modules, such as 
[`actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql).
2. Composing your own combination of Comunica modules.

The first option is the easiest one and will usually only require a single install,
such as `npm install @comunica/actor-init-sparql`.
In this case, SPARQL querying within your application can be done as follows:

```javascript
const newEngine = require('@comunica/actor-init-sparql').newEngine;

const myEngine = newEngine();
myEngine.query('SELECT * { ?s ?p <http://dbpedia.org/resource/Belgium>. ?s ?p ?o } LIMIT 100',
  { sources: [ { type: 'hypermedia', value: 'http://fragments.dbpedia.org/2015/en' } ] })
  .then(function (result) {
    result.bindingsStream.on('data', function (data) {
      console.log(data.toObject());
    });
  });
```

Please refer to the [README of `actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql#usage-within-application)
or other pre-composed modules for more information on their installation.

## Custom combinations

If you want to combine Comunica modules in a custom way,
there are a few requirements:

1. A Components.js configuration.
2. All components referenced in the configuration need to be installed (i.e. present in package.json and installed in `node_modules`).
3. Dependency on the Comunica [Runner](https://github.com/comunica/comunica/tree/master/packages/runner).

A manual on how configurations are created can be found [here](configuration.md).

Depending on your needs there are multiple options:
you can do a single run of a configuration,
or you can set up an instance based on the configuration that can handle requests.
Both options require use of the `Setup` class from the Runner package.

### Single run
The [`Setup.run`](https://comunica.github.io/comunica/classes/_runner_lib_setup_.setup.html#run)
function takes a configuration,
loads all the corresponding modules,
run the instance on the given input,
and return the output.

This has the advantage of being easy to set up,
only a single call is required,
but has the disadvantage of having to load all the modules every time.

The `run` function has the following interface:
```typescript
public static async run(configResourceUrl: string, action: IActionInit, runnerUri?: string,
                        properties?: ISetupProperties): Promise<any>
```

* `configResourceUrl`: the path (or URL) to the configuration file.
* `action`: the input for the 'init' Actor of the configuration.
* `runnerUri`: the URI of the runner in the configuration.
* `properties`: these are `components.js`
[Loader Options](http://componentsjs.readthedocs.io/en/latest/loading/loader/).

The result of this call is a promise returning the result.

### Set up Runner instance

In case you will need to do multiple calls to Comunica,
you will want to set up a reusable instance,
thereby not having to load in the configuration every time.
For this we have the 
[`Setup.instantiateComponent`](https://comunica.github.io/comunica/classes/_runner_lib_setup_.setup.html#instantiatecomponent) 
function.

The interface is quite similar to the `run` function:
```typescript
public static async instantiateComponent(configResourceUrl: string, runnerUri: string,
                                           properties?: ISetupProperties): Promise<any>
```

All parameters are the same as above, except `action` is missing,
since that will be given separately once the instance is ready.

In this case the response is not the result of running an instance though,
the result is the actual Runner instance you will make requests against:
```typescript
const runner: Runner = await Setup.instantiateComponent(configResourceUrl, runnerUri, properties);
```

Once you have your runner, you can use its `run` function to send actions:
```typescript
const output = await runner.run(action);
```

The difference with `Setup.run` is that this will be much more peformant
if you execute multiple Comunica calls.

#### Initialize and deinitialize
Certain components have some actions that need to be executed once before they can be run,
e.g., opening up connections, which then also need to be stopped once the process is finished.
For this we have the `initialize` and `deinitialize` functions.
This means that both of these need to be called once each on the runner object,
`runner.initialize()` before the first call,
and `runner.deinitialize()` after the last call to clean up.

So in summary, your code should look similar to this:
```typescript
  const runner = await Setup.instantiateComponent(configResourceUrl, runnerUri, properties);
  runner.initialize();
  
  const result1 = await runner.run(action1);
  const result2 = await runner.run(action2);
  const result3 = await runner.run(action3);
  
  runner.deinitialize();
```

### TODO: Static compilation