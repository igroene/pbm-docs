# Point-in-time recovery oplog replay

!!! admonition "Version added: 1.7.0"

You can replay the [oplog](../reference/glossary.md#oplog) for a specific period on top of any backup: logical, physical, storage level snapshot (like EBS-snapshot). Starting with version 1.8.0, you can save oplog slices without the mandatory base backup snapshot. This behavior is controlled by the `pitr.oplogOnly` configuration parameter:

```yaml
pitr:
   oplogOnly: true
```

By replaying these oplog slices on top of the backup snapshot with the [`pbm oplog replay`](../reference/pbm-commands.md#pbm-oplog-replay) command, you can manually restore sharded clusters and non sharded replica sets to a specific point in time from a backup made by any tool and not only by Percona Backup for MongoDB. Plus, you reduce time, storage space and administration efforts on making the redundant base backup snapshot.

!!! warning

    Use the oplog replay functionality with caution, only when you are sure about the starting time to replay oplog from. The oplog replay does not guarantee data consistency when restoring from any backup, however, it is less error-prone for backups made with Percona Backup for MongoDB.

## Oplog replay for physical backups

To replay the oplog on top of physical backups made with Percona Backup for MongoDB, do the following:


1. Stop point-in-time recovery, if enabled, to release the lock.


2. Run `pbm status` or `pbm list` commands to find oplog chunks available for replay.


3. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags with the timestamps.

    ```sh
    pbm oplog-replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"
    ```

4. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

## Oplog replay for storage level snapshots

When making a backup, Percona Backup for MongoDB stops the point-in-time recovery. This is done to maintain data consistency after the restore.

Storage-level snapshots are saved with point-in-time recovery enabled. Thus, after the database restore from such a backup, point-in-time recovery is automatically enabled and starts oplog slicing. These new oplog slices might conflict with the existing oplogs saved during the backup. To replay the oplog in such a case, do the following after the restore:


1. Disable point-in-time recovery
2. Delete the oplog slices that might have been created
3. Resync the data from the storage
4. Run the `pbm oplog-replay` command and specify the `--start` and `--end` flags with the timestamps.

    ```sh
    pbm oplog-replay --start="2022-01-02T15:00:00" --end="2022-01-03T15:00:00"
    ```

5. After the oplog replay, make a fresh backup and enable the point-in-time recovery oplog slicing.

### Known limitations

The oplog replay fails if you rename the entire database or a collection.
