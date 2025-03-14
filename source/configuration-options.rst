.. _pbm.config.options:

Configuration file options
********************************************************************************

This page describes configuration file options available in |PBM|. For how to use configuration file, see :ref:`pbm.config`.

.. _pbm.storage.config.options:

Remote backup storage options
============================================================================

|PBM| supports the following types of remote storages: 

- S3-compatible storage,
- `Microsoft Azure Blob storage <https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction>`_, and 
- filesystem. 

|PBM| should work with other S3-compatible storage but was only tested with the following ones: 

- `Amazon Simple Storage Service <https://docs.aws.amazon.com/s3/index.html>`_, 
- `Google Cloud Storage <https://cloud.google.com/storage>`_, 
- `MinIO <https://min.io/>`_.

.. _storage-type:

storage.type
^^^^^^^^^^^^^^^^^^^
   
:type: string
:required: YES

Remote backup storage type. Supported values: ``s3``, ``filesystem``, ``azure``.

.. _s3-opt:

.. rubric:: S3 type storage options

.. code-block:: yaml

   storage:
     type: s3
     s3:
       region: <string>
       bucket: <string>
       prefix: <string>
       endpointUrl: <string>
       credentials:
         access-key-id: <your-access-key-id-here>
         secret-access-key: <your-secret-key-here>
       uploadPartSize: <int>
       maxUploadParts: <int>
       storageClass: <string>
       serverSideEncryption:
         sseAlgorithm: aws:kms
         kmsKeyID: <your-kms-key-here>

.. _storage-s3-provider:

storage.s3.provider
^^^^^^^^^^^^^^^^^^^

:type: string
:required: NO

The storage provider's name. Supported values: aws, gcs
  
.. _storage-s3-bucket:

storage.s3.bucket
^^^^^^^^^^^^^^^^^^^^^^^^
  
:type: string
:required: YES

The name of the storage :term:`bucket <Bucket>`. See the `AWS Bucket naming rules <https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules>`_ and `GCS bucket naming guidelines <https://cloud.google.com/storage/docs/naming-buckets#requirements>`_ for bucket name requirements

.. _storage-s3-region:

storage.s3.region
^^^^^^^^^^^^^^^^^^^

:type: string
:required: YES (for AWS and GCS)

The location of the storage bucket. 
Use the `AWS region list <https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region>`_ and `GCS region list <https://cloud.google.com/storage/docs/locations>`_ to define the bucket region

.. _storage-s3-prefix:

storage.s3.prefix
^^^^^^^^^^^^^^^^^^^

:type: string
:required: NO

The path to the data directory on the bucket. If undefined, backups are stored in the bucket root directory

.. _storage-s3-endpointurl:

storage.s3.endpointUrl
^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:required: YES (for MinIO and GCS)

The URL to access the bucket. The default value for GCS is ``https://storage.googleapis.com``

.. _s3-cred-access-key:

storage.s3.credentials.access-key-id
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:required: YES

Your access key to the storage bucket. This option can be ommitted when you run |PBM| using an EC2 instance profile. To learn more, refer to :ref:`automate-s3-access`
   
.. _s3-cred-secret:

storage.s3.credentials.secret-access-key
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:required: YES

The key to sign your programmatic requests to the storage bucket. This option can be ommitted when you run |PBM| using an EC2 instance profile. To learn more, refer to :ref:`automate-s3-access` 

.. _s3-upload-part-size:

storage.s3.uploadPartSize
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    
:type: int
:required: NO

The size of data chunks in bytes to be uploaded to the storage bucket. Default: 10MB
    
|PBM| automatically increases the ``uploadPartSize`` value if the size of the file to be uploaded exceeds the max allowed file size. (The max allowed file size is calculated with the default values of uploadPartSize * `maxUploadParts <https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#pkg-constants>`_ and is appr. 97,6 GB).

The ``uploadPartSize`` value is printed in the :ref:`pbm-agent log <pbm-agent.log>`.

By setting this option, you can manually adjust the size of data chunks if |PBM| failed to do it for some reason. The defined ``uploadPartSize`` value overrides the default value and is used for calculating the max allowed file size

.. _s3-max-upload-parts:

storage.s3.maxUploadParts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: int
:required: NO
:default: 10,000

The maximum number of data chunks to be uploaded to the storage bucket. Default: 10,000

By setting this option, you can override the value defined in the `AWS SDK <https://docs.aws.amazon.com/sdk-for-go/api/service/s3/s3manager/#MaxUploadParts>`_.

It can be useful when using an S3 provider that supports a smaller number of chunks for multipart uploads.

The ``maxUploadParts`` value is printed in the :ref:`pbm-agent log <pbm-agent.log>`.

.. _s3-storage-class:

storage.s3.storageClass
^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:required: NO

The `storage class <https://aws.amazon.com/s3/storage-classes/>`_ assigned to objects stored in the S3 bucket. If not provided, the ``STANDARD`` storage class will be used. This option is available in |PBM| as of v1.7.0.

.. _s3-debug-log-levels:

storage.s3.debugLogLevels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:type: string
:required: NO

Enables S3 debug logging for different types of S3 requests. S3 log messages are printed in the ``pbm logs`` output.

Supported values are: ``LogDebug``, ``Signing``, ``HTTPBody``, ``RequestRetries``, ``RequestErrors``, ``EventStreamBody``. 

To specify several event types, separate them by comma. To lean more about the event types, see `the documentation <https://pkg.go.dev/github.com/aws/aws-sdk-go@v1.40.7/aws#LogLevelType>`_

When undefined, no S3 debug logging is performed. 

.. _s3-skip-TLS:

storage.s3.insecureSkipTLSVerify
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: bool
:default: False
:required: NO

