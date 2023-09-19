# View detailed information about a backup

To view a detailed information about a backup, run the following command:

```{.bash data-prompt="$"}
$ pbm describe-backup <backup-name>
```

The output provides the backup name, type, status, size and the information about the cluster topology it was taken in. For [selective backups](../features/selective-backup.md), it also shows the namespaces that were backed up. 

??? example "Output"

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

!!! admonition "Version added: [2.3.0](../release-notes/2.3.0.md)"

You can view the list of collections included in the *logical* or *selective* backup. This simplifies troubleshooting as it helps identify the backup contents for environments where databases are frequently created or dropped.

To view the backup contents, use the `--with-collections` flag:

```{.bash data-prompt="$"}
$ pbm describe-backup <backup-name> --with-collections
```

??? example "Output"

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
