## Before you start

Before taking a backup, verify PBM is installed and running.
{.power-number}

1. [Install](../installation.md) and [set up Percona Backup for MongoDB](../install/initial-setup.md)
2. Check that `pbm agent` is running with the [`pbm status`](../reference/pbm-commands.md#pbm-status) command
3. Check that all `pbm-agents` and PBM CLI have the same version. Otherwise we cannot guarantee successful backups and data consistency in them. 

    To check the version, run the following commands:

    *  [`pbm status`](../reference/pbm-commands.md#pbm-status) to check the version of pbm-agents 
    *  [`pbm version`](../reference/pbm-commands.md#pbm-status) to check the version of PBM CLI. 
