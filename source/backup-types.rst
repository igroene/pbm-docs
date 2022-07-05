.. _backup-types:

************************
Backup and restore types
************************

As of version 1.7.0, |PBM| supports physical and logical backups and restores. This document describes each type.

.. important::

   Physical backups and restores is the technical preview feature [1]_. Before using them in production, we recommend that you test restoring from physical backups in your environment, and also use an alternative backup method for redundancy.

`Physical` backup is copying of physical files from the |PSMDB| ``dbPath`` data directory to the remote backup storage. These files include data files, journal, index files, etc. Physical restore is the reverse process: ``pbm-agents`` shut down the ``mongod`` nodes, clean up the ``dbPath`` data directory and copy the physical files from the storage to it. 

During physical backups and restores, ``pbm-agents`` don’t connect to the database and don’t read the data. This significantly reduces the backup / restore time compared to logical ones and is the recommended backup method for big (multi-terabyte) databases.

Physical backups and restores are available for |PSMDB| starting from versions 4.2.15-16, 4.4.6-8, 5.0 and higher. Since physical backups heavily rely on the WiredTiger $backupCursor functionality, they are available only for WiredTiger storage engine.

.. seealso::

   Percona Blog: 

   * `Physical Backup Support in Percona Backup for MongoDB <https://www.percona.com/blog/physical-backup-support-in-percona-backup-for-mongodb/>`_
   * `$backupCursorExtend in Percona Server for MongoDB <https://www.percona.com/blog/2021/06/07/experimental-feature-backupcursorextend-in-percona-server-for-mongodb/>`_

`Logical` backup is the copying of the actual database data. A ``pbm-agent`` connects to the database, retrieves the data and writes it to the remote backup storage. During the restore, the reverse process occurs: the ``pbm-agent`` retrieves the backup data from the storage and inserts it to the ``dbPath`` data directory.

Logical backups allow for point in time recovery 

.. list-table::
   :widths: auto
   :header-rows: 1
  
   * - **Type**
     - **Advantages**
     - **Disadvantages**
   * - **Physical**
     - * Faster backup and restore speed
       * Recommended for big, multi-terabyte datasets
       * No database overhead
     - * The backup size is bigger than for logical backups due to data fragmentation extra cost of keeping data and indexes in appropriate data structures 
       * Extra manual operations are required after the restore
       * Point in time recovery is not supported
   * - **Logical**
     - * Easy to operate with, using a single command 
       * Support for incremental backups and point-in-time recovery
       * The backup size is smaller as it includes only the data
     - * Much slower than physical backup / restore
       * Adds database overhead on reading and inserting the data


.. [1] Tech Preview Features are not yet ready for enterprise use and are not included in support via |SLA|. They are included in this release so that users can provide feedback prior to the full release of the feature in a future |GA| release (or removal of the feature if it is deemed not useful). This functionality can change (APIs, CLIs, etc.) from tech preview to GA.
         
.. include:: .res/replace.txt
