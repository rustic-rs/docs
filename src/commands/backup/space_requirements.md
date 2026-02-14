# Space requirements

rustic currently assumes that your backup repository has sufficient space for
the backup operation you are about to perform. This is a realistic assumption
for many cloud providers, but may not be true when backing up to local disks.

Should you run out of space during the middle of a backup, there will be some
additional data in the repository, but the snapshot will never be created as it
would only be written at the very (successful) end of the backup operation.
Previous snapshots will still be there and will still work.

# Recovering from out of space

It can be attempted to recover from such situation by running `rustic prune` with
the following flags:

      --instant-delete
          Delete files immediately instead of marking them. This also removes all files already marked for deletion.

          # Warning

          * Only use if you are sure the repository is not accessed by parallel processes!

      --early-delete-index
          Delete index files early if instant-delete is chosen. This allows to run prune if there is few or no space left.

          # Warning

          * If prune aborts, this can lead to a (partly) missing index which must be repaired!
