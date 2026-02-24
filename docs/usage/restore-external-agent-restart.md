# Snapshot-based restore with pbm-agent restart

Percona Backup for MongoDB supports the `pbm-agent` to be restarted only at the `copyReady` step during an external (snapshot-based) restore. At this stage, `mongod` is stopped, and the `datadir` has been cleared. Consequently, the nodes will remain on hold, waiting for new snapshot data files to be supplied by an external method, such as snapshotting or rsync.


## Procedure

This workflow lets you pause an external restore at `copyReady`, restart `pbm-agent`, copy snapshot data into the wiped `datadir`, and then resume the restore to completion.
{.power-number}

1. Start external restore and terminate agents at `copyReady`.

    ```bash
    pbm restore --external --exit
    ```

    Where:

    - Without the `--exit` flag, agents wait at `copyReady` until data files appear.
    - With the `--exit` flag, each agent exits automatically when it reaches `copyReady`.

2. After files are in place, start `pbm-agent` on every node.

    !!! note
        This step is required only if the agents were stopped with the `--exit` parameter.

    Run the following on every node in the restore:

    ```bash
    pbm-agent restore-finish <restore_name> \
      -c <pbm-config.yaml> \
      --rs <rs_name> --node <node_name> \
      [--db-config <db-config.yaml>]
    ```

    **Parameters:**

    - `<restore_name>` (required): used to find temporary restore data on backup storage.
    - `-c/--config` (required): PBM config to access backup storage.
    - `--rs, --node` (required): identify the node without connecting to mongod.
    - `--db-config` (optional): required only when you use encryption-at-rest (PBM does not store encryption options in metadata).



    This is the configuration file that PBM will use during the restore. It should contain the [security options :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/configuration-options/#security-options)                

        ```bash
        security:
           keyFile: <string>
           clusterAuthMode: <string>
           authorization: <string>
           transitionToAuth: <boolean>
           javascriptEnabled:  <boolean>
           redactClientLogData: <boolean>
           clusterIpSourceAllowlist:
             - <string>
           ```

    ??? info "What happens under the hood"
        Normal `pbm-agent` startup needs a mongod connection, but at `copyReady` mongod is down (and the datadir is wiped), so the agent can’t initialize. `pbm-agent` `restore-finish` starts the agent in a special finalize-only mode so it can complete the external restore without connecting to mongod, then exit.

3. Continue the restore.

    After all agents are restarted and waiting at `copyReady`, run once from any host with PBM CLI access:

    ```bash
    pbm restore-finish <restore_name> -c <pbm-config.yaml>
    ```

## Limitation

Only external backups created with PBM are supported. Backups created outside PBM are not supported.

## Useful links

- Learn more about external restores in [External restore](./restore-external.md).
- See how to monitor and troubleshoot restores in [Track restore progress](./restore-progress.md).
