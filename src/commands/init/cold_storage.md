# Cold storage

Cold storage is a storage tier that offers a lower cost of storing your data by
increasing the cost or time to upload and/or retrieve your data. Cold storage
has operational semantics of a physical disk that needs to be loaded into a
drive before the provider can fulfill your retrieval request. It typically works
by the customer initiating a *restore* operation for objects, waiting for
restores to complete, then initiating a *get* operation for those objects.

Examples of cold storage are
[Amazon S3 Glacier storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/glacier-storage-classes.html)
and
[OVH Cloud Object Storage - Cold Archive class](https://help.ovhcloud.com/csm/en-public-cloud-storage-s3-choosing-right-storage-class?id=kb_article_view&sysparm_article=KB0047289#object-storage-cold-archive-class-s3-compatible).

rustic supports repositories across two locations: data in cold storage,
metadata in hot storage. Because different providers use different mechanisms
for requesting restoration of cold storage data, rustic has generic support for
invoking a warmup command that you supply.

Hence, at a high level, rustic supports arbitrary cold storage by supporting
*two storage buckets* and *a warmup program* that requests data from the cold
storage location before rustic downloads it.

The rest of this article describes setup and use of AWS S3 Glacier storage
classes.

### creating storage

rustic's cold storage support requires initializing a repository with both
locations. rustic doesn't support single repositories/buckets which have objects
that can be in different "states" (something like this is not contained in the
restic REST protocol; rclone therefore isn't able to provide this kind of
information).

With this split in buckets, the "hot" objects are then in fact stored in both
buckets. This shouldn't be much overhead as it is only metadata which usually is
less than 1% of the repo size. (The side effect is that the cold repo contains a
full restic-compatible repository; if you warm up it completely, you can use it
a standard repository within rustic and even restic).

At a minimum, you need to create two S3 buckets, one for hot and one for cold
storage, and an IAM identity to access these buckets from rustic.

With just this minimal infrastructure, your warmup program will *not* know when
restoration is complete. It needs to receive a list of S3 paths, call into AWS
S3 to request restoration of each, wait at least as long as the S3 restoration
SLA (N hours), then exit with success code.

To wait only until all objects are ready for retrieval, your warmup program can
subscribe to S3 notifications. This involves creating an AWS SQS queue,
subscribing it to S3 notification events, and polling the queue until all S3
objects are ready.

The last optimization concerns operations involving many packs: your warmup
program can choose to receive all needed S3 paths at once and batch-restore
them, instead of receiving them one at a time. This involves additional S3
buckets and an IAM role; refer to AWS Batch Operations documentation.

You can create all this infrastructure manually in the AWS Console, or you can
(should) use tooling like CloudFormation and CDK. The
[warmup-s3-archives CDK](https://gitlab.com/philipmw/warmup-s3-archives-cdk)
open-source project provides a model CDK application that creates all the above
described infrastructure, supporting cold storage, batch restore, and
restoration event processing. You can build and deploy this CDK into your
account as-is or adapt it for your needs.

### warmup program

Cold storage providers have unique processes for requesting cold objects; there
is no standard at this time. Hence, rustic abstracts this logic into a
user-provided warmup program.

rustic invokes the warmup program for all data packs (N at a time; one at a
time, all at once, or in between) before retrieving them from cold storage. The
warmup program's responsibility is to make these data packs ready for retrieval,
then exit with a status code letting rustic know whether it can proceeed.

The `--warm-up-command` argument takes a mix of static and variable parameters.
The supported variables are as follows:

    <warmup-command> %id        # invokes command for one pack ID at a time
    <warmup-command> %path      # invokes command for one pack path at a time
    <warmup-command> %ids       # invokes command for N pack IDs at a time
    <warmup-command> %paths     # invokes command for N pack paths at a time

Which variable to use depends on how packs are referenced in your cold storage
backend and whether both the backend and the warmup command support batching.

The most direct warmup config could be something like:

    warm-up-command  = 'aws s3api restore-object --bucket COLD-BUCKET --key %pack --restore-request \'{"Days":25,"GlacierJobParameters":{"Tier":"Standard"}}\''
    warm-up-time = '4h'

([Details on the S3 RestoreObject operation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/restoring-objects.html))

For AWS Glacier storage classes, a more comprehensive warmup program is
[warmup-s3-archives](https://gitlab.com/philipmw/warmup-s3-archives), an
open-source project that batch-requests packs for restoration and waits for
restore notifications. To support batching and restore notifications, this
warmup program expects some additional AWS infrastructure beyond just a pair of
buckets for hot/cold storage. This additional infrastructure can be created by
hand or by deploying
[warmup-s3-archives CDK](https://gitlab.com/philipmw/warmup-s3-archives-cdk).

rustic also has a configurable warmup batch size (`--warm-up-batch=N`), with the
default batch size being 1. With batch size 1, `%ids` and `%paths` is equivalent
to `%id` and `%path`: the warmup command receives one ID/path per invocation.
When batch size is >1, `%ids` and `%paths` is expanded to multiple IDs/paths per
invocation, while `%id` and `%path` invoke N instances of the warmup command in
parallel.

### configuring storage in rustic (`rustic init`)

The bucket for hot data can be specified by the `hot-repo` option or the
`RUSTIC_REPO_HOT` environment variable, e.g.:

```toml
[repository]
repository = "rclone:foo:cold-repo"
repo-hot = "rclone:foo:hot-repo"
```

or, as CLI-arguments

```console
rustic -r rclone:foo:cold-repo --repo-hot rclone:foo:hot-repo init
```

In this example in the repository `rclone:foo:cold-repo` all data is saved. In
the repository `rclone:foo:hot-repo` only hot data is saved, i.e. this is not a
complete repository.

Note that you can also specify repository options individually for hot and cold
repository, e.g.:

```toml
[repository]
repository = "opendal:s3"
repo-hot = "opendal:s3"

# options for both parts
[repository.options]
access_key_id = "xxx"
secret_access_key = "yyy"

# options only for hot part
[repository.options-hot]
bucket = "bucket_name_hot"
root = "/path/to/repo"

# options only cold part
[repository.options-cold]
bucket = "bucket_name_cold"
root = "/path/to/repo"
default_storage_class = "DEEP_ARCHIVE"
```

**Warning**: You have to specify both the cold repository and the hot repository
in the `init` command and all other commands which access and work with the
repository. Best, you use a config profile to set both within.

### backing up data (`rustic backup`)

With the above configuration, backing up data is done just like with a single
hot bucket.

### restoring data (`rustic restore`)

The restore step is where the warmup command ([see above](#warmup-program))
comes in. If the warmup command blocks until data packs are ready for retrieval,
then you do not need to specify a wait time. Else, use the
`--warm-up-wait=<duration>` parameter, setting it to the longest likely time
that the cold storage system needs to make the packs available. If the wait time
is too low, rustic will attempt to retrieve the packs before they are ready,
which will cause the restore operation to fail, so err on the side of setting
the wait time too high than too low.

Aside from the warmup parameters, there is no difference in rustic configuration
between restoring from hot and cold storage.

### things to know

The hot data location acts like a cache for all metadata, i.e.
config/key/snapshot/index files and tree packs. As all commands except `restore`
only need to access the metadata, they are fully functional but only need the
cold storage to list files while everything else is read from the "hot repo".
Note that the "hot repo" on its own is not a valid rustic repository. The "cold
repo", however, contains all files and is nothing but a standard rustic
repository.

If you additionally use a cache, you effectively have a first level cache on
your local disc and a second level cache with the "hot repo". Note that the "hot
repo" can be also a remote repo, so hot/cold repositories also work for multiple
rustic clients backing up to the same repository.

Removing cold data (`prune`) should work out of the box, but note that by
default `prune` doesn't repack any cold pack file; instead it keeps them until
the last blob within is no longer used. To do more repacking, have a look at the
`repack-cacheable` option which does allow to repack cold pack files.

Use the `keep-pack` option if you have some minimum holding duration, i.e. if
you would get charged if you remove objects too early. This option ensures that
only pack files older than the given time will be removed.

Once you get a working configuration, please share it
[in the main repository](https://github.com/rustic-rs/rustic/tree/main/config/services)
so other users can use it as well!
