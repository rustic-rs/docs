# Reading data from StdIn

Sometimes it can be nice to directly save the output of a program, e.g.
`mysqldump` so that the SQL can later be restored. rustic supports this mode of
operation, just supply `-` as backup source to the `backup` command like this:

```console
set -o pipefail
mysqldump [...] | rustic backup -
```

This creates a new snapshot of the output of `mysqldump`. You can then use e.g.
the fuse mounting option (see below) to mount the repository and read the file.

By default, the file name `stdin` is used, a different name can be specified
with `--stdin-filename`, e.g. like this:

```console
mysqldump [...] | rustic --stdin-filename production.sql -
```

The option `pipefail` is highly recommended so that a non-zero exit code from
one of the programs in the pipe (e.g. `mysqldump` here) makes the whole chain
return a non-zero exit code. Refer to the
`Use the Unofficial Bash Strict Mode <http://redsymbol.net/articles/unofficial-bash-strict-mode/>`__
for more details on this.
