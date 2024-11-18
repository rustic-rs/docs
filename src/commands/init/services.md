# Supported Services

rustic integrates `opendal`, a data access layer for many different services.

The following services are enabled in `rustic`:

| Service       | Description                   | Windows | Linux | MacOS |
| ------------- | ----------------------------- | ------- | ----- | ----- |
| `azblob`      | Azure Blob Storage            | ✅      | ✅    | ✅    |
| `azdls`       | Azure Data Lake Storage       | ✅      | ✅    | ✅    |
| `azfile`      | Azure File Storage            | ✅      | ✅    | ✅    |
| `b2`          | Backblaze B2                  | ✅      | ✅    | ✅    |
| `cos`         | Tencent Cloud Object Storage  | ✅      | ✅    | ✅    |
| `dropbox`     | Dropbox                       | ✅      | ✅    | ✅    |
| `fs`          | Local Filesystem              | ✅      | ✅    | ✅    |
| `ftp`         | FTP                           | ✅      | ✅    | ✅    |
| `gcs`         | Google Cloud Storage          | ✅      | ✅    | ✅    |
| `gdrive`      | Google Drive                  | ✅      | ✅    | ✅    |
| `ghac`        | GitHub Actions Cache          | ✅      | ✅    | ✅    |
| `http`        | HTTP                          | ✅      | ✅    | ✅    |
| `ipmfs`       | IPFS based on IPFS MFS API    | ✅      | ✅    | ✅    |
| `memory`      | In-memory storage             | ✅      | ✅    | ✅    |
| `obs`         | Huawei Cloud Object Storage   | ✅      | ✅    | ✅    |
| `onedrive`    | OneDrive                      | ✅      | ✅    | ✅    |
| `oss`         | Aliyun Object Storage Service | ✅      | ✅    | ✅    |
| `sftp`        | SFTP                          | ❌      | ✅    | ✅    |
| `swift`       | OpenStack Swift               | ✅      | ✅    | ✅    |
| `s3`          | Amazon S3                     | ✅      | ✅    | ✅    |
| `webdav`      | WebDAV                        | ✅      | ✅    | ✅    |
| `webhdfs`     | WebHDFS                       | ✅      | ✅    | ✅    |
| `yandex-disk` | Yandex Disk                   | ✅      | ✅    | ✅    |

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
