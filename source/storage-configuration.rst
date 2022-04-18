.. _storage.config:

Remote backup storage 
*********************************************************************************

On this page: 

.. contents::
   :local:

Overview
========

|PBM| supports the following types of remote backup storages:

* :ref:`S3-compatible storage <s3>`
* :ref:`Filesystem type storage <filesystem-remote>`
* :ref:`Microsoft Azure Blob storage <azure>`

.. _s3:

.. rubric:: S3 compatible storage

|PBM| should work with other S3-compatible storages, but was only tested with the following ones:

- `Amazon Simple Storage Service <https://docs.aws.amazon.com/s3/index.html>`_ 
- `Google Cloud Storage <https://cloud.google.com/storage>`_
- `MinIO <https://min.io/>`_
  
As of v1.3.2, |PBM| supports :term:`server-side encryption <Server-side encryption>` for :term:`S3 buckets <Bucket>` with customer managed keys stored in |AWS KMS|.

.. seealso::

   `Protecting Data Using Server-Side Encryption with CMKs Stored in AWS Key Management Service (SSE-KMS) <https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html>`_

.. _filesystem-remote:

.. rubric:: Remote Filesystem Server Storage

This storage must be a remote file server mounted to a local directory. It is the
responsibility of the server administrators to guarantee that the same remote
directory is mounted at exactly the same local path on all servers in the
MongoDB cluster or non-sharded replica set.

.. warning::
   |PBM| uses the directory as if it were any normal directory, and does not
   attempt to confirm it is mounted from a remote server.
   If the path is accidentally a normal local directory, errors will eventually
   occur, most likely during a restore attempt. This will happen because
   |pbm-agent| processes of other nodes in the same replica set can't access
   backup archive files in a normal local directory on another server.

.. rubric:: Local Filesystem Storage

This cannot be used except if you have a single-node replica set. (See the warning
note above as to why). We recommend using any object store you might be already
familiar with for testing. If you don't have an object store yet, we recommend
using MinIO for testing as it has simple setup. If you plan to use a remote
filesytem-type backup server, please see the :ref:`filesystem-remote`
above.

.. _azure:

.. rubric:: Microsoft Azure Blob Storage

As of v1.5.0, you can use `Microsoft Azure Blob Storage <https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction>`_ as the remote backup storage for |PBM|. 

This gives users a vendor choice. Companies with Microsoft-based infrastructure can set up |PBM| with less administrative efforts.

.. note:: 
   
   Regardless of the remote backup storage you use, grant the ``List/Get/Put/Delete`` permissions to this storage for the user identified by the access credentials.

   The following example shows the permissions configuration to the ``pbm-testing`` bucket on the AWS S3 storage. 

   .. code-block:: json

      {
          "Version": "2012-10-17",
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
      
   Please refer to the documentation of your selected storage for the data access management.

   .. seealso::

      * AWS documentation: `Controlling access to a bucket with user policies <https://docs.aws.amazon.com/AmazonS3/latest/userguide/walkthrough1.html>`_
      * Google Cloud Storage documentation: `Overview of access control <https://cloud.google.com/storage/docs/access-control>`_
      * Microsoft Azure documentation: `Assign an Azure role for access to blob data <https://docs.microsoft.com/en-us/azure/storage/blobs/assign-azure-role-data-access?tabs=portal>`_
      * MinIO documentation : `Policy Management <https://docs.min.io/minio/baremetal/security/minio-identity-management/policy-based-access-control.html>`_

.. _pbm.config.example_yaml:

Example config files
================================================================================

Provide the remote backup storage configuration as a YAML config file. The following are the examples of config fles for supported remote storages. For how to insert the config file, see :ref:`pbm.config.initialize`. 

.. rubric:: S3-compatible remote storage

Amazon Simple Storage Service

.. include:: .res/code-block/yaml/example-amazon-s3-storage.yaml

GCS

.. include:: .res/code-block/yaml/example-gcs-s3-storage.yaml

MinIO

.. include:: .res/code-block/yaml/example-minio-s3-storage.yaml

.. rubric:: Remote filesystem server storage

.. include:: .res/code-block/yaml/example-local-file-system-store.yaml

.. rubric:: Microsoft Azure Blob Storage

.. include:: .res/code-block/yaml/example-azure-storage.yaml

For the description of configuration options, see :ref:`pbm.config.options`.

.. include:: .res/replace.txt     