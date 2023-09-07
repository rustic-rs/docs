# Recovering from "no free space" errors

In some cases when a repository has grown large enough to fill up all disk space
or the allocated quota, then `prune` might fail to free space. `prune` works in
such a way that a repository remains usable no matter at which point the command
is interrupted. However, this also means that `prune` requires some scratch
space to work.

In most cases it is sufficient to instruct `prune` to use as little scratch
space as possible by running it as `prune --max-repack-size 0`. Note that for
rustic versions before 0.13.0 `prune --max-repack-size 1` must be used.
Obviously, this can only work if several snapshots have been removed using
`forget` before. This then allows the `prune` command to actually remove data
from the repository. If the command succeeds, but there is still little free
space, then remove a few more snapshots and run `prune` again.

If `prune` fails to complete, then
`prune --unsafe-recover-no-free-space SOME-ID` is available as a method of last
resort. It allows prune to work with little to no free space. However, a
**failed** `prune` run can cause the repository to become **temporarily
unusable**. Therefore, make sure that you have a stable connection to the
repository storage, before running this command. In case the command fails, it
may become necessary to manually remove all files from the `index/` folder of
the repository and run `rebuild-index` afterwards.

To prevent accidental usages of the `--unsafe-recover-no-free-space` option it
is necessary to first run `prune --unsafe-recover-no-free-space SOME-ID` and
then replace `SOME-ID` with the requested ID.
