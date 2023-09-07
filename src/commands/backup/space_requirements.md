# Space requirements

Rustic currently assumes that your backup repository has sufficient space for
the backup operation you are about to perform. This is a realistic assumption
for many cloud providers, but may not be true when backing up to local disks.

Should you run out of space during the middle of a backup, there will be some
additional data in the repository, but the snapshot will never be created as it
would only be written at the very (successful) end of the backup operation.
Previous snapshots will still be there and will still work.
