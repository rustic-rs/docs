# File History

Sometimes you know about a file which is present in your repository and you want
to know when this given file has been changed. Basically you want to see the
history of this file in the snapshots that have been saved in your repository.

To do so, simply use the `--path` option of `rustic find`. This will find the
given `path` in *all* snapshots. However, by default identical results are shown
as `(+x)` (like in the `rustic snapshots` output). So, this gives you exactly
the history of that file.

Typically, you also know that you can filter out some snapshots by using the
path(s) stored in the snapshots, so using a suitable `--filter-paths` will
reduce the search by excluding snapshots which cannot contain that file.

An example call:

```console
$ rustic find --path /home/user/.config/mc/ini --filter-paths /home
[00:00:01] reading index...               ████████████████████████████████████████        451/451

searching in snapshots group (host [myhost], label [], paths [/home])...
found in 03521ca4 from 2022-06-11 06:29:55
-rw-r--r--     user     user      4036 28 Mar 07:22 "/home/user/.config/mc/ini" 
found in 772fc402 from 2022-06-30 23:30:21 (+1)
-rw-r--r--     user     user      4036 13 Jun 08:19 "/home/user/.config/mc/ini" 
found in dd4d75e2 from 2022-08-31 23:30:33 (+16)
-rw-r--r--     user     user      4036 12 Aug 12:55 "/home/user/.config/mc/ini" 
found in b31f134e from 2023-10-29 23:30:20 (+230)
-rw-r--r--     user     user      4078 24 Oct 06:41 "/home/user/.config/mc/ini" 
found in bfacf88a from 2024-01-29 09:30:27 (+256)
-rw-r--r--     user     user      4078 28 Jan 23:32 "/home/user/.config/mc/ini" 
found in fef232e8 from 2024-04-23 23:30:52 (+432)
-rw-r--r--     user     user      4078 15 Apr 12:18 "/home/user/.config/mc/ini"
```

This shows that the given file exists in quite some snapshots - we exactly get
the snapshots where the file content changed. So we can easily restore the
content of the file from the snapshot we want.
