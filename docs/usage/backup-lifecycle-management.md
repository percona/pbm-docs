# Backup lifecycle management

The Backup Lifecycle Management feature automates the retention and rotation of Percona Backup for MongoDB (PBM) backups. This feature allows administrators to define a **Grandfather-Father-Son (GFS)** style retention policy to automatically purge aged data while preserving specific historical recovery points for long-term compliance, disaster recovery, and cost management.

PBM supports two retention strategies:

- `rolling` keeps the newest available backup in each retention window.

- `calendar` keeps backups from configured days of the week or month.

A lifecycle policy applies to all backups, or to a single storage profile.

!!! warning

    Lifecycle rotation permanently deletes the backups selected for purging. Run `pbm lifecycle --dry-run` and review the report before you enable rotation.

## How retention works

Three retention tiers make up a policy:

- Daily: keeps every completed backup for the configured number of days.

- Weekly: keeps one backup for each weekly retention window.

- Monthly: keeps one backup for each monthly retention window.

Set a retention value to `0` to turn that tier off.

For example, with the following settings:

```yaml
lifecycle:
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

PBM keeps every completed backup from the last seven days, with no thinning inside that window. Take a backup every six hours and all 28 stay. Older backups move into the weekly and monthly tiers, where PBM keeps one backup per window and purges the rest.

### Rolling strategy

The `rolling` strategy is the default.

PBM measures the age of a backup from the time the rotation runs, rather than from the calendar date on the backup. Past the daily window, the timeline splits into seven-day windows for the weekly tier and 30-day windows for the monthly tier. PBM keeps the newest backup in each window.

This behavior helps when backups do not run at the same time every day, or when a scheduled backup is missed. PBM keeps the available backup that fits the window best, so a gap in the schedule does not break the policy.

```yaml
lifecycle:
  strategy: rolling
```

### Calendar strategy

The `calendar` strategy targets configured days of the week and month.

Use `weeklyDay` to specify the day of the week for weekly retention:

- `0` = Sunday

- `1` = Monday

- `2` = Tuesday

- `3` = Wednesday

- `4` = Thursday

- `5` = Friday

- `6` = Saturday

Use `monthlyDay` to specify the day of the month, from `1` to `31`.

??? example

    ```yaml
    lifecycle:
      strategy: calendar
      weeklyRetention: 8
      weeklyDay: 5
      monthlyRetention: 6
      monthlyDay: 15
    ```

    This configuration targets Friday backups for weekly retention and backups from the 15th of the month for monthly retention.

!!! note

    A calendar policy does not require a backup on the target day. If the 15th has no backup, PBM keeps the closest available backup from that month.

## Configuration

Configure lifecycle management in the `lifecycle` section of the PBM configuration.

| **Option** | **Type** | **Default** | **Description** |
| --- | --- | --- | --- |
| `lifecycle.enabled` | Boolean | `false` | Enables lifecycle rotation. |
| `lifecycle.strategy` | String | `rolling` | Retention strategy. Supported values are `rolling` and `calendar`. |
| `lifecycle.minKeep` | Integer | `1` | Minimum number of backups that must remain after a rotation. PBM aborts the rotation if the number would fall below this value. |
| `lifecycle.prompt` | Boolean | `true` | Prompts for confirmation before deleting backups. Set to `false` when you run lifecycle rotation without interactive input. |
| `lifecycle.purgeFailed` | Boolean | `false` | Controls retention of failed and canceled backups. When `false`, PBM protects them indefinitely. When `true`, PBM keeps them for the `dailyRetention` period. |
| `lifecycle.dailyRetention` | Integer | `0` | Number of days to keep every completed backup. |
| `lifecycle.weeklyRetention` | Integer | `0` | Number of weeks to retain one weekly backup. |
| `lifecycle.weeklyDay` | Integer | `0` | Day of the week to target when `strategy` is `calendar`. |
| `lifecycle.monthlyRetention` | Integer | `0` | Number of months to retain one monthly backup. |
| `lifecycle.monthlyDay` | Integer | `1` | Day of the month to target when `strategy` is `calendar`. |

Set individual options from the command line:

```bash
pbm config --set lifecycle.dailyRetention=7
```

You can also apply a configuration file:

```bash
pbm config --file=<PATH_TO_CONFIG_FILE>
```

For details about the configuration file and how to apply it, see [Configure PBM](../reference/config.md).

## Example retention policies

### Rolling retention

The following policy keeps all completed backups for seven days, one weekly backup for four weeks, and one monthly backup for 12 months. This policy suits most deployments.

```yaml
lifecycle:
  enabled: false
  strategy: rolling
  minKeep: 1
  prompt: true
  purgeFailed: true
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

