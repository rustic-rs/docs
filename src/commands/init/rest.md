# REST Server

In order to backup data to the remote server via HTTP or HTTPS protocol, you
must first set up a remote
[REST server](https://github.com/rustic-rs/rust-server) instance. Once the
server is configured, accessing it is achieved by changing the URL scheme like
this:

```toml
[repository]
repository = "rest:http://host:8000/"
```

Depending on your REST server setup, you can use HTTPS protocol, password
protection, multiple repositories or any combination of those features. The
TCP/IP port is also configurable. Here are some more examples:

```toml
repository = "rest:http://host:8000/"
repository = "rest:http://user:pass@host:8000/"
repository = "rest:http://user:pass@host:8000/my_backup_repo"
```

If you use TLS, rustic will use the system's CA certificates to verify the
server certificate. When the verification fails, rustic refuses to proceed and
exits with an error. If you have your own self-signed certificate, or a custom
CA certificate should be used for verification, you can pass rustic the
certificate filename via the `--cacert` option. It will then verify that the
server's certificate is contained in the file passed to this option, or signed
by a CA certificate in the file. In this case, the system CA certificates are
not considered at all.

REST server uses exactly the same directory structure as local backend, so you
should be able to access it both locally and via HTTP, even simultaneously.
