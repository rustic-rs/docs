# Comparing Snapshots

Rustic has a `diff` command which shows the difference between two snapshots or
a snapshot and a local path/dir

```console
$ rustic -r /srv/rustic-repo diff 5845b002 2ab627a6
password is correct
comparing snapshot ea657ce5 to 2ab627a6:

C   /rustic/cmd_diff.go
+   /rustic/foo
C   /rustic/rustic
```
