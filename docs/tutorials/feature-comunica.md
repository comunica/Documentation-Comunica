# Adding a feature to Comunica

There are two ways to add a feature to the Comunica framework
You can either:

1. Implement a feature in the Comunica monorepo project.

2. Implement a feature into a standalone project.

These two methods will be explained separately hereafter.

## Implement a feature in the Comunica monorepo project

If your feature makes sense to add in the standard set of supported Comunica features, then your feature can be added into the Comunica monorepo project, i.e., the [Comunica GitHub repository](https://github.com/comunica/comunica).

Examples of features that belong here are:

* Implementations of SPARQL operators.

* RDF parsers and serializers.

* Optimizations for certain querying algorithms.

### Initialize monorepo

The easiest way to get started is to clone the repository as follows:

```bash
$ git clone git@github.com:comunica/comunica
```

After that, you'll have to initialize the project and its dependencies as follows:

```bash
$ cd comunica
$ yarn install
```

This will install the dependencies of all modules, and bootstrap the Lerna monorepo. After that, all [Comunica packages](https://github.com/comunica/comunica/tree/master/packages)can be used in a development environment, such as querying with [Comunica SPARQL](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql).

Furthermore, this will add [pre-commit hooks](https://www.npmjs.com/package/pre-commit) to build, lint and test. These hooks can temporarily be disabled at your own risk by adding the `-n` flag to the commit command.

### Creating a new package

The Comunica monorepo contains a large collection of packages in the [`packages/`](https://github.com/comunica/comunica/tree/master/packages) directory. This contains [different types of packages](http://comunica.readthedocs.io/en/latest/core/) _(actors, mediators and buses)_.

For each type of package, you can use a generator tool to initialize a template repo. For this, you can use the [generator-comunica](https://github.com/comunica/generate-comunica) project (a [Yo](https://www.npmjs.com/package/yo) generator), which you can initialize as follows:

```bash
$ yarn global add yo
$ git clone git@github.com:comunica/generate-comunica.git
$ cd generate-comunica
$ yarn install
$ yarn link
```

This will expose the `comunica:bus`, `comunica:actor`, and `comunica:actor-query-operation` (subtype of `comunica:actor` for the `query-operation` bus) generators for initializing projects of the respective types.

For example, if we want to create an actor in the `rdf-parse` bus, we call the following:

```
$ yo comunica:actor
? Actor name (without actor-bus- prefix) my-parser
? Bus name (without bus- prefix) rdf-parse
? The full readable name of the actor My Parser
? The full readable name of the bus RDF Parse
? The component base name of the actor (without Bus part) MyParser
? The component base name of the bus RdfParse
? A description of the actor A comunica My Parser RDF Parse Actor.
? The component context prefix carpmp
? The component bus context prefix cbrp
```

This will initialize a package at `packages/actor-rdf-parse-my-parser/`.

In order to link the dependencies of this new package, make sure to run `yarn install` again in the monorepo root. You'll see some compilation errors, which you can ignore, as your new actor has not been implemented yet.

After that, you can start implementing your actor in `packages/actor-rdf-parse-my-parser/lib/ActorRdfParseMyParser.ts` and its tests in `packages/actor-rdf-parse-my-parser/test/ActorRdfParseMyParser-test.ts`.

Once the implementation is finished, you can —if it makes sense to do so— test it by adding it as a dependency to `actor-init-sparql`  by updating its `packages/actor-init-sparql/package.json` file. After that, you can add your actor to `packages/actor-init-sparql/config/config-default.json` as follows:

```json
...
"@context": {
  ...
  "https://linkedsoftwaredependencies.org/contexts/comunica-actor-my-parser.jsonld",
  ...
},
"@graph": [
  ...
  {
    "@id": "ex:myParser",
    "@type": "ActorMyParser"
  },
  ...
]
```

After that, you can test your package with one of the binaries of [`actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql).

More information on the actual development of your package can be found in the [module tutorial](http://comunica.readthedocs.io/en/latest/tutorials/module/).

### Sending a pull request

Once your package has been implemented, its unit tests succeed (at 100% coverage) and no linting errors occur, then you can submit [a pull request](https://github.com/comunica/comunica/pulls) for merging your code into the master branch.

When you submit a pull request, several things will happen:

* [Travis](https://travis-ci.org/) will build your branch, and test it.

* [Coveralls](https://coveralls.io/) will check the code coverage (which should be 100%) once the build finishes.

* [CLA assistant](https://cla-assistant.io/comunica/comunica) will check if you have signed the Contributor License Agreement. How to sign this will be explained by the assistant once you have sent in your pull request. 

Once all of these things succeed, a Comunica maintainer will review your code, and either merge it, or request changes.

## Implement a feature into a standalone project

If your feature should not be part of the standard set of Comunica features, or you deliberately want to keep it separated, you can develop your module outside of the Comunica monorepo.

The development of this is in line with the development of any NPM package.

As an example, we'll consider the case where an actor is being developed for querying quad patterns within an [OSTRICH (versioned) triple store](https://github.com/rdfostrich). The final example package can be found here: https://github.com/rdfostrich/comunica-actor-rdf-resolve-quad-pattern-ostrich

### Creating a new package

Considering the [internal Comunica naming strategy](http://comunica.readthedocs.io/en/latest/),  we propose the following naming strategy for external modules: Instead of prefixing package names with `@comunica/`, packages should be prefixed with `comunica-`. For example, if we want to create an `ostrich` actor in the `rdf-resolve-quad-pattern` bus, then our package name would be `comunica-actor-rdf-resolve-quad-pattern-ostrich` (instead of `@comunica/actor-rdf-resolve-quad-pattern-ostrich`).

You can use the generators as mentioned in the first part of this tutorial for generating base files. As you are working externally now, you will have to add more dev-dependencies, such as jest for testing, and typescript for compiling. An example can be found here: https://github.com/rdfostrich/comunica-actor-rdf-resolve-quad-pattern-ostrich/blob/master/package.json

### Creating a runner engine

If you want to provide an easy-to-use binary for running your new feature in Comunica SPARQL, you can extend the [Comunica SPARQL config file](https://github.com/comunica/comunica/blob/master/packages/actor-init-sparql/config/config-default.json), and wrap your own binary around it.

Assuming your config file is located at `config/config-default.json`, then your binary `bin/run.ts` could look like this:

```typescript
#!/usr/bin/env node
import {runArgsInProcess} from "@comunica/runner-cli";
const inDev: boolean = require('fs').existsSync(__dirname + '/run.ts');
runArgsInProcess(__dirname + '/../', __dirname + '/../config/config-default.json', inDev);
```

Make sure to expose this binary by including a `"bin"` entry in your `package.json` file, which could look like this:

```json
...
  "bin": {
    "comunica-sparql-ostrich": "./bin/run.js"
  },
...
```

A real-world example of this process can be found here: https://github.com/rdfostrich/comunica-actor-init-sparql-ostrich

!!! note
    As external project lose the monorepo's advantage of always being up-to-date with its (Comunica) dependencies.
    In order to automatically be up-to-date with the latest Comunica versions,
    we suggest using an automatic dependency manager such as [Greenkeper](https://greenkeeper.io/).
    Greenkeeper will automatically send pull requests to your project when Comunica updates occur that fall outside of your supported version range.


