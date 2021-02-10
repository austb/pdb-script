# PuppetDB Dev Helper Script

## Prerequisites

* PostgreSQL 9.6+
* Put the [`pgbox`](https://gitlab.com/pgbox-org/pgbox) script on your `$PATH`

## Setting up the script

The script contains a section of static full path configs that it needs to
function. You should set these for your local dev environment, and unless you
move anything around on your system they shouldn't need to be changed.

Open the script in an editor and find the section marked static config. Set each
setting for your system.

#### `pdb_dir`

The full path to your checkout of the open source puppetdb repository

```
pdb_dir=~/src/puppetdb
```

#### `pe_pdb_dir`

The full path to your checkout of the pe-puppetdb-extensions repository

```
pe_pdb_dir=~/wrk/pe-puppetdb-extensions
```
#### `pg_bin`

The full path to the postgres bin

```
# On a Mac it should look something like
pg_bin=/Applications/Postgres.app/Contents/Versions/11/bin/
```
```
# On Linux it might look like
pg_bin=/usr/pgsql-11/bin
```

#### `pg_sandbox`

The full path to a directory where all the pdbboxes made by this script will
live on your machine.

```
pg_sandbox=~/sandbox/pdb
```

With that path the default pdbbox will be created in a directory at
`~/sandbox/pdb/tmp_pg`


## Usage

See `pdb --help` for the full set of commands

Most commands need you to be in the directory that is either your PuppetDB or
pe-puppetdb-extensions repo.  Some commands (such as `run` and `benchmark`) will
make different choices based on the repository that you are in.

### Run PuppetDB

Assuming you have both repos checked out and configured in the script's static
configs, this should work to setup and run either puppetdb or
pe-puppetdb-extension

```
pdb init
pdb run
```

After restarting your machine, the postgres instance inside your pdbbox will not
be running, it can be started with `pdb start` (and stopped if you want with
`pdb stop`).

### Creating another pdbbox

If you want to create a non-default pdbbox (for testing a different version of
puppetdb, or to create a pdbbox that won't be deleted every time you run `pdb
init`) you can provide the `--name` argument to specify it.

```
pdb --name non-default init
```

This will create a pdbbox at `$pg_sandbox/non-default`, you'll need to provide
the name argument to every command afterwards so it can find the correct pdbbox
settings.

```
pdb --name non-default run
```

Or you can run test with

```
pdb --name non-default test
```


### Running tests

Open source puppetdb has three sets of tests, clojure unit tests, clojure
integration tests (with puppetserver and puppet), and external tests. This
script can run all three for you.

#### Clojure unit tests

This command works similarly in both the open source and PE repos.

```
# clojure unit tests
pdb test
```

If a clojure test fails, the pdb script support the same syntax for re-running
that test as `lein` does. The test output will print a line like this when a
test fails.

```
lein test :only puppetlabs.puppetdb.scf.migrate-partitioning-test/migration-79-schema-diff
```

You can re-run just that test by using your pdbbox by swapping the `lein` out
for `pdb` and running

```
pdb test :only puppetlabs.puppetdb.scf.migrate-partitioning-test/migration-79-schema-diff
```

#### Open source integration tests

The integration tests checkout both puppet and puppetserver from source. If you
switch major versions of puppetdb (`git checkout 6.x` from `main` or vice
versa), you'll have the wrong versions checked out locally, run `pdb clean` (or
`lein distclean`) to reset your local dev testing environment.

```
# clojure integration tests
pdb integration
```

#### External tests

These tests the command line interface of the puppetdb jar.

```
# ext tests
pdb ext
```

### Running HA sync

From the pe-puppetdb-extension repo

```
pdb init-sync
```

This will create two "special" pdbboxes named `sync1` and `sync2`. Then simply
run them from two terminals, `pdb -n sync1 run`, `pdb -n sync2 run`.

### Populating the database

This command functions differently based on whether your current directory is
puppetdb or pe-puppetdb-extensions.

```
pdb benchmark
```

By default the command will at 10 nodes worth of data by submitting 30 commands.
This includes one factset, catalog, and report for each node.

If you'd like more or fewer nodes that can be provided as a suplemental
argument. The following command will submit 1 node (3 commands).

```
pdb benchmark -- 1
```

You can submit any amount of nodes, just note that when submitting hundreds or
more commands, the benchmark command will complete submission of the commands
while there is still a large PuppetDB command queue to be worked through before
the data is actually present in the database and query-able. And obviously
submitting 1,000 nodes worth of data will take a bit of time compared to just
10.

```
pdb benchmark -- 1000
```
