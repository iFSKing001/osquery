osquery
=======

osquery is an operating system instrumentation toolchain for *nix based hosts. osquery makes low-level operating system analytics and monitoring both performant and intuitive.

osquery exposes an operating system as a high-performance relational database. This allows you to write SQL-based queries to explore operating system data. With osquery, SQL tables represent abstract concepts such as

- running processes
- loaded kernel modules
- open network connections

SQL tables are implemented via an easily extendable API. A bunch of tables already exist and more are constantly being written. To best understand the expressiveness that is afforded to you by osquery, consider the following SQL queries:

```sql
--------------------------------------------------------
-- get the name, pid and attached port of all processes 
-- which are listening on all interfaces
--------------------------------------------------------
SELECT DISTINCT 
  process.name, 
  listening.port, 
  process.pid
FROM processes AS process
JOIN listening_ports AS listening
ON process.pid = listening.pid
WHERE listening.address = '0.0.0.0';
```
```sql
--------------------------------------------------------
-- find every launchdaemon on an OS X host which 
--   * launches an executable when the operating 
--     system starts
--   * keeps the executable running 
-- return the name of the launchdaemon and the full 
-- path (with arguments) of the executable to be ran.
--------------------------------------------------------
SELECT 
  name, 
  program || program_arguments AS executable 
FROM launchd 
WHERE 
  (run_at_load = 'true' AND keep_alive = 'true') 
AND 
  (program != '' OR program_arguments != '');
```

These queries can be:
- performed on an ad-hoc basis to explore operating system state
- executed via a scheduler to monitor operating system state across a distributed set of hosts over time
- launched from custom applications using osquery APIs

## Building on OS X

To build osquery on OS X, all you need installed is `pip` and `brew`.
`make deps` will take care of installing the appropriate library
dependencies, but I recommend taking a look at the Makefile, just in case
you see something that might conflict with your personal setup.

Anything that doesn't have a homebrew package is built from source from
https://github.com/osquery/third-party, which is a git submodule of this
repository which is set up by `make deps`.

The complete installation/build steps are as follows:

```
git clone git@github.com:facebook/osquery.git
cd osquery
make deps
make
```

Once the project is built, try running the project's unit tests:

```
make test
```

## Developing on OS X

`make deps` will take care of installing everything you need to compile osquery
but to properly develop and contribute code, you'll need to install two additional
programs:

- clang-format: we use clang-format to format all code in osquery
- valgrind: performance is a top priority for osquery, so all code should be
	thouroughly tested with valgrind

## Table Development

### Top easy virtual tables

- [Crontab virtual table](https://github.com/facebook/osquery/issues/19)
- [Networking settings virtual table](https://github.com/facebook/osquery/issues/10)
- [Full Disk Encryption Virtual Tables](https://github.com/facebook/osquery/issues/15)

### High impact virtual tables
- [Installed browser plugins virtual table](https://github.com/facebook/osquery/issues/24)
- [System-trusted root certificated virtual table](https://github.com/facebook/osquery/issues/8)
- [Startup items virtual table](https://github.com/facebook/osquery/issues/6)


### Testing your table for memory leaks

Use valgrind to test your table for memory leaks before you commit it. The
osqueryd daemon is a very long running processes, so avoiding memory leaks is
critical. The "run" tool is useful for testing a specific query. From the root
of the osquery repository, run the following (substitute your table name in the
query):

```
valgrind --tool=memcheck --leak-check=yes --suppressions=osquery.supp ./build/tools/run --query="select * from time;"
```
