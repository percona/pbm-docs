# View detailed information about a backup

To view a detailed information about a backup, run the following command:

```bash
pbm describe-backup <backup-name>
```

The output provides the backup name, type, status, size and the information about the cluster topology it was taken in. For [selective backups](../features/selective-backup.md), it also shows the namespaces that were backed up. 

??? example "Sample output"

    ```{.text .no-copy}
    name: "2022-08-17T10:49:03Z"
    type: logical
    last_write_ts: 1662039300,2
    last_transition_ts: "1662039304"
    namespaces:
    - Invoices.*
    mongodb_version: 5.0.10-9
    pbm_version: 2.0.0
    status: done
    size: 10234670
    error: ""
    replsets:
    - name: rs1
      status: done
      iscs: false
      last_write_ts: 1662039300,2
      last_transition_ts: "1662039304"
      error: ""
    ```

## Backup timing

PBM shows the start time, finish time, and duration of a backup. You can use this information to compare backup performance and identify operations that take longer than expected.

You can view this timing information with the following commands:

* `pbm status`
* `pbm list`
* `pbm describe-backup`
* `pbm logs`

### Check duration with `pbm status`

The **Snapshots** section of the `pbm status` output includes a `DURATION` column:

```bash
pbm status
```

??? example "Sample output"

    ```text
    Cluster:
    ...

    Snapshots:
      NAME                      SIZE        TYPE          PROFILE               SEL    BASE  RESTORE TIME         DURATION    STATUS
      ------------------------------------------------------------------------------------------------------------------------------
      2026-08-24T12:17:58Z      840.17MB    logical                             no     no    2026-08-24T12:21:36  3m51s       done
      2026-08-24T12:17:24Z      2.06GB      physical                            no     no    2026-08-24T12:17:27  20s         done
    ```
### Check duration with `pbm list`

The `pbm list` output includes the duration of every listed backup:

```bash
pbm list
```

??? example "Sample output"

    ```text
    Backup snapshots:
      NAME                      TYPE          PROFILE               SELECTIVE   BASE    RESTORE TIME         DURATION
      ---------------------------------------------------------------------------------------------------------------
      2026-08-24T12:17:24Z      physical                            no          no      2026-08-24T12:17:27  20s
      2026-08-24T12:17:58Z      logical                             no          no      2026-08-24T12:21:36  3m51s
    ```

For more information, see [List backups](list-backup.md).

### Check start and finish times with `pbm describe-backup`

Use `pbm describe-backup` to view the start time, finish time, and duration of a specific backup:

```bash
pbm describe-backup 2026-08-24T12:17:58Z
```

??? example "Sample output"
    ```{.yaml .no-copy}
    name: "2026-08-24T12:17:58Z"
    ...
    start: "2026-08-24T12:17:58Z"
    finish: "2026-08-24T12:21:49Z"
    duration: 3m51s
    ...
    ```

The timing fields have the following meanings:

| **Field**   | **Description**                          |
|-------------|------------------------------------------|
| `start`     | Time when PBM started the backup         |
| `finish`    | Time when the backup finished            |
| `duration`  | Elapsed time between start and finish    |

Start and finish timestamps are shown in UTC and use the [RFC 3339 format :octicons-link-external-16:](https://www.rfc-editor.org/rfc/rfc3339){target=_blank}. Durations are written in formats: `20s, 3m51s, 1h14m2s`. Anything under a minute shows seconds only.

For a backup that is still running, the finish time and duration are not available. PBM also omits the duration when the stored timestamps do not form a valid interval.

### Check timing in the logs with `pbm logs`

When a backup finishes, the `pbm logs` output includes a summary with the backup name, start time, finish time, and duration:

```bash
pbm logs
```

??? example "Sample output"
    ```{.text .no-copy}
    ...
    2026-08-24T12:21:49Z I [cfg/cfg00:30000] [backup/2026-08-24T12:17:58Z] backup finished
    2026-08-24T12:21:49Z I [cfg/cfg00:30000] [backup/2026-08-24T12:17:58Z] backup: 2026-08-24T12:17:58Z, start: 2026-08-24T12:17:58Z, finish: 2026-08-24T12:21:49Z, duration: 3m51s
    ...
    ```

To view log entries for a specific backup, filter by the backup event:

```bash
pbm logs --event=backup/2026-08-24T12:17:58Z
```
For more information about filtering log output, see [View backup logs](logs.md).

!!! note
    Backup duration and restore time represent different values. Duration shows how long the backup operation ran. Restore time identifies the latest point to which the backup can restore data. See [Restore to time](list-backup.md#restore-to-time).


## View backup size

!!! admonition "Version added: 2.10.0"

The command output displays the uncompressed backup size for the whole cluster and the compressed/uncompressed size for each replica set. This helps PBM evaluate the required disk space when doing [physical restores with a fallback directory](../features/physical.md#physical-restores-with-a-fallback-directory).

??? example "Sample output"

    ```{.text .no-copy}
    pbm describe-backup 2025-06-05T16:57:35Z
    name: "2025-06-05T16:57:35Z"
    opid: 6841cc7f1f576b79efb26752
    type: physical
    ...
    status: done
    size_h: 3.3 GiB
    size_uncompressed_h: 4.0 GiB
    ....
    replsets:

    - name: rs2
      status: done
      node: rs202:30202
      size_h: 3.3 GiB
      size_uncompressed_h: 3.6 GiB
    ```

## View collections in a backup

!!! admonition "Version added: [2.3.0](../release-notes/2.3.0.md)"

You can view the list of collections included in the *logical* or *selective* backup. This simplifies troubleshooting as it helps identify the backup contents for environments where databases are frequently created or dropped.

To view the backup contents, use the `--with-collections` flag:

```bash
pbm describe-backup <backup-name> --with-collections
```

??? example "Sample output"

    ```{.text .no-copy}
    name: "2023-09-14T14:44:33Z"
    opid: 65031c51e6a16fa0e3deeb5f
    type: logical
    last_write_time: "2023-09-14T14:44:39Z"
    last_transition_time: "2023-09-14T14:44:57Z"
    mongodb_version: 6.0.9-7
    fcv: "6.0"
    pbm_version: 2.2.1
    status: done
    size_h: 89.3 KiB
    replsets:
    - name: rs0
      status: done
      node: rs00:30000
      last_write_time: "2023-09-14T14:44:38Z"
      last_transition_time: "2023-09-14T14:44:56Z"
      collections:
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - db0.c0
      - db0.c1
      - db1.c0
    - name: rs1
      status: done
      node: rs10:30100
      last_write_time: "2023-09-14T14:44:38Z"
      last_transition_time: "2023-09-14T14:44:49Z"
      collections:
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - db0.c0
      - db1.c0
      - db1.c1
    - name: cfg
      status: done
      node: cfg0:27000
      last_write_time: "2023-09-14T14:44:39Z"
      last_transition_time: "2023-09-14T14:44:42Z"
      configsvr: true
      collections:
      - admin.pbmAgents
      - admin.pbmBackups
      - admin.pbmCmd
      - admin.pbmConfig
      - admin.pbmLock
      - admin.pbmLockOp
      - admin.pbmLog
      - admin.pbmOpLog
      - admin.pbmPITRChunks
      - admin.pbmRRoles
      - admin.pbmRUsers
      - admin.system.roles
      - admin.system.users
      - admin.system.version
      - config.chunks
      - config.collections
      - config.databases
      - config.settings
      - config.shards
      - config.tags
      - config.version
    ```