Keep `enabled: false` while you review the policy. Run a dry run before you enable rotation.

### Calendar retention

The following policy keeps all completed backups for 14 days, targets Friday backups for eight weeks, and targets the 15th of each month for six months.

```yaml
lifecycle:
  enabled: false
  strategy: calendar
  minKeep: 1
  prompt: true
  purgeFailed: false
  dailyRetention: 14
  weeklyRetention: 8
  weeklyDay: 5
  monthlyRetention: 6
  monthlyDay: 15
```

## Run a lifecycle rotation

Lifecycle rotation deletes backups permanently, so validate the policy first. The command uses the same connection options, environment variables, and authentication as every other PBM command.

### 1. Run a dry run

Use `--dry-run` to see which backups PBM would keep and purge. The flag works while `lifecycle.enabled` is `false`, so you can test a policy before you turn rotation on.

```bash
pbm lifecycle --dry-run
```

The dry run deletes nothing.

```text
Lifecycle Report (Dry Run: true)
Enabled: false | Strategy: ROLLING | Purge Failed: true
Daily: 7 | Weekly: 4 [Auto (Newest in bucket)] | Monthly: 6 [Auto (Newest in bucket)]

Backups to KEEP (3):
  - 2026-03-26T04:02:01Z
  - 2026-03-22T04:02:01Z
  - 2026-03-15T04:02:01Z

Backups to PURGE (2):
  - 2026-03-25T04:02:02Z
  - 2026-03-24T04:02:01Z
```

Check that the restore points you need appear under `Backups to KEEP`.

### 2. Enable lifecycle rotation

After you verify the dry-run results, enable lifecycle rotation:

```bash
pbm config --set lifecycle.enabled=true
```

### 3. Run the rotation

Run the lifecycle command:

```bash
pbm lifecycle
```

With `lifecycle.prompt` set to `true`, PBM displays the backups selected for retention and purging, then asks for confirmation before it deletes them.

```text
Are you sure you want to permanently delete the purged backups? [y/N]: y
Starting deletion...
Purging backup 2026-03-18T04:02:01Z...
Lifecycle rotation complete.
```

Enter `N` or press `Ctrl+C` at the confirmation prompt to cancel the operation. If deletion has already started, `Ctrl+C` stops the remaining purge operations.

## Use lifecycle policies with storage profiles

You can configure lifecycle policies globally or per storage profile.

| Scope | Description | Command |
| --- | --- | --- |
| Global | Applies the lifecycle policy to backups managed by the global configuration. | `pbm lifecycle` |
| Profile | Applies the lifecycle policy to backups routed to a specific storage profile. | `pbm lifecycle --profile=<PROFILE_NAME>` |

Use a storage profile when different backup sets need different retention periods. A common split keeps physical backups in one profile for a year, and logical backups in another for a few days.

Add the `lifecycle` section to the profile configuration file, alongside the storage settings. The following file, `pbm-physical.conf`, holds the long-term policy:

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
  prompt: true
  purgeFailed: true
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

Apply the configuration:

```bash
pbm profile add physical-backup pbm-physical.conf
```

A second file, `pbm-logical.conf`, holds a shorter retention period. A value of `0` turns off the weekly and monthly tiers:

