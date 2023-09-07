# Ensuring deduplication for copied snapshots

Even though the copy command can transfer snapshots between arbitrary
repositories, deduplication between snapshots from the source and destination
repository may not work. To ensure proper deduplication, both repositories have
to use the same parameters for splitting large files into smaller chunks, which
requires additional setup steps. With the same parameters rustic will for both
repositories split identical files into identical chunks and therefore
deduplication also works for snapshots copied between these repositories.

The chunker parameters are generated once when creating a new (destination)
repository. That is for a copy destination repository we have to instruct rustic
to initialize it using the same chunker parameters as the source repository:

```console
rustic -r /srv/rustic-repo-copy init --repo2 /srv/rustic-repo --copy-chunker-params
```

Note that it is not possible to change the chunker parameters of an existing
repository.
