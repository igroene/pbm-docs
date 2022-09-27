# Delete backups

Use the ``pbm delete-backup`` command for backup rotation. With this command you can delete either a specified backup snapshot or all backup snapshots older than the specified time.

!!! note 

    You can only delete a backup that is not running (has the “done” or the “error” state). To check the backup state, run the [`pbm status`](../reference/pbm-commands.md#pbm-status) command.

Starting with version 1.6.0, the command deletes only backup snapshots. Starting with version 2.0.0, you can also delete [selective backups](selective-backups.md). 

To delete point-in-time recovery oplog slices, use the [`pbm delete-pitr`](../reference/pbm-commands.md#pbm-delete-pitr) command.


## Considerations

To ensure oplog continuity for [point-in-time restore](point-in-time-recovery.md#restore-to-the-point-in-time), the `pbm delete-backup` command deletes any backup(s) but for the following ones:

* A backup that can serve as the base for any point in time recovery and has point-in-time recovery time ranges deriving from it

* The most recent backup if point-in-time recovery is enabled and there are no oplog slices following this backup yet.

To illustrate this, let’s take the following `pbm list` output:

```
Backup snapshots:
2021-07-20T03:10:59Z [restore_to_time: 2021-07-20T03:21:19Z]
2021-07-21T22:27:09Z [restore_to_time: 2021-07-21T22:36:58Z]
2021-07-24T23:00:01Z [restore_to_time: 2021-07-24T23:09:02Z]
2021-07-26T17:42:04Z [restore_to_time: 2021-07-26T17:52:21Z]

PITR <on>:
2021-07-21T22:36:59-2021-07-22T12:20:23
2021-07-24T23:09:03-2021-07-26T17:52:21
```

You can delete a backup `2021-07-20T03:10:59Z` since it has no time ranges for point-in-time recovery deriving from it. You cannot delete `2021-07-21T22:27:09Z` as it can be the base for recovery to any point in time from the `PITR` time range `2021-07-21T22:36:59-2021-07-22T12:20:23`. Nor can you delete `2021-07-26T17:42:04Z` backup since there are no oplog slices following it yet.

## Delete backup snapshots

To delete a backup, specify the `<backup_name>` as an argument.

```
pbm delete-backup 2021-12-20T13:45:59Z
```

By default, the ``pbm delete-backup`` command asks for your confirmation
to proceed with the deletion. To bypass it, add the `-f` or
`--force` flag.

```
pbm delete-backup --force 2021-04-20T13:45:59Z
```

To delete backups that were created before the specified time, pass the `--older-than` flag to the `pbm delete-backup` command. Specify the timestamp as an argument for `pbm delete-backup` in the following format:

* `%Y-%M-%DT%H:%M:%S` (for example, 2021-04-20T13:13:20Z) or
* `%Y-%M-%D` (2021-04-20).

### Example

View backups:

```sh
pbm list
```

**Output**:

```
Backup snapshots:
  2021-04-20T20:55:42Z
  2021-04-20T23:47:34Z
  2021-04-20T23:53:20Z
  2021-04-21T02:16:33Z
```

Delete backups created before the specified timestamp

```sh
pbm delete-backup -f --older-than 2021-04-21
```

**Output**:

```
Backup snapshots:
  2021-04-21T02:16:33Z
```

### Delete oplog slices

Running `pbm delete-pitr` allows you to delete old and/or unnecessary slices and save storage space. You can either delete all chunks by passing the  `--all` flag. Or you can delete all slices that are made earlier than the specified time by passing the `--older-than` flag. In this case, specify the timestamp as an argument for `pbm delete-pitr` in the following format:

* `%Y-%M-%DT%H:%M:%S` (for example, 2021-07-20T10:01:18) or
* `%Y-%M-%D` (2021-07-20).

```sh
pbm delete-pitr --older-than 2021-07-20T10:01:18
```

To enable [point in time recovery](point-in-time-recovery.md#restore-to-the-point-in-time) from the most recent backup snapshot, Percona Backup for MongoDB does not delete slices that were made after that snapshot. For example, if the most recent snapshot is `2021-07-20T07:05:23Z [restore_to_time: 2021-07-21T07:05:44]` and you specify the timestamp `2021-07-20T07:05:44`, Percona Backup for MongoDB deletes only slices that were made before `2021-07-20T07:05:23Z`.

