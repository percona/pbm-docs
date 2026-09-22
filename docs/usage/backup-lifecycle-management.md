# Backup lifecycle management

Backup lifecycle management helps you control how long Percona Backup for MongoDB (PBM) keeps backups. You define a retention policy, and PBM identifies the backups that have aged out and can be removed.

PBM uses a Grandfather-Father-Son (GFS) retention model. You can keep every recent backup, then retain fewer recovery points as the backups age. For example, you can keep all backups for seven days, one backup per week for four weeks, and one backup per month for 12 months.

You can define a lifecycle policy for the main storage or for an individual [storage profile](../features/multi-storage.md).

!!! warning

    Lifecycle cleanup permanently deletes the selected backups. Always run `pbm cleanup --lifecycle --dry-run` and review the report before you proceed with deletion.

## How retention works

A lifecycle policy can include three retention tiers:

* **Daily** keeps every eligible backup within the configured number of days.
* **Weekly** keeps one backup of each type in every weekly retention window.
* **Monthly** keeps one backup of each type in every monthly retention window.

PBM evaluates the three tiers independently. A backup is retained if it matches at least one tier. For example, the same backup can satisfy the daily, weekly, and monthly rules.

Set a retention value to `0` to disable that tier.

!!! warning
    - If you set all three retention values and `minKeep` to `0`, PBM can select every eligible backup for deletion.

    - PBM evaluates backup times in UTC. In-progress backups and backups created at or after the evaluation time are not considered for deletion.

## Choose a retention strategy

PBM supports `rolling` and `calendar` retention strategies.

### Rolling strategy

The `rolling` strategy is the default. It measures backup age from the time the lifecycle command runs and divides the history into fixed windows:

* Weekly retention uses seven-day windows.
* Monthly retention uses 30-day windows.

PBM keeps the newest backup of each type in every window. This strategy works well when backup times vary or an occasional scheduled backup is missed.

```yaml
lifecycle:
  strategy: rolling
```

### Calendar strategy

The `calendar` strategy groups backups by calendar week and calendar month. Within each period, PBM selects the backup closest to the configured target day:

* `weeklyDay` sets the target weekday. Use `0` for Sunday through `6` for Saturday.
* `monthlyDay` sets the target day of the month. Use a value from `1` through `31`.

If no backup exists on the target day, PBM keeps the closest eligible backup in that week or month. When `monthlyDay` is later than the last day of a month, PBM uses the last day of that month as the target.

??? example "Keep Friday and mid-month backups"

    ```yaml
    lifecycle:
      strategy: calendar
      weeklyRetention: 8
      weeklyDay: 5
      monthlyRetention: 6
      monthlyDay: 15
    ```

    This policy keeps one backup of each type closest to Friday in every retained week and one backup of each type closest to the 15th in every retained month.

## Configuration options

Define lifecycle settings in the `lifecycle` section of the [PBM configuration](../reference/config.md).

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `lifecycle.enabled` | Boolean | `false` | Enables lifecycle policy evaluation and cleanup. When disabled, PBM keeps all backups. |
| `lifecycle.strategy` | String | `rolling` | Sets the retention strategy. Supported values are `rolling` and `calendar`. |
| `lifecycle.minKeep` | Integer | `1` | Sets the minimum number of completed, successful restore points that must remain. PBM aborts the cleanup if it would retain fewer restore points. Set to `0` to disable this safeguard. |
| `lifecycle.purgeFailed` | Boolean | `false` | Controls the retention of failed and canceled backups. When `false`, PBM keeps them. When `true`, PBM keeps them only during the daily retention period. |
| `lifecycle.dailyRetention` | Integer | `0` | Sets the number of days to keep every eligible backup. |
| `lifecycle.weeklyRetention` | Integer | `0` | Sets the number of weeks for which PBM keeps one backup of each type per week. |
| `lifecycle.weeklyDay` | Integer | `0` | Sets the target weekday for the `calendar` strategy. Use `0` for Sunday through `6` for Saturday. |
| `lifecycle.monthlyRetention` | Integer | `0` | Sets the number of months for which PBM keeps one backup of each type per month. |
| `lifecycle.monthlyDay` | Integer | `1` | Sets the target day of the month for the `calendar` strategy. Use a value from `1` through `31`. |

You can set an individual option from the command line:

```bash
pbm config --set lifecycle.dailyRetention=7
```

To apply several settings together, add the `lifecycle` section to the configuration file:

```bash
pbm config --file <PATH_TO_CONFIG_FILE> --wait
```

## Example policies

### Rolling retention

The following policy keeps all eligible backups for seven days, one backup of each type per week for four weeks, and one backup of each type per 30-day window for 12 months:

```yaml
lifecycle:
  enabled: true
  strategy: rolling
  minKeep: 1
  purgeFailed: true
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

With `purgeFailed: true`, failed and canceled backups are kept during the seven-day daily retention period and become eligible for deletion after that period.

### Calendar retention

The following policy keeps all eligible backups for 14 days, one backup of each type closest to Friday for eight calendar weeks, and one backup of each type closest to the 15th for six calendar months:

```yaml
lifecycle:
  enabled: true
  strategy: calendar
  minKeep: 1
  purgeFailed: false
  dailyRetention: 14
  weeklyRetention: 8
  weeklyDay: 5
  monthlyRetention: 6
  monthlyDay: 15