```yaml
storage:
  type: s3
  s3:
    region: us-east-1
    bucket: mongo-logical-backups
    prefix: pbm/logical
lifecycle:
  enabled: true
  strategy: rolling
  minKeep: 1
  prompt: true
  purgeFailed: true
  dailyRetention: 3
  weeklyRetention: 0
  monthlyRetention: 0
```

```bash
pbm profile add logical-backup pbm-logical.conf
```

Use the `--profile` option to evaluate a specific profile:

```bash
pbm lifecycle --profile=physical-backup --dry-run
pbm lifecycle --profile=logical-backup --dry-run
```

When you name a profile, PBM evaluates the lifecycle policy configured for that profile and ignores the global configuration.

See [Storage profiles](../usage/profiles.md) for information about creating and configuring PBM storage profiles.

## Retention of different backup types

PBM can manage physical, logical, and incremental backups.

Lifecycle management keeps each backup type in a separate retention group. A logical backup does not replace a physical backup in the same retention window, and a physical backup does not replace a logical one.

For example, if a weekly retention window holds both a physical and a logical backup, PBM retains one backup of each type for that window.

If you need different retention periods for different backup types, use separate storage profiles and configure a lifecycle policy for each profile.

!!! note

    Long retention periods for both physical and logical backups increase storage usage. Separate storage profiles give each backup type its own retention period.

## Automate lifecycle rotation

You can run lifecycle rotation from `cron` or another scheduler.

Before you automate the command, disable the interactive confirmation prompt. A prompt in a scheduled job waits for an answer that never arrives:

```bash
pbm config --set lifecycle.prompt=false
```

If you use storage profiles, set `prompt: false` in the lifecycle configuration for each profile that you automate.

Keep the `minKeep` safety setting in place:

```bash
pbm config --set lifecycle.minKeep=1
```

Run lifecycle rotation at a different time from your backup jobs, so that backup and retention operations do not compete for resources. The `--out json` option writes machine-readable output, which suits a log file.

The following `cron` entry runs the global lifecycle rotation every day at 3:00 AM:

```bash
0 3 * * * /usr/bin/pbm lifecycle --out json >> /var/log/pbm-lifecycle-global.log 2>&1
```

You can also schedule profile rotations separately:

```bash
0 2 * * * /usr/bin/pbm lifecycle --profile=physical-backup --out json >> /var/log/pbm-lifecycle-phys.log 2>&1
30 2 * * * /usr/bin/pbm lifecycle --profile=logical-backup --out json >> /var/log/pbm-lifecycle-logi.log 2>&1
```

## Safety checks

PBM applies several checks while it evaluates backups for removal.

| Situation | PBM behavior |
| --- | --- |
| A rotation would leave fewer backups than `minKeep` | PBM aborts the rotation. |
| A backup is required as the base for an active point-in-time recovery (PITR) chain | PBM does not delete the backup and reports an `ErrBaseForPITR` warning. |
| A backup is in progress | Backups in the `starting`, `running`, or `dumpDone` state are excluded from lifecycle evaluation. |
| A backup matches more than one retention rule | PBM keeps the backup until the longest applicable retention period expires. |
| A backup failed or was canceled | With `purgeFailed: false`, PBM protects it indefinitely. With `purgeFailed: true`, PBM keeps it for the `dailyRetention` period. Failed backups are not selected for weekly or monthly retention. |
| No backup exists on a calendar target date | PBM retains the closest available backup from that month. |

If `minKeep` aborts an automated rotation, PBM reports the reason in its output.

```text
WARNING: This rotation would leave you with 0 backup(s), which is below
the safety threshold of 1 (minKeep).

Automated run (prompt: false) detected. Purge aborted to protect your backups.
```

Review the lifecycle configuration and the backups selected for retention before you run the rotation again.

## Related topics

- [Configure backup storage](../reference/config.md)

- [Storage profiles](../usage/profiles.md)

- [Delete backups](../usage/delete-backup.md)

- [Restore a backup](../usage/restore.md)

- [Point-in-time recovery](../features/point-in-time-recovery.md)

