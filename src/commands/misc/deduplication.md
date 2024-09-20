# Ensuring deduplication for copied snapshots

Different repositories usually have different parameters for splitting larger
files into smaller chunks. When copying snapshots between arbitrary repository,
deduplication between snapshots from the source and destination repository
doesn't work unless the repositories share the same parameters, the so-called
chunker parameters.

rustic enforces identical chunker parameters - if you try to copy to a
repository with different chunker parameter, you get an error like

```console
cannot copy to repository with different chunker parameter (re-chunking not implemented)!
```

Note: Currently it is not possible to change the chunker parameters of existing
repositories (re-chunking is not yet implemented).

To create a repository with identical chunker parameters to a source repository,
don't initialize the target repository, but instead run the first `copy` command
with the `--init` option. This option initializes non-existing repositories with
the correct chunker parameter:

```console
rustic copy --init [SNAPSHOTS]
```

Note: The target repositories must be defined in the config file.
