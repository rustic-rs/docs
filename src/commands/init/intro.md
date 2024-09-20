# Init - Preparing a new repository

The place where your backups will be saved is called a "repository". This
chapter explains how to create ("init") such a repository. The repository can be
stored locally, or on some remote server or service. We'll first cover using a
local repository; the remaining sections of this chapter cover all the other
options. You can skip to the next chapter once you've read the relevant section
here.

Note that rustic supports to store configuration in a config profile file in the
TOML format which is the preferred way to configure rustic.

For the password, several options exist:

- Setting the password directly in the config profile file:

```toml
[repository]
password = "secret"
```

- Setting the environment variable `RUSTIC_PASSWORD`

- Specifying the path to a file with the password via the CLI option
  `--password-file`, the environment variable `RUSTIC_PASSWORD_FILE`, or setting
  in the configuration profile

```toml
[repository]
password-file = "/path/to/my/password.txt"
```

- Configuring a program to be called when the password is needed via the CLI
  option `--password-command` , the environment variable
  `RUSTIC_PASSWORD_COMMAND` or setting in the configuration profile

```toml
[repository]
password-command = "get_my_password.sh"
```

If none of these password options is used, rustic will query for a password
(followed by a confirmation query if the password is to be set).

If you have configured your repository and password in the config profile,
simply run

```console
rustic init
```

and your repository is created.

The `init` command has an option called `--set-version` which can be used to
explicitly set the version for the new repository.

The below table shows which rustic version is required to use a certain
repository version and shows new features introduced by the repository format.

| Repository version | Minimum rustic version | Major new features  |
| :----------------: | :--------------------: | :-----------------: |
|        `1`         |      any version       |                     |
|        `2`         |        >=0.2.0         | Compression support |

Moreover, there are different options which can be set when initializing a
repository:

Options to specify the target pack size:

- `--set-treepack-size`, `--set-datapack-size` specify the default target pack
  size for tree and data pack files. Arguments can given using TODO For example,
  valid sizes are "4048kiB", "2MB", "30MiB", etc. If not specified, the default
  is 4 MiB for tree packs and 32 MiB for data packs.

- `--set-treepack-growfactor`, `--set-datapack-growfactor` specify how much the
  target pack size should be increased per square root of the total pack size in
  bytes of the given type. This equals to 32kiB per square root of the total
  pack size in GiB.

Note that larger pack sizes have advantages, especially for large repository or
remote repositories. They lead to less packs in the repository and transfer
larger datasets to the repository which can increase the throughput. But there
are also disadvantages. rustic keeps the whole pack in memory before writing it
to the backend. As writes are parallelized, multiple packs are kept. So larger
pack sizes increase the memory usage of the `backup` command. Moreover larger
pack sizes lead to increased repack rates during `prune` or `forget --prune`.
