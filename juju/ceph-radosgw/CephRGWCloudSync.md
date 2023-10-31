# Ceph RGW Cloud Sync

In order to configure Ceph RGW with cloud sync module, we need to deploy
a [secondary Ceph RGW](https://ubuntu.com/ceph/docs/setting-up-multi-site) as usual, with the only exceptions being:

1. We will use the same Ceph cluster for the primary and secondary Ceph RGW
   Juju applications.

2. The secondary Ceph RGW, configured with cloud sync, needs to be related using
   the `cloud-sync` relation with the primary Ceph RGW:

    ```bash
    juju relate ceph-radosgw:primary ceph-radosgw-cloud-sync:cloud-sync
    ```

## Cloud Sync S3 Targets Configuration

The `ceph-radowsgw` application with cloud sync will block unless we configure
the S3 targets.

These are configured by relating [s3-integrator](https://github.com/canonical/s3-integrator)
Juju applications.

It is important to note that the each deployed instance of the `s3-integrator`
charm maps to a single S3 target. The `s3-integrator` applications will relate
to the `ceph-radosgw` application via `s3-credentials` relation:

```bash
juju relate ceph-radosgw-cloud-sync:s3-credentials s3-target-a:s3-credentials
juju relate ceph-radosgw-cloud-sync:s3-credentials s3-target-b:s3-credentials
juju relate ceph-radosgw-cloud-sync:s3-credentials s3-target-c:s3-credentials
```

The cloud sync `ceph-radosgw` identifies S3 targets via the related
`s3-integrator` Juju application name.

Let's focus on the application configs of the `s3-integrator` charm. This is
the way we configure each S3 target for cloud sync.

Let's take this example of using Minio as an S3 target:

```yaml
minio-site-a:
  charm: s3-integrator
  num_units: 1
  options:
    endpoint: http://10.13.1.1:9010
    region: eu-central-1
    bucket: staging,documents,test*
    s3-uri-style: path
```

These are all the relevant config options that are used in the `s3-credentials`
relation with the cloud sync `ceph-radosgw`:

* `endpoint`: This is the endpoint that the cloud sync `ceph-radosgw` will use
  to connect to the S3 target.

* `region`: This is the region that the cloud sync `ceph-radosgw` will use to
  connect to the S3 target.

* `bucket`: Comma-separate list of buckets (or buckets prefixes) that the
  `ceph-radosgw` with cloud sync will use to sync data to the S3 target.
  The bucket prefix must be given only as `PREFIX*` (string ending with `*`).

  For example, `test*` will match all the buckets that start with `test`, and
  sync them to this S3 target.

* `s3-uri-style`: This is the URI style that the cloud sync `ceph-radosgw`
  will use to connect to the S3 target. Allows values are: `path` or `virtual`.
  If this is omitted, Ceph cloud sync with use `path` by default.

  This represents the way the Ceph cloud sync makes API requests to the S3
  target. More details about this topic [here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAPI.html).

  It depends on how the S3 target is configured. By default, MiniO allows `path`
  API requests, but it can be configured with virtual-host-style requests if a
  DNS domain is configured for it (outside the scope of this document).

Each S3 target needs connection credentials as well (access key and secret
key). These are passed to the `s3-integrator` application using the following
Juju action:

```bash
juju run minio-site-a/leader sync-s3-credentials --string-args access-key=ACCESS_KEY secret-key=ACCESS_SECRET
```

It is important to know that Ceph cloud sync allows syncing to multiple S3
targets (via bucket name or bucket prefix). However, it is mandatory to
configure a default S3 target. This is needed for all the buckets that are not
explicitly set to a specific S3 target.

The default S3 target is configured via `cloud-sync-default-s3-target` in the
`ceph-radosgw` cloud sync application. This is a string that represents
the name of the related s3-integrator application which is going to be the
default cloud sync S3 target.

For example, this is how we configure the default S3 target to be `s3-target-a`:

```bash
juju config ceph-radosgw-cloud-sync cloud-sync-default-s3-target=s3-target-a
juju relate ceph-radosgw-cloud-sync:s3-credentials s3-target-a:s3-credentials
```

As a summary, the cloud sync `ceph-radosgw` won't configure any S3 targets,
unless the related `s3-integrator` applications are configured with:

1. `endpoint`, `region` - both are mandatory, and are passed via Juju configs.

2. `bucket` - mandatory, and is passed via Juju config. The only situation
   when this is optional (and it's ignored) is when the S3 target is configured
   as the default S3 target.

3. `access-key`, `secret-key` - mandatory, and are passed via the
   `sync-s3-credentials` Juju action.

4. `s3-uri-style`- optional, and is passed via Juju config. If this is omitted,
   Ceph cloud sync with use `path` by default.
