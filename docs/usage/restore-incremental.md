# Restore from an incremental backup

--8<-- "restore-intro.md"

## Considerations

1. The Percona Server for MongoDB version for both backup and restore data must be within the same major release.
2. Incremental backups made with PBM before PBM 2.1.0 are incompatible for restore with PBM 2.1.0 and onwards.
3. Physical restores are not supported for deployments with arbiter nodes.
   
## Before you start

1. Shut down all `mongos` nodes, `pmm-agent` processes and clients that can do writes to the database as it won’t be available while the restore is in progress.
2. Check that the `systemctl` restart policy for the `pbm-agent.service` is not set to `always` or `on-success`:

    ```{.bash data-prompt="$"}
    sudo systemctl show pbm-agent.service | grep Restart
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        Restart=no
        RestartUSec=100ms
        ```
    
    During physical restores, the database must not be automatically restarted as this is controlled by the `pbm-agent`.
   
## Restore a database

Restore flow from an incremental backup is the same as the restore from a full physical backup: specify the backup name for the `pbm restore` command:

```{.bash data-prompt="$"}
$ pbm restore 2022-11-25T14:13:43Z
```

Percona Backup for MongoDB recognizes the backup type, finds the base incremental backup, restores the data from it and then restores the modified data from applicable incremental backups.

### Post-restore steps

After the restore is complete, do the following:

1. Restart all `mongod` nodes and `pbm-agents`. 

    !!! note

        You may see the following message in the `mongod` logs after the cluster restart:

        ```{.text .no-copy}
        "s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}}}
        ```

        This is expected behavior of periodic checks upon the database start. During the restore, the `config.system.sessions` collection is dropped but Percona Server for MongoDB recreates it eventually. It is a normal procedure. No action is required from your end.
    
2. Resync the backup list from the storage. 
3. Start the balancer and the `mongos` node.
4. As the general recommendation, make a new base backup to renew the starting point for subsequent incremental backups.


## Useful links 

* [View restore progress](../usage/restore-progress.md)




  