Disables the TLS verification of the S3 storage. This allows |PBM| to upload data to S3-like storages that use self-issued TLS certificates. Available in |PBM| as of version 1.7.0.

.. warning::

   Use this option with caution as it might leave a hole for man-in-the-middle attacks. 

.. _server-encryption-options:

.. rubric:: Server-side encryption options

.. _sse-algorythm:

serverSideEncryption.sseAlgorythm
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   
:type: string
:required: NO

The key management mode used for server-side encryption 

Supported value: ``aws:kms``
   
.. _kms-key-id:

serverSideEncryption.kmsKeyID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
     
:type: string
:required: NO

Your customer-managed key

.. rubric:: Upload retry options

retryer.numMaxRetries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: int
:required: NO
:default: 3

The maximum number of retries to upload data to S3 storage. A zero value means no retries will be performed. Available in |PBM| as of 1.7.0.

retryer.minRetryDelay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: time.Duration
:required: NO
:default: 30

The minimum time (in ms) to wait till the next retry. Available in |PBM| as of 1.7.0.

retryer.maxRetryDelay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: time.Duration
:required: NO
:default: 5 

The maximum time (in minutes) to wait till the next retry. Available in |PBM| as of 1.7.0.

.. _filesystem:

.. rubric:: Filesystem storage options

.. code-block:: yaml

   storage:
     type: filesystem
     filesystem:
       path: <string>

.. _filesystem-path:

storage.filesystem.path
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:required: YES

The path to the backup directory

.. _azure-options:

.. rubric:: Microsoft Azure Blob storage options

.. code-block:: yaml

   storage:
     type: azure
     azure:
       account: <string>
       container: <string>
       prefix: <string>
       credentials:
         key: <your-access-key>

.. _azure-account:

storage.azure.account
^^^^^^^^^^^^^^^^^^^^^^  

:type: string      
:required: YES

The name of your storage account. 

.. _azure-container:

storage.azure.container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  

:type: string      
:required: YES

The name of the storage :term:`container <Container>`. See the  `Container names <https://docs.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names>`_ for naming conventions.

.. _azure-prefix:

storage.azure.prefix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  

:type: string      
:required: NO

The path (sub-folder) to the backups inside the container. If undefined, backups are stored in the container root directory.

.. _azure-cred-key:

storage.azure.credentials.key 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 

:type: string      
:required: YES

Your access key to authorize access to data in your storage account. 

.. _pitr.config:

Point-in-time recovery options
=================================================================================

.. code-block:: yaml

   pitr:
     enabled: <boolean> 
     oplogSpanMin: <float64>
     compression: <string>
     compressionLevel: <int>

.. _pitr-enabled:

pitr.enabled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  
:type: boolean
:default: False

Enables point-in-time recovery

.. _pitr-oplog-span:

pitr.oplogSpanMin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: float64
:default: 10

The duration of an oplog span in minutes. If set when the |pbm-agent| is making an oplog slice, the slice’s span is updated right away.

If the new duration is smaller than the previous one, the |pbm-agent| is triggered to save a new slice with the updated span. If the duration is larger, then the next slice is saved with the updated span in scheduled time.  

.. _pitr-compression:

pitr.compression
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:default: s2

The compression method for |PITR| oplog slices. Available in |PBM| as of version 1.7.0.

Supported values: ``gzip``, ``snappy``, ``lz4``, ``s2``, ``pgzip``, ``zstd``. Default: ``s2``. 

.. _pitr-compression-level:

pitr.compressionLevel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: int

The compression level is from ``0`` till ``10``. Default value depends on the compression method used. 

Note that the higher value you specify, the more time and computing resources it will take to compress / retrieve the data. 

.. _pitr-oplog-only:

pitr.oplogOnly
^^^^^^^^^^^^^^

:type: boolean
:default: False
:required: NO

Controls whether the base backup is required to start |PITR| recovery oplog slicing. When set to true, |PBM| saves oplog chunks without the base backup snapshot. Available in |PBM| starting with version 1.8.0. To learn more about the usage, see :ref:`oplog-replay`.

.. _backup-options:

Backup options
===================================================


.. code-block:: yaml

   backup:
     priority:
       "localhost:28019": 2.5
       "localhost:27018": 2.5
       "localhost:27020": 2.0
       "localhost:27017": 0.1
     compression: <string>
     compressionLevel: <int>

.. _backup-priority:

priority
^^^^^^^^^^^^

:type: array of strings

The list of ``mongod`` nodes and their priority for making backups. The node with the highest priority is elected for making a backup. If several nodes have the same priority, the one among them is randomly elected to make a backup. 

If not set, the replica set nodes have the default priority as follows: 

- hidden nodes - 2.0, 
- secondary nodes - 1.0, 
- primary node - 0.5. 

backup.compression
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: string
:default: s2

The compression method for backup snapshots. Available in |PBM| as of version 1.8.0.

When ``none`` is specified, backups are made without compression.

Supported values: ``gzip``, ``snappy``, ``lz4``, ``s2``, ``pgzip``, ``zstd``. Default: ``s2``. 

.. backup-compression-level:

backup.compressionLevel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:type: int

The compression level from ``0`` till ``10``. Default value depends on the compression method used. 

Note that the higher value you specify, the more time and computing resources it will take to compress / retrieve the data. 

.. _restore.config:

Restore options
=================================================================================

.. code-block:: yaml

   restore:
     batchSize: <int>
     numInsertionWorkers: <int>

.. _restore-batch-size:

batchSize
^^^^^^^^^^^
   
:type: int
:default: 500

The number of documents to buffer. 

.. _restore-insertion-workers:

numInsertionWorkers 
^^^^^^^^^^^^^^^^^^^^^^

:type: int
:default: 10

The number of workers that add the documents to buffer.      
       
.. include:: .res/replace.txt      