# Recovering from "no free space" errors

In some cases when a repository has grown large enough to fill up all disk space
or the allocated quota, then `prune` might fail to free space. `prune` works in
such a way that a repository remains usable no matter at which point the command
is interrupted. However, this also means that `prune` requires some scratch
space to work.

In most cases it is sufficient to instruct `prune` to remove all packs marked
for removal and use as little scratch space as possible. Note that packs marked
for removal are automatically removed by a prune run once they are old enough.
If you can guarantee that the repository is not used by parallel processes, you
can also use `rustic prune --instant-delete`.

To use as little scratch space as possibe, run
`rustic prune --max-repack-size 0 --instant-delete` (and make sure no parallel
process acesses the repository!). This removes all unneeded packs without
repacking partly used packs. You may need to remove some snapshots using
`forget` before. This then allows the `prune` command to actually remove data
from the repository. If the command succeeds, but there is still little free
space, then remove a few more snapshots and run `prune` again.

In cases where the above commands do not work, `prune` fails because it first
writes modified index files before removing old ones. To first remove the
no-loger used index files, use
`rustic prune --max-repack-size 0 --instant-delete --early-delete-index`. Note
that if this `prune` run is aborted for some reasons, you first must rebuild
your index.
