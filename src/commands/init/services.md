# Supported Services

rustic integrates `opendal`, a data access layer for many different services.
The following services are supported:

| Service Name                  | Postfix    |
| ----------------------------- | ---------- |
| Backblaze B2                  | `b2`       |
| SFTP                          | `sftp`     |
| OpenStack Swift               | `swift`    |
| Azure Blob Storage            | `azblob`   |
| Azure Data Lake Storage Gen2  | `azdls`    |
| Azure File Storage            | `azfile`   |
| Tencent Cloud Object Storage  | `cos`      |
| Local Filesystem              | `fs`       |
| FTP                           | `ftp`      |
| Dropbox                       | `dropbox`  |
| Google Drive                  | `gdrive`   |
| Google Cloud Storage          | `gcs`      |
| GitHub Actions Cache          | `ghac`     |
| HTTP                          | `http`     |
| IPFS based on IPFS MFS API    | `ipmfs`    |
| In-memory storage             | `memory`   |
| Huawei Cloud OBS              | `obs`      |
| Microsoft OneDrive            | `onedrive` |
| Aliyun Object Storage Service | `oss`      |
| Amazon S3                     | `s3`       |
| WebDAV                        | `webdav`   |
| WebHDFS                       | `webhdfs`  |

## Configuration

The configuration for the services is done via the `rustic` configuration file.

For example, to configure the `Amazon S3` service, you would add the following
to the configuration file:

```toml
[repository]
repository = "opendal:s3"
password = "password"

# Other options can be given here - note that opendal also support reading config from env
# files or AWS config dirs, see the opendal S3 docs for more information
# https://opendal.apache.org/docs/rust/opendal/services/struct.S3.html
[repository.options]
access_key_id = "xxx" # this can be ommited, when AWS config is used
secret_access_key = "xxx" # this can be ommited, when AWS config is used
bucket = "bucket_name"
root = "/path/to/repo"
```

`opendal` is used to access the services. The `s3` postfix is used to specify
the `Amazon S3` service. For other services, the postfix is replaced with the
respective service postfix.

You can find more service templates in the
[rustic repository](https://github.com/rustic-rs/rustic/tree/main/config).

To see the service-dependent options to be set in `[repository.options]`, please
refer to the
[opendal service documentation](https://opendal.apache.org/docs/rust/opendal/services/).
