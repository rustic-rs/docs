# Recovering from "no free space" errors

In some cases when a repository has grown large enough to fill up all disk space
or the allocated quota, then `prune` might fail to free space. `prune` works in
such a way that a repository remains usable no matter at which point the command
is interrupted. However, this also means that `prune` requires some scratch
space to work.

In most cases it is sufficient to instruct `prune` to remove all packs marked for
removal and use as little scratch space as possible.
Note that packs marked for removal are automatically removed by a prune run once 
they are old enough. If you can guarantee that the repository is not used by parallel
processes, you can also use `rustic prune --instant-delete`.

To use as little scratch space as possibe, run `rustic prune --max-repack-size 0`.
This removes all unneeded packs without repacking partly used packs.
Obviously, this can only work if several snapshots have been removed using
`forget` before. This then allows the `prune` command to actually remove data
from the repository. If the command succeeds, but there is still little free
space, then remove a few more snapshots and run `prune` again.
