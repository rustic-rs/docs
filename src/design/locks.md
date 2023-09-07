# Locks

The restic repository structure is designed in a way that allows parallel access
of multiple instance of restic and even parallel writes. However, there are some
functions that work more efficient or even require exclusive access of the
repository. In order to implement these functions, restic processes are required
to create a lock on the repository before doing anything.

Locks come in two types: Exclusive and non-exclusive locks. At most one process
can have an exclusive lock on the repository, and during that time there must
not be any other locks (exclusive and non-exclusive). There may be multiple
non-exclusive locks in parallel.

A lock is a file in the subdir `locks` whose filename is the storage ID of the
contents. It is stored in the file encoding described in the "Unpacked Data
Format" section and contains the following JSON structure:

```json
{
    "time": "2015-06-27T12:18:51.759239612+02:00",
    "exclusive": false,
    "hostname": "kasimir",
    "username": "fd0",
    "pid": 13607,
    "uid": 1000,
    "gid": 100
}
```

The field `exclusive` defines the type of lock. When a new lock is to be
created, restic checks all locks in the repository. When a lock is found, it is
tested if the lock is stale, which is the case for locks with timestamps older
than 30 minutes. If the lock was created on the same machine, even for younger
locks it is tested whether the process is still alive by sending a signal to it.
If that fails, restic assumes that the process is dead and considers the lock to
be stale.

When a new lock is to be created and no other conflicting locks are detected,
restic creates a new lock, waits, and checks if other locks appeared in the
repository. Depending on the type of the other locks and the lock to be created,
restic either continues or fails.
