# Forget - Removing backup snapshots

All backup space is finite, so rustic allows removing old snapshots. This can be
done either manually (by specifying a snapshot ID to remove) or by using a
policy that describes which snapshots to forget. For all remove operations, two
commands need to be called in sequence: `forget` to remove snapshots, and
`prune` to remove the remaining data that was referenced only by the removed
snapshots. The latter can be automated with the `--prune` option of `forget`,
which runs `prune` automatically if any snapshots were actually removed.

Pruning snapshots can be a time-consuming process, depending on the number of
snapshots and data to process. During a prune operation, the repository is
locked and backups cannot be completed. Please plan your pruning so that there's
time to complete it and it doesn't interfere with regular backup runs.

It is advisable to run `rustic check` after pruning, to make sure you are
alerted, should the internal data structures of the repository be damaged.
