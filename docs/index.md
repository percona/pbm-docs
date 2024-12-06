# Percona Backup for MongoDB documentation

!!! note ""

    This is the documentation for the latest release, **PBM {{release}}** ([Release Notes](release-notes/{{release}}.md)).

Percona Backup for MongoDB (PBM) is an open source and distributed solution for consistent backups and restores of [MongoDB sharded clusters and replica sets](details/deployments.md) to a specific point in time. 

Here's how you can make backups on a running server and/or restore your database to a specific point in time:

* using the [PBM command line interface](reference/pbm-commands.md). 
* from a web interface [with PBM and Percona Monitoring and Management :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html). 

Read [how PBM works](intro.md). Check [supported MongoDB versions](details/versions.md).


## Why to choose PBM?

* Data consistency across clusters and replica sets with [every supported backup type](features/backup-types.md)
* No notable performance nor operating degradation associated with PBM
* A variety of [supported storage types](details/storage-configuration.md) means no vendor lock-in
* Open source solution with [enterprise-grade features](features/comparison.md) 

<div data-grid markdown><div data-banner markdown>

### :material-progress-download: Installation guides { .title }

Ready to try out PBM? Get started quickly with the step-by-step installation instructions.

[Quickstart guides :material-arrow-right:](installation.md){ .md-button }

</div><div data-banner markdown>

### :material-backup-restore: Backup management { .title }

Learn what you can do to maintain your backup strategy.

[Backup management :material-arrow-right:](usage/start-backup.md){ .md-button }

</div><div data-banner markdown>

### :fontawesome-solid-gears: Administration { .title }

Tweak PBM to effectively perform your day-to-day operations.

[Administration :material-arrow-right:](manage/overview.md){.md-button}
</div><div data-banner markdown>

### :material-frequently-asked-questions: Diagnostics and FAQ { .title }

Our comprehensive resources will help you overcome challenges, from everyday issues to specific doubts.

[Run diagnostics :material-arrow-right:](troubleshoot/index.md){.md-button}

</div>
</div>



