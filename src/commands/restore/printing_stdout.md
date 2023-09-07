# Printing files to stdout

Sometimes it's helpful to print files to stdout so that other programs can read
the data directly. This can be achieved by using the `dump` command, like this:

```console
rustic -r /srv/rustic-repo dump latest production.sql | mysql
```

If you have saved multiple different things into the same repo, the `latest`
snapshot may not be the right one. For example, consider the following snapshots
in a repo:

```console
$ rustic -r /srv/rustic-repo snapshots
ID        Date                 Host        Tags        Directory
----------------------------------------------------------------------
562bfc5e  2018-07-14 20:18:01  mopped                  /home/user/file1
bbacb625  2018-07-14 20:18:07  mopped                  /home/other/work
e922c858  2018-07-14 20:18:10  mopped                  /home/other/work
098db9d5  2018-07-14 20:18:13  mopped                  /production.sql
b62f46ec  2018-07-14 20:18:16  mopped                  /home/user/file1
1541acae  2018-07-14 20:18:18  mopped                  /home/other/work
----------------------------------------------------------------------
```

Here, rustic would resolve `latest` to the snapshot `1541acae`, which does not
contain the file we'd like to print at all (`production.sql`). In this case, you
can pass rustic the snapshot ID of the snapshot you like to restore:

```console
rustic -r /srv/rustic-repo dump 098db9d5 production.sql | mysql
```

Or you can pass rustic a path that should be used for selecting the latest
snapshot. The path must match the patch printed in the "Directory" column, e.g.:

```console
rustic -r /srv/rustic-repo dump --path /production.sql latest production.sql | mysql
```

It is also possible to `dump` the contents of a whole folder structure to
stdout. To retain the information about the files and folders rustic will output
the contents in the tar (default) or zip format:

```console
rustic -r /srv/rustic-repo dump latest /home/other/work > restore.tar
```

```console
rustic -r /srv/rustic-repo dump -a zip latest /home/other/work > restore.zip
```
