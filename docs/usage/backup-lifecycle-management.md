# Backup lifecycle management

Backup lifecycle management tells Percona Backup for MongoDB (PBM) how long to keep each backup and when to remove the ones that have aged out. You define the policy once, and PBM handles the cleanup.

The policy follows the Grandfather-Father-Son (GFS) retention scheme. Keep every backup for a recent period, then keep fewer as backups age. A common policy keeps every backup for seven days, one backup per week for four weeks, and one backup per month for 12 months.

A lifecycle policy applies to all backups, or to a single storage profile.

!!! warning

    Lifecycle rotation permanently deletes the backups selected for purging. Run `pbm lifecycle --dry-run` and review the report before you enable rotation.

## How retention works

Three retention tiers make up a policy:

- Daily: keeps every completed backup for the configured number of days.

- Weekly: keeps one backup for each weekly retention window.

- Monthly: keeps one backup for each monthly retention window.

Set a retention value to `0` to turn that tier off.

PBM keeps every completed backup inside the daily window, with no thinning. Older backups move into the weekly and monthly tiers, where PBM keeps one backup per window and purges the rest.

## Retention strategies

### Rolling strategy

The `rolling` strategy is the default. PBM measures backup age from the time rotation runs, not from the calendar date. Past the daily window, the timeline splits into seven-day windows for weekly retention and 30-day windows for monthly retention. PBM keeps the newest backup in each window.

This approach is resilient when backups do not run at the same time every day, or when a scheduled backup is missed. PBM selects the best available backup for each window.

```yaml
lifecycle:
  strategy: rolling
```

### Calendar strategy

The `calendar` strategy targets specific days:

- `weeklyDay`: day of the week, from `0` for Sunday to `6` for Saturday.

- `monthlyDay`: day of the month, from `1` to `31`.

??? example "Example"

    ```yaml
    lifecycle:
      strategy: calendar
      weeklyRetention: 8
      weeklyDay: 5
      monthlyRetention: 6
      monthlyDay: 15
    ```

    This policy keeps Friday backups for eight weeks and backups from the 15th of each month for six months.

!!! note

    A calendar policy does not require a backup on the target day. If no backup exists, PBM keeps the closest available backup from that month.

## Configuration options

Configure lifecycle management in the `lifecycle` section of the PBM configuration.

| **Option** | **Type** | **Default** | **Description** |
| --- | --- | --- | --- |
| `lifecycle.enabled` | Boolean | `false` | Enables lifecycle rotation. |
| `lifecycle.strategy` | String | `rolling` | Retention strategy: `rolling` or `calendar`. |
| `lifecycle.minKeep` | Integer | `1` | Minimum backups to keep. Rotation aborts if fewer would remain. |
| `lifecycle.prompt` | Boolean | `true` | Prompts for confirmation before deletion. Set to `false` for scheduled runs. |
| `lifecycle.purgeFailed` | Boolean | `false` | Retention of failed and canceled backups. `false` keeps them indefinitely. `true` keeps them for the daily retention period. |
| `lifecycle.dailyRetention` | Integer | `0` | Days to keep all backups. |
| `lifecycle.weeklyRetention` | Integer | `0` | Weeks to keep one backup per week. |
| `lifecycle.weeklyDay` | Integer | `0` | Target day for weekly retention. Applies to the `calendar` strategy. |
| `lifecycle.monthlyRetention` | Integer | `0` | Months to keep one backup per month. |
| `lifecycle.monthlyDay` | Integer | `1` | Target day for monthly retention. Applies to the `calendar` strategy. |

Set options from the command line:

```bash
pbm config --set lifecycle.dailyRetention=7
```

Or apply a configuration file:

```bash
pbm config --file=<PATH_TO_CONFIG_FILE>
```

## Example policies

### Rolling retention

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

This policy suits most deployments. Keep `enabled: false` while you review it, and run a dry run before you enable rotation.

### Calendar retention

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

This policy keeps backups for 14 days, targets Fridays for weekly retention, and the 15th of each month for six months.

## Running lifecycle rotation

Lifecycle rotation deletes backups permanently. Validate the policy first.
{.power-number}

1. Run a dry run

    ```bash
    pbm lifecycle --dry-run
    ```

    The report shows which backups PBM would keep and purge. No deletions occur. The flag works while `lifecycle.enabled` is `false`, so you can test a policy before you enable rotation.

      ```text
      Lifecycle Report (Dry Run: true)
      Enabled: false | Strategy: ROLLING | Purge Failed: true

        Backups to KEEP (3):
          - 2026-03-26T04:02:01Z
          - 2026-03-22T04:02:01Z
          - 2026-03-15T04:02:01Z

        Backups to PURGE (2):
          - 2026-03-25T04:02:02Z
          - 2026-03-24T04:02:01Z
      ```

2. Enable rotation

    ```bash
    pbm config --set lifecycle.enabled=true
    ```

3. Run the rotation

    ```bash
    pbm lifecycle
    ```

    With `lifecycle.prompt` set to `true`, PBM displays the selected backups and asks for confirmation before deletion. Enter `N` or press `Ctrl+C` to cancel.

## Storage profiles

Lifecycle policies can apply globally or per storage profile.

| **Scope** | **Description** | **Command** |
| --- | --- | --- |
| Global | Applies to all backups. | `pbm lifecycle` |
| Profile | Applies to backups in a specific profile. | `pbm lifecycle --profile=<PROFILE_NAME>` |

Use profiles when different backup sets need different retention. A common split keeps physical backups for the long term and logical backups for a few days.

Add the `lifecycle` section to the profile configuration file, alongside the storage settings:

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

Evaluate a specific profile:

```bash
pbm lifecycle --profile=physical-backup --dry-run
```

When you name a profile, PBM evaluates the lifecycle policy configured for that profile and ignores the global configuration.

## Retention by backup type

PBM manages physical, logical, and incremental backups separately. A logical backup does not replace a physical backup in the same retention window, and a physical backup does not replace a logical one. If a weekly window holds both, PBM keeps one of each.

For different retention periods, use separate storage profiles.

!!! note

    Long retention for both physical and logical backups increases storage usage. Separate profiles help balance storage needs.

## Automating rotation

You can schedule lifecycle rotation with `cron` or another scheduler. Disable interactive prompts first:

```bash
pbm config --set lifecycle.prompt=false
```

Keep `minKeep` in place for safety:

```bash
pbm config --set lifecycle.minKeep=1
```

Run rotation at a different time than your backup jobs to avoid resource contention. Use `--out json` for machine-readable logs.

Example `cron` entry:

```bash
0 3 * * * /usr/bin/pbm lifecycle --out json >> /var/log/pbm-lifecycle-global.log 2>&1
```

## Safety checks

PBM applies safeguards during rotation.

| **Situation** | **Behavior** |
| --- | --- |
| Rotation leaves fewer than `minKeep` | Aborts rotation. |
| Backup is the base for an active point-in-time recovery (PITR) chain | Backup is kept. PBM reports `ErrBaseForPITR`. |
| Backup in progress | Excluded from evaluation. |
| Backup matches multiple rules | Retained until the longest rule expires. |
| Failed or canceled backup | Kept indefinitely with `purgeFailed: false`, or for the daily retention period with `purgeFailed: true`. |
| No backup on calendar target date | Keeps the closest available backup. |

## Next steps

- [Configure PBM](../reference/config.md)

- [Delete backups](../usage/delete-backup.md)

- [Point-in-time recovery](../features/point-in-time-recovery.md)