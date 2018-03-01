# Including Comunica in your project

There are 2 options to include Comunica in your project as a dependency:

1. Use one of the pre-composed modules, such as 
[`actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql).
2. Composing your own combination of Comunica modules.

The first instance is the easiest one and will usually only require a single install.
Details on how to use those pre-composed modules can be found in the corresponding documentation for each of those.

## Custom combinations

If you want to combine Comunica modules in a custom way,
there are a few requirements:

1. A Components.js configuration is required.
2. All components referenced in the configuration need to be installed (i.e. present in package.json).
3. The Comunica [Runner](https://github.com/comunica/comunica/tree/master/packages/runner)
module needs to be included as well.

We will not cover how to write a configuration, that can be found [here](configuration.md).

Depending on your needs there are multiple options:
you can do a single run of a configuration,
or you can set up an instance based on the configuration that can handle requests.
Both options require use of the `Setup` class from the Runner package.

### Single run
The `Setup.run` function takes a configuration,
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
TODO: link to jsdoc here

TODO: how good does this work with relative paths and stuff?

* `configResourceUrl`: the path (or URL) to the configuration file.
* `action`: the input for the 'init' Actor of the configuration.
* `runnerUri`: the URI of the runner in the configuration.
* `properties`: these are `components.js`
[Loader Options](http://componentsjs.readthedocs.io/en/latest/loading/loader/).

The result of this call is a promise returning the result.

### Set up Runner instance

TODO: could also link to actual code in git repo but line numbers can change

The easiest way to explain how to set up a Runner instance
is by showing the source of the `Setup.run` function:
```typescript
  public static async run(configResourceUrl: string, action: IActionInit, runnerUri?: string,
                          properties?: ISetupProperties): Promise<any> {
    if (!runnerUri) {
      runnerUri = 'urn:comunica:my';
    }

    const runner: Runner = await Setup.instantiateComponent(configResourceUrl, runnerUri, properties);
    await runner.initialize();
    let output: IActorOutputInit[];
    try {
      output = await runner.run(action);
    } catch (e) {
      await runner.deinitialize();
      throw e;
    }
    await runner.deinitialize();
    return output;
  }
```

To set up your own Runner instance, a similar flow has to be followed:

 1. Set up the Runner using `Setup.instantiateComponent`
 2. Intialize the Runner using `runner.initialize`
 3. Get results with `runner.run`. This can be done multiple times!
 4. When finished, deinitialize the runner with `runner.deinitialize`

The initialize/deiniatilize functions are required for certain actors
that have certain actions that need to be run once,
such as reading in files.