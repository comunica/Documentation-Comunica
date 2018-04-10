# Init Actors
Init actors are actors made to be an "entrypoint" into a Comunica system,
usually by making use of command line arguments.

The default interface and bus can be found in [`@comunica/bus-init`](https://github.com/comunica/comunica/tree/master/packages/bus-init).
Their input action contains everything required to accept all the user input:
an `argv` string array,
an `env` object mapping the environment and
a `stdin` input stream.
[`@comunica/runner-cli`](https://github.com/comunica/comunica/tree/master/packages/runner-cli)
can be used to start up a Comunica process while parsing the command line arguments and sending them to the init actor.
