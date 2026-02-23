# Snapshot-based restore with pbm-agent restart

Percona Backup for MongoDB supports restarting pbm-agent only at the `copyReady` step during an external (snapshot-based) restore. At `copyReady`, `mongod` is down and the `datadir` is wiped, so nodes wait for new snapshot data files to be provided by an external mechanism (snapshot/rsync/etc.).


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

2. After files are in place, restart the agent on every node. 

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


    ??? info "What happens under the hood"
        Normal `pbm-agent` startup needs a mongod connection, but at `copyReady` mongod is down (and the datadir is wiped), so the agent can’t initialize. `pbm-agent` `restore-finish` starts the agent in a special finalize-only mode so it can complete the external restore without connecting to mongod, then exit.

3. Continue the restore.

    After all agents are restarted and waiting at `copyReady`:

    ```bash
    pbm restore-finish <restore_name> -c <pbm-conf.yaml>
    ```

## Limitation

Only external backups created with PBM are supported. Backups created outside PBM are not supported.

## Next steps

- Learn more about external restores in [External restore](./restore-external.md).
- See how to monitor and troubleshoot restores in [Track restore progress](./restore-progress.md).
