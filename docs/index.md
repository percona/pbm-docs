# Percona Backup for MongoDB Documentation

Percona Backup for MongoDB (PBM) is an open source and distributed solution for consistent backups and restore of [MongoDB sharded clusters and replica sets](details/deployments.md). There is no notable performance nor operating degradation associated with PBM.

You can make backups on a running server and restore your database to a specific point in time using the PBM command line interface. To do these tasks from a user interface, [use PBM with Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/get-started/backup/index.html). Read more [how PBM works :material-arrow-top-right:](intro.md).


!!! note ""

    This is the documentation for the latest release, **PBM {{release}}** ([Release Notes](release-notes/{{release}}.md)).

## What's in it for you?

* Data consistency across clusters and replica sets with [every supported backup type](features/backup-types.md)
* A variety of [supported storage types](details/storage-configuration.md) means no vendor lock-in
* Open source solution with [enterprise-grade features](features/comparison.md) 

<div data-grid markdown><div data-banner markdown>

## :material-progress-download: Installation guides { .title }

Ready to try out PBM? Get started quickly with the step-by-step installation instructions.

[Quickstart Guides :material-arrow-right:](installation.md){ .md-button }

</div><div data-banner markdown>

### :material-backup-restore: Backup management { .title }

Learn what you can do to maintain your backup strategy.

[Backup Management :material-arrow-right:](usage/start-backup.md){ .md-button }

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




