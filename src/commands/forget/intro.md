# Forget - Removing backup snapshots

All backup space is finite, so rustic allows removing old snapshots. This can be
done either manually (by specifying a snapshot ID to remove) or by using a
policy that describes which snapshots to forget. For all remove operations, two
commands need to be called in sequence: `forget` to remove snapshots, and
`prune` to remove the remaining data that was referenced only by the removed
snapshots. The latter can be automated with the `--prune` option of `forget`,
which runs `prune` automatically if any snapshots were actually removed.

It is advisable to run `rustic check` after pruning, to make sure you are
alerted, should the internal data structures of the repository be damaged. But
note: If you run a `backup` parallel to this `check` command, the output may
produce some warnings as `check` cannote distinguish whether it's a running
`backup` and OK or an aborted backup with left-overs.
