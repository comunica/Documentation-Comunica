# Executing a SPARQL query

SPARQL queries can easily be executed using the
[`actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql)
module.

The easiest way to get started is by installing the module as follows:

```bash
$ [sudo] npm install -g @comunica/actor-init-sparql
```

After that, SPARQL queries can be executed as follows:

```bash
$ comunica-sparql http://fragments.dbpedia.org/2015-10/en "CONSTRUCT WHERE { ?s ?p ?o } LIMIT 100"
```

Further documentation for that can be found in the corresponding
[README](https://github.com/comunica/comunica/blob/master/packages/actor-init-sparql/README.md).

## Data Flow
Here we will describe how the packages interact when solving a SPARQL query,
based on the default configuration of
[`actor-init-sparql`](https://github.com/comunica/comunica/tree/master/packages/actor-init-sparql).
