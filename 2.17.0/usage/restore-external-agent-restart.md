> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-backup-mongodb/2.17.0/llms.txt).

# Snapshot-based restore with pbm-agent restart

Percona Backup for MongoDB supports restarting `pbm-agent` only at the `copyReady` step during an external (snapshot-based) restore. At this stage, `mongod` is stopped, and the `datadir` has been cleared. Consequently, the nodes will remain on hold, waiting for new snapshot data files to be supplied by an external method, such as snapshotting or rsync.

## Procedure

This workflow lets you pause an external restore at `copyReady`, restart `pbm-agent`, copy snapshot data into the wiped `datadir`, and then resume the restore to completion.

1. Start external restore and terminate agents at `copyReady`.

   ```
   pbm restore --external --exit
   ```

   Where:

   - Without the `--exit` flag, agents wait at `copyReady` until data files appear.
   - With the `--exit` flag, each agent exits automatically when it reaches `copyReady`.

   Version added: [2.14.0](https://docs.percona.com/percona-backup-mongodb/2.17.0/release-notes/2.14.0.html)

   Before a restore operation is executed you have to confirm the action (to bypass it, add the `-y` or `--yes` flag).

1. After files are in place, start `pbm-agent` on every node.

   Note

   This step is required only if the agents were stopped with the `--exit` parameter.

   Run the following on every node in the restore:

   ```
   pbm-agent restore-finish <restore_name> \
     -c <pbm-config.yaml> \
     --rs <rs_name> --node <node_name> \
     [--db-config <db-config.yaml>]
   ```

   **Parameters:**

   - `<restore_name>` (required): used to find temporary restore data on backup storage.
   - `-c/--config` (required): PBM config to access backup storage.
   - `--rs, --node` (required): identify the node without connecting to mongod.
   - `--db-config` (optional): required only when you restore from an externally taken backup (without PBM) of a database with encryption at rest (in that case, PBM does not store encryption options in metadata). In that case, you have to provide MongoDB configuration manually via this parameter. The configuration must match the configuration from the node from which the backup was taken, not the current one. For external backups assisted by PBM, you can skip that parameter. PBM creates metadata file within `dbPath` with all necessary `mongod` options, so for the external restore PBM can read this file and provide the options to `pbm-agent restore-finish` automatically.

   This is the configuration file that PBM will use during the restore. It should contain the [security options](https://www.mongodb.com/docs/manual/reference/configuration-options/#security-options)

   ```
   security:
      enableEncryption: true
      kmip:
       serverName: <kmip_server_name>
       port: <kmip_port>
       clientCertificateFile: </path/client_certificate.pem>
       serverCAFile: </path/ca.pem>
       keyIdentifier: <key_name>
   ```

   What happens under the hood

   Normal `pbm-agent` startup needs a mongod connection, but at `copyReady` mongod is down (and the datadir is wiped), so the agent can’t initialize. `pbm-agent` `restore-finish` starts the agent in a special finalize-only mode so it can complete the external restore without connecting to mongod, then exit.

1. Continue the restore.

   After all agents are restarted and waiting at `copyReady`, run once from any host with PBM CLI access:

   ```
   pbm restore-finish <restore_name> -c <pbm-config.yaml>
   ```

## Limitation

Only external backups created with PBM are supported. Backups created outside PBM are not supported.

## Useful links

- Learn more about external restores in [External restore](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-external.html).
- See how to monitor and troubleshoot restores in [Track restore progress](https://docs.percona.com/percona-backup-mongodb/2.17.0/usage/restore-progress.html).
