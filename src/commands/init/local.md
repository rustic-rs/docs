# Local backend

In order to create a repository at `/srv/rustic-repo`, run the following command
and enter the same password twice:

```console
$ rustic init -r /srv/rustic-repo
enter password for new repository:
created rustic repository 085b3c76b9 at /srv/rustic-repo
```

**Warning**: Remembering your password is important! If you lose it, you won't
be able to access data stored in the repository.

<!-- TODO! **Warning**: On Linux, storing the backup repository on a CIFS (SMB) share is
not recommended due to compatibility issues. Either use another backend or set
the environment variable `GODEBUG` to `asyncpreemptoff=1`. Refer to [GitHub issue
#2659](https://github.com/restic/restic/issues/2659) for further
explanations. -->
