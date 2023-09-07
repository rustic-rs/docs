# Remove a single snapshot

The command `snapshots` can be used to list all snapshots in a repository like
this:

```console
$ rustic -r /srv/rustic-repo snapshots
enter password for repository:
ID        Date                 Host      Tags  Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir         /home/user/work
79766175  2015-05-08 21:40:19  kasimir         /home/user/work
bdbd3439  2015-05-08 21:45:17  luigi           /home/art
590c8fc8  2015-05-08 21:47:38  kazik           /srv
9f0bc19e  2015-05-08 21:46:11  luigi           /srv
```

In order to remove the snapshot of `/home/art`, use the `forget` command and
specify the snapshot ID on the command line:

```console
$ rustic -r /srv/rustic-repo forget bdbd3439
enter password for repository:
removed snapshot bdbd3439
```

Afterwards this snapshot is removed:

```console
$ rustic -r /srv/rustic-repo snapshots
enter password for repository:
ID        Date                 Host     Tags  Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir        /home/user/work
79766175  2015-05-08 21:40:19  kasimir        /home/user/work
590c8fc8  2015-05-08 21:47:38  kazik          /srv
9f0bc19e  2015-05-08 21:46:11  luigi          /srv
```

But the data that was referenced by files in this snapshot is still stored in
the repository. To cleanup unreferenced data, the `prune` command must be run:

```console
$ rustic -r /srv/rustic-repo prune
enter password for repository:
repository 33002c5e opened successfully, password is correct
loading all snapshots...
loading indexes...
finding data that is still in use for 4 snapshots
[0:00] 100.00%  4 / 4 snapshots
searching used packs...
collecting packs for deletion and repacking
[0:00] 100.00%  5 / 5 packs processed

to repack:            69 blobs / 1.078 MiB
this removes:         67 blobs / 1.047 MiB
to delete:             7 blobs / 25.726 KiB
total prune:          74 blobs / 1.072 MiB
remaining:            16 blobs / 38.003 KiB
unused size after prune: 0 B (0.00% of remaining size)

repacking packs
[0:00] 100.00%  2 / 2 packs repacked
rebuilding index
[0:00] 100.00%  3 / 3 packs processed
deleting obsolete index files
[0:00] 100.00%  3 / 3 files deleted
removing 3 old packs
[0:00] 100.00%  3 / 3 files deleted
done
```

Afterwards the repository is smaller.

You can automate this two-step process by using the `--prune` switch to
`forget`:

```console
    $ rustic forget --keep-last 1 --prune
    snapshots for host mopped, directories /home/user/work:

    keep 1 snapshots:
    ID        Date                 Host        Tags        Directory
    ----------------------------------------------------------------------
    4bba301e  2017-02-21 10:49:18  mopped                  /home/user/work

    remove 1 snapshots:
    ID        Date                 Host        Tags        Directory
    ----------------------------------------------------------------------
    8c02b94b  2017-02-21 10:48:33  mopped                  /home/user/work

    1 snapshots have been removed, running prune
    loading all snapshots...
    loading indexes...
    finding data that is still in use for 1 snapshots
    [0:00] 100.00%  1 / 1 snapshots
    searching used packs...
    collecting packs for deletion and repacking
    [0:00] 100.00%  5 / 5 packs processed

    to repack:           69 blobs / 1.078 MiB
    this removes         67 blobs / 1.047 MiB
    to delete:            7 blobs / 25.726 KiB
    total prune:         74 blobs / 1.072 MiB
    remaining:           16 blobs / 38.003 KiB
    unused size after prune: 0 B (0.00% of remaining size)

    repacking packs
    [0:00] 100.00%  2 / 2 packs repacked
    rebuilding index
    [0:00] 100.00%  3 / 3 packs processed
    deleting obsolete index files
    [0:00] 100.00%  3 / 3 files deleted
    removing 3 old packs
    [0:00] 100.00%  3 / 3 files deleted
    done
```
