# Other Services via rclone

The program `rclone` *can be used to access many other different services and
store data there. First, you need to install and `configure`* rclone. The
general backend specification format is `rclone:<remote>:<path>`, the
`<remote>:<path>` component will be directly passed to rclone. When you
configure a remote named `foo`, you can then call rustic as follows to initiate
a new repository in the path `bar` in the repo:

```console
rustic -r rclone:foo:bar init
```

rustic takes care of starting and stopping rclone.

As a more concrete example, suppose you have configured a remote named `b2prod`
for Backblaze B2 with rclone, with a bucket called `yggdrasil`. You can then use
rclone to list files in the bucket like this:

```console
rclone ls b2prod:yggdrasil
```

In order to create a new repository in the root directory of the bucket, call
rustic like this:

```console
rustic -r rclone:b2prod:yggdrasil init
```

If you want to use the path `foo/bar/baz` in the bucket instead, pass this to
rustic:

```console
rustic -r rclone:b2prod:yggdrasil/foo/bar/baz init
```

Listing the files of an empty repository directly with rclone should return a
listing similar to the following:

```console
$ rclone ls b2prod:yggdrasil/foo/bar/baz
    155 bar/baz/config
    448 bar/baz/keys/4bf9c78049de689d73a56ed0546f83b8416795295cda12ec7fb9465af3900b44
```

Rclone can be configured with environment variables prefixed by `RCLONE_`, so
for instance configuring a bandwidth limit for rclone can be achieved by setting
the `RCLONE_BWLIMIT` environment variable:

```console
export RCLONE_BWLIMIT=1M
```

For debugging rclone, you can set the environment variable `RCLONE_VERBOSE=2`.
