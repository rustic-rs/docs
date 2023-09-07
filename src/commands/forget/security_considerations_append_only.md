# Security considerations in append-only mode

**Note**: TL;DR: With append-only repositories, one should specifically use the
`--keep-within` option of the `forget` command when removing snapshots.

To prevent a compromised backup client from deleting its backups (for example
due to a ransomware infection), a repository service/backend can serve the
repository in a so-called append-only mode. This means that the repository is
served in such a way that it can only be written to and read from, while delete
and overwrite operations are denied. rustic's `rest-server` *features an
append-only mode, but few other standard backends do. To support append-only
with such backends, one can use `rclone`* as a complement in between the backup
client and the backend service.

- [rest-server](https://github.com/rustic/rest-server/)
- [rclone](https://rclone.org/commands/rclone_serve_rustic/)

To remove snapshots and recover the corresponding disk space, the `forget` and
`prune` commands require full read, write and delete access to the repository.
If an attacker has this, the protection offered by append-only mode is naturally
void. The usual and recommended setup with append-only repositories is therefore
to use a separate and well-secured client whenever full access to the repository
is needed, e.g. for administrative tasks such as running `forget`, `prune` and
other maintenance commands.

However, even with append-only mode active and a separate, well-secured client
used for administrative tasks, an attacker who is able to add garbage snapshots
to the repository could bring the snapshot list into a state where all the
legitimate snapshots risk being deleted by an unsuspecting administrator that
runs the `forget` command with certain `--keep-*` options, leaving only the
attacker's useless snapshots.

For example, if the `forget` policy is to keep three weekly snapshots, and the
attacker adds an empty snapshot for each of the last three weeks, all with a
timestamp (see the `backup` command's `--time` option) slightly more recent than
the existing snapshots (but still within the target week), then the next time
the repository administrator (or a scheduled job) runs the `forget` command with
this policy, the legitimate snapshots will be removed (since the policy will
keep only the most recent snapshot within each week). Even without running
`prune`, recovering data would be messy and some metadata lost.

To avoid this, `forget` policies applied to append-only repositories should use
the `--keep-within` option, as this will keep not only the attacker's snapshots
but also the legitimate ones. Assuming the system time is correctly set when
`forget` runs, this will allow the administrator to notice problems with the
backup or the compromised host (e.g. by seeing more snapshots than usual or
snapshots with suspicious timestamps). This is, of course, limited to the
specified duration: if `forget --keep-within 7d` is run 8 days after the last
good snapshot, then the attacker can still use that opportunity to remove all
legitimate snapshots.
