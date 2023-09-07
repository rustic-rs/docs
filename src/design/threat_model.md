# Threat Model

The design goals for restic include being able to securely store backups in a
location that is not completely trusted (e.g., a shared system where others can
potentially access the files) or even modify or delete them in the case of the
system administrator.

General assumptions:

- The host system a backup is created on is trusted. This is the most basic
  requirement, and it is essential for creating trustworthy backups.
- The user uses an authentic copy of restic.
- The user does not share the repository password with an attacker.
- The restic backup program is not designed to protect against attackers
  deleting files at the storage location. There is nothing that can be done
  about this. If this needs to be guaranteed, get a secure location without any
  access from third parties.
- The whole repository is re-encrypted if a key is leaked. With the current key
  management design, it is impossible to securely revoke a leaked key without
  re-encrypting the whole repository.
- Advances in cryptography attacks against the cryptographic primitives used by
  restic (i.e., AES-256-CTR-Poly1305-AES and SHA-256) have not occurred. Such
  advances could render the confidentiality or integrity protections provided by
  restic useless.
- Sufficient advances in computing have not occurred to make brute-force attacks
  against restic's cryptographic protections feasible.

The restic backup program guarantees the following:

- Unencrypted content of stored files and metadata cannot be accessed without a
  password for the repository. Everything except the metadata included for
  informational purposes in the key files is encrypted and authenticated. The
  cache is also encrypted to prevent metadata leaks.
- Modifications to data stored in the repository (due to bad RAM, broken
  harddisk, etc.) can be detected.
- Data that has been tampered will not be decrypted.

With the aforementioned assumptions and guarantees in mind, the following are
examples of things an adversary could achieve in various circumstances.

An adversary with read access to your backup storage location could:

- Attempt a brute force password guessing attack against a copy of the
  repository (please use strong passwords with sufficient entropy).
- Infer which packs probably contain trees via file access patterns.
- Infer the size of backups by using creation timestamps of repository objects.

An adversary with network access could:

- Attempt to DoS the server storing the backup repository or the network
  connection between client and server.
- Determine from where you create your backups (i.e., the location where the
  requests originate).
- Determine where you store your backups (i.e., which provider/target system).
- Infer the size of backups by observing network traffic.

The following are examples of the implications associated with violating some of
the aforementioned assumptions.

An adversary who compromises (via malware, physical access, etc.) the host
system making backups could:

- Render the entire backup process untrustworthy (e.g., intercept password, copy
  files, manipulate data).
- Create snapshots (containing garbage data) which cover all modified files and
  wait until a trusted host has used `forget` often enough to remove all correct
  snapshots.
- Create a garbage snapshot for every existing snapshot with a slightly
  different timestamp and wait until certain `forget` configurations have been
  run, thereby removing all correct snapshots at once.

An adversary with write access to your files at the storage location could:

- Delete or manipulate your backups, thereby impairing your ability to restore
  files from the compromised storage location.
- Determine which files belong to what snapshot (e.g., based on the timestamps
  of the stored files). When only these files are deleted, the particular
  snapshot vanishes and all snapshots depending on data that has been added in
  the snapshot cannot be restored completely. Restic is not designed to detect
  this attack.

An adversary who compromises a host system with append-only (read+write allowed,
delete+overwrite denied) access to the backup repository could:

- Capture the password and decrypt backups from the past and in the future (see
  the "leaked key" example below for related information).
- Render new backups untrustworthy *after* the host has been compromised (due to
  having complete control over new backups). An attacker cannot delete or
  manipulate old backups. As such, restoring old snapshots created *before* a
  host compromise remains possible.
- Potentially manipulate the use of the `forget` command into deleting all
  legitimate snapshots, keeping only bogus snapshots added by the attacker.
  Ransomware might try this in order to leave only one option to get your data
  back: paying the ransom. For safe use of `forget`, please see the
  corresponding documentation on removing backup snapshots and append-only mode.

An adversary who has a leaked (decrypted) key for a repository could:

- Decrypt existing and future backup data. If multiple hosts backup into the
  same repository, an attacker will get access to the backup data of every host.
  Note that since the local encryption key gives access to the master key, a
  password change will not prevent this. Changing the master key can currently
  only be done using the `copy` command, which moves the data into a new
  repository with a new master key, or by making a completely new repository and
  new backup.