```

## Preview and run lifecycle cleanup

Configure the policy and enable it before you run a dry run. When `lifecycle.enabled` is `false`, PBM reports all backups as retained and does not calculate deletion candidates.
{.power-number}

1. Enable the policy:

    ```bash
    pbm config --set lifecycle.enabled=true --wait
    ```

2. Preview the result:

    ```bash
    pbm cleanup --lifecycle --dry-run
    ```

    PBM reports the backups it would keep and purge without deleting any data. Review the **Backups to PURGE** list carefully.

3. Run the cleanup:

    ```bash
    pbm cleanup --lifecycle --wait
    ```

    PBM displays the lifecycle report and asks you to confirm the deletion. Enter `N` or press `Ctrl+C` at the confirmation prompt to cancel.

    The `--wait` flag keeps the command attached until the cleanup finishes. Without it, PBM starts the cleanup and returns control to the shell.

    !!! note
        Use `--yes` to skip the confirmation prompt only after you have reviewed a dry run. This flag is required for unattended cleanup.

## Apply a policy to a storage profile

The lifecycle policy in the main PBM configuration applies only to backups in the main storage. A policy in a storage profile applies only to backups created in that profile.

For example, create `pbm-physical.conf` with the storage and lifecycle settings for the profile:

```yaml
storage:
  type: s3
  s3:
    region: us-east-1
    bucket: mongo-physical-backups
    prefix: pbm/physical

lifecycle:
  enabled: true
  strategy: rolling
  minKeep: 1
  purgeFailed: true
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

Add the profile:

```bash
pbm profile add physical-backup pbm-physical.conf --wait
```

Preview the policy for this profile:

```bash
pbm cleanup --lifecycle --profile=physical-backup --dry-run
```

Run the cleanup after reviewing the report:

```bash
pbm cleanup --lifecycle --profile=physical-backup --wait
```

When you specify `--profile`, PBM uses the lifecycle policy from that profile and evaluates only its backups. Without `--profile`, PBM uses the main configuration and evaluates backups in the main storage.

For details about creating and managing storage profiles, see [Multiple storages for backups](../features/multi-storage.md).

## Retention by backup type

For weekly and monthly retention, PBM evaluates logical, physical, and incremental backups separately. One backup type cannot replace another as the retained backup for a window. If a weekly window contains eligible logical and physical backups, PBM keeps one of each type.

The same retention values apply to all backup types in a given storage. To use different retention periods, send the backup types to separate storage profiles and define a lifecycle policy in each profile.

PBM also preserves incremental backup dependencies. If any backup in an incremental chain must be retained, PBM keeps the complete chain required to restore it.

## Automate lifecycle cleanup

You can schedule lifecycle cleanup with `cron` or another scheduler. Use `--yes` so the command does not wait for confirmation, and keep `minKeep` enabled as a safeguard.

Run lifecycle cleanup at a different time from scheduled backups. The following example evaluates the main storage every day at 03:00. It writes JSON output to one log file and progress messages or errors to another:

```bash
0 3 * * * /usr/bin/pbm cleanup --lifecycle --yes --wait --out=json >> /var/log/pbm-lifecycle.jsonl 2>> /var/log/pbm-lifecycle.err
```

To automate cleanup for a storage profile, include `--profile=<PROFILE_NAME>`:

```bash
0 3 * * * /usr/bin/pbm cleanup --lifecycle --profile=physical-backup --yes --wait --out=json >> /var/log/pbm-lifecycle-physical.jsonl 2>> /var/log/pbm-lifecycle-physical.err
```

## Cleanup safeguards

PBM applies the following safeguards when it evaluates and deletes backups:

| **Situation** | **PBM behavior** |
| --- | --- |
| Lifecycle management is disabled | Keeps all backups and does not produce deletion targets. |
| Cleanup would retain fewer successful restore points than `minKeep` | Aborts the cleanup. |
| A backup is required as the base for an active [point-in-time recovery](../features/point-in-time-recovery.md) timeline | Keeps the backup and reports it as a PITR base snapshot. |
| A retained backup belongs to an incremental chain | Keeps the complete chain required to restore that backup. |
| An incremental chain is incomplete or invalid | Keeps the affected chain members. |
| A backup is in progress | Excludes it from lifecycle evaluation. |
| A selective backup is present | Excludes it from lifecycle cleanup and keeps it. |
| A backup matches more than one retention rule | Keeps it until it no longer matches any rule. |
| A backup failed or was canceled | Keeps it when `purgeFailed` is `false`. When `purgeFailed` is `true`, keeps it only within the daily retention period. Failed and canceled backups are not selected for weekly or monthly retention. |

## Next steps

* [Configure PBM](../reference/config.md)
* [Multiple storages for backups](../features/multi-storage.md)
* [Delete backups](../usage/delete-backup.md)
* [Point-in-time recovery](../features/point-in-time-recovery.md)
