# PuppetDB Dev Helper Script

## Prerequisites

* PostgreSQL 9.6+
* Put the [`pgbox`](https://gitlab.com/pgbox-org/pgbox) script on your `$PATH`

## Setting up the script

The script contains a section of static full path configs that it needs to
function. You should set these for your local dev environment, and unless you
move anything around on your system they shouldn't need to be changed.

Open the script in an editor and find the section marked static config. Set
each setting for your system.

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

The full path to a directory where all the pdbboxes made by this script will live
on your machine.

```
pg_sandbox=~/sandbox/pdb
```

With that path the default pdbbox will be created in a directory at `~/sandbox/pdb/tmp_pg`


## Usage

See `pdb --help` for the full set of commands

Most commands need you to be in the directory that is either your PuppetDB or pe-puppetdb-extensions repo.
Some commands (such as `run` and `benchmark`) will make different choices based on the repository that you are in.

### Run PuppetDB

Assuming you have both repos checked out and configured in the script's static configs,
this should work to setup and run either puppetdb or pe-puppetdb-extension

```
pdb init
pdb run
```

### Creating another pdbbox

If you want to create a non-default pdbbox (for testing a different version of puppetdb, or
to create a pdbbox that won't be deleted every time you run `pdb init`) you can provide the 
`--name` argument to specify it.

```
pdb --name non-default init
```

This will create a pdbbox at `$pg_sandbox/non-default`, you'll need to provide the name argument
to every command afterwards so it can find the correct pdbbox settings.

```
pdb --name non-default run
```

Or you can run test with

```
pdb --name non-default test
```


### Running tests

Open source puppetdb has three sets of tests, clojure unit tests, clojure
integration tests (with puppetserver and puppet), and external tests. This script
can run all three for you.

```
# clojure unit tests
pdb test
```

```
# clojure integration tests
pdb integration
```

```
# ext tests
pdb ext
```

If you want to run pe-puppetdb-extension clojure unit tests, just navigate to your git checkout and run

```
pdb test
```

### Running HA sync

From the pe-puppetdb-extension repo

```
pdb init-sync
```

This will create two "special" pdbboxes named `sync1` and `sync2`. Then simply run them from two terminals,
`pdb -n sync1 run`, `pdb -n sync2 run`.
