# Remote backup storage

Percona Backup for MongoDB supports the following types of remote backup storage:


* [S3-compatible storage](#s3-compatible-storage)

* [Filesystem type storage](#remote-filesystem-server-storage)

* [Microsoft Azure Blob storage](#microsoft-azure-blob-storage)

## S3-compatible storage

Percona Backup for MongoDB should work with other S3-compatible storages, but was only tested with the following ones:


* [Amazon Simple Storage Service](https://docs.aws.amazon.com/s3/index.html)


* [Google Cloud Storage](https://cloud.google.com/storage)


* [MinIO](https://min.io/)

As of version 1.3.2, Percona Backup for MongoDB supports [server-side encryption](https://docs.percona.com/percona-backup-mongodb/glossary.html#term-Server-side-encryption) for [S3 buckets](../reference/glossary.md#bucket) with customer managed keys stored in AWS KMS.

!!! admonition "See also"

    [Protecting Data Using Server-Side Encryption with CMKs Stored in AWS Key Management Service (SSE-KMS)](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)

### Debug logging

!!! admonition "Version added: 1.7.0" 

You can enable debug logging for different types of S3 requests in Percona Backup for MongoDB. Percona Backup for MongoDB prints S3 log messages in the `pbm logs` output so that you can debug and diagnose S3 request issues or failures.

To enable S3 debug logging, set the `storage.s3.DebugLogLevel` option in Percona Backup for MongoDB configuration. The supported values are: `LogDebug`, `Signing`, `HTTPBody`, `RequestRetries`, `RequestErrors`, `EventStreamBody`.

### Storage classes 

!!! admonition "Version added: 1.7.0" 

Percona Backup for MongoDB supports [Amazon S3 storage classes](https://aws.amazon.com/s3/storage-classes/). Knowing your data access patterns, you can set the S3 storage class in Percona Backup for MongoDB configuration. When Percona Backup for MongoDB uploads data to S3, the data is distributed to the corresponding storage class. The support of S3 bucket storage types allows you to effectively manage S3 storage space and costs.

To set the storage class, specify the `storage.s3.storageClass` option in Percona Backup for MongoDB configuration file

```yaml
storage:
  type: s3
  s3:
    storageClass: INTELLIGENT_TIERING
```

When the option is undefined, the S3 Standard storage type is used.

### Configure upload retries 

!!! admonition "Version added: 1.7.0" 

You can set up the number of attempts for Percona Backup for MongoDB to upload data to S3 storage as well as the min and max time to wait for the next retry. Set the options `storage.s3.retryer.numMaxRetries`, `storage.s3.retryer.minRetryDelay` and `storage.s3.retryer.maxRetryDelay` in Percona Backup for MongoDB configuration.

```yaml
retryer:
  numMaxRetries: 3
  minRetryDelay: 30
  maxRetryDelay: 5
```

This upload retry increases the chances of data upload completion in cases of unstable connection.

### Data upload for storage with self-issued TLS certificates

!!! admonition "Version added: 1.7.0"

Percona Backup for MongoDB supports data upload to S3-like storage that supports self-issued TLS certificates. To make this happen, disable the TLS verification of the S3 storage in Percona Backup for MongoDB configuration:

```sh
pbm config --set storage.s3.insecureSkipTLSVerify=True
```

!!! warning 

    Use this option with caution as it might leave a hole for man-in-the-middle attacks.

## Remote Filesystem Server Storage

This storage must be a remote file server mounted to a local directory. It is the responsibility of the server administrators to guarantee that the same remote directory is mounted at exactly the same local path on all servers in the
MongoDB cluster or non-sharded replica set.

!!! warning

    Percona Backup for MongoDB uses the directory as if it were any normal directory, and does not attempt to confirm it is mounted from a remote server.

    If the path is accidentally a normal local directory, errors will eventually
    occur, most likely during a restore attempt. This will happen because **pbm-agent** processes of other nodes in the same replica set can’t access backup archive files in a normal local directory on another server.

## Local Filesystem Storage

This cannot be used except if you have a single-node replica set. (See the warning note above as to why). We recommend using any object store you might be already familiar with for testing. If you don’t have an object store yet, we recommend using MinIO for testing as it has simple setup. If you plan to use a remote filesytem-type backup server, please see the [Remote Filesystem Server Storage](#remote-filesystem-server-storage) above.

## Microsoft Azure Blob Storage

!!! admonition "Version added: 1.5.0"

You can use [Microsoft Azure Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) as the remote backup storage for Percona Backup for MongoDB.

This gives users a vendor choice. Companies with Microsoft-based infrastructure can set up Percona Backup for MongoDB with less administrative efforts.

## Permissions setup

Regardless of the remote backup storage you use, grant the `List/Get/Put/Delete` permissions to this storage for the user identified by the access credentials.

The following example shows the permissions configuration to the `pbm-testing` bucket on the AWS S3 storage.

```json
{
    "Version": "2021-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::pbm-testing"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::pbm-testing/*"
        }
    ]
}
```

Please refer to the documentation of your selected storage for the data access management.

!!! admonition "See also"

    * AWS documentation: [Controlling access to a bucket with user policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html)
    * Google Cloud Storage documentation: [Overview of access control](https://cloud.google.com/storage/docs/access-control)
    * Microsoft Azure documentation: [Assign an Azure role for access to blob data](https://docs.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access?tabs=portal)
    * MinIO documentation: [Policy Management](https://docs.min.io/minio/baremetal/security/minio-identity-management/policy-based-access-control.html)

*[AWS KMS]: Amazon Web Services Key Management Service