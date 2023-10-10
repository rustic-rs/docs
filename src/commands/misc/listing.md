# Listing snapshots

To list all snapshots in the repository, use the `snapshots` command:

```console
$ rustic -r /srv/rustic-repo snapshots
enter password for repository:
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir        /home/user/work
79766175  2015-05-08 21:40:19  kasimir        /home/user/work
bdbd3439  2015-05-08 21:45:17  luigi          /home/art
590c8fc8  2015-05-08 21:47:38  kazik          /srv
9f0bc19e  2015-05-08 21:46:11  luigi          /srv
````

You can filter the listing by directory path:

```console
$ rustic -r /srv/rustic-repo snapshots --path="/srv"
enter password for repository:
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
590c8fc8  2015-05-08 21:47:38  kazik          /srv
9f0bc19e  2015-05-08 21:46:11  luigi          /srv
```

Or filter by host:

```console
$ rustic -r /srv/rustic-repo snapshots --host luigi
enter password for repository:
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
bdbd3439  2015-05-08 21:45:17  luigi          /home/art
9f0bc19e  2015-05-08 21:46:11  luigi          /srv
```

Combining filters is also possible.

Furthermore you can group the output by the same filters (host, paths, tags):

```console
$ rustic -r /srv/rustic-repo snapshots --group-by host

enter password for repository:
snapshots for (host [kasimir])
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
40dc1520  2015-05-08 21:38:30  kasimir        /home/user/work
79766175  2015-05-08 21:40:19  kasimir        /home/user/work
2 snapshots
snapshots for (host [luigi])
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
bdbd3439  2015-05-08 21:45:17  luigi          /home/art
9f0bc19e  2015-05-08 21:46:11  luigi          /srv
2 snapshots
snapshots for (host [kazik])
ID        Date                 Host    Tags   Directory
----------------------------------------------------------------------
590c8fc8  2015-05-08 21:47:38  kazik          /srv
1 snapshots
```
