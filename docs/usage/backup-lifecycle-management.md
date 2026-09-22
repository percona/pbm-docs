# Backup lifecycle management

2026-09-22 · @Rasika Chivate

Backup lifecycle management rotates your backups for you. You define a retention policy once, and Percona Backup for MongoDB (PBM) purges aged backups while it protects the restore points you depend on. The feature follows the Grandfather-Father-Son (GFS) rotation scheme, which holds three tiers at the same time:

- Daily backups, for immediate disaster recovery
- Weekly backups, for short-term historical rollback
- Monthly backups, as long-term anchors for audits

A GFS policy replaces the `cron` and `pbm delete-backup` scripts that many teams maintain by hand. Unlike a script, the engine checks point-in-time recovery (PITR) chains and running backups before a purge.


!!! note

    Lifecycle management is turned off after an upgrade. PBM deletes nothing until you set `lifecycle.enabled` to `true`.


## How rotation works

A backup passes through three stages as the backup ages.

### The daily window

The `dailyRetention` option defines a keep-everything zone, measured backward from the run time. Every successful backup inside the window stays, with no bucketing. A value of `7` with a backup every six hours keeps all 28 backups.

### The weekly and monthly windows

Past the daily window, PBM groups backups into buckets and keeps one backup per bucket. The `strategy` option decides how the grouping works.

| Strategy | Grouping method | Best for |
| --- | --- | --- |
| `rolling` (default) | Time decay. PBM divides the timeline into seven-day and 30-day chunks, then keeps the newest backup in each chunk. | Cloud deployments and schedules that drift. A paused or failed backup job costs you nothing, since PBM keeps the closest available backup in the window. |
| `calendar` | Strict anchoring. PBM keeps backups taken on the exact days named by `weeklyDay` and `monthlyDay`. | Finance, healthcare, and any audit that names a date, such as the end-of-month state of the database. |


!!! note

    Under the `calendar` strategy, a missed anchor date has a fallback. If no backup exists for the 15th, PBM scans that month and keeps the closest backup, such as the one from the 14th or the 16th.


## Mixed backup types

Many deployments take both physical and logical backups. The bucketing engine sorts candidates into separate lanes by type, so one type never crowds out another. Each week and each month keeps its own logical, physical, and incremental anchor. The rule applies to the global configuration and to storage profiles alike.


!!! note

    Type separation protects both backup types, but long-term archives of both consume storage. To keep physical backups for a year and logical backups for three days, use separate storage profiles.


## Configuration options

The `lifecycle` block in the PBM configuration controls the feature:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `lifecycle.enabled` | Boolean | `false` | Turns on automated rotation. While the value stays `false`, PBM purges nothing. |
| `lifecycle.strategy` | String | `rolling` | Sets the rotation algorithm. Values: `rolling` or `calendar`. |
| `lifecycle.minKeep` | Integer | `1` | Circuit breaker. PBM aborts the whole rotation if a purge drops the count of kept backups below this number. |
| `lifecycle.prompt` | Boolean | `true` | Prints the keep and purge lists and waits for a `[y/N]` answer. Set to `false` for `cron` jobs. |
| `lifecycle.purgeFailed` | Boolean | `false` | Controls failed and canceled backups. `false` protects them without a time limit. `true` keeps them for the length of the daily window, then deletes them. |
| `lifecycle.dailyRetention` | Integer | `0` | Number of days to keep every completed backup. Set to `0` to turn the tier off. |
| `lifecycle.weeklyRetention` | Integer | `0` | Number of weeks to keep one weekly backup. Set to `0` to turn the tier off. |
| `lifecycle.weeklyDay` | Integer | `0` | Day of the week to target, from `0` for Sunday to `6` for Saturday. Applies to the `calendar` strategy. |
| `lifecycle.monthlyRetention` | Integer | `0` | Number of months to keep one monthly backup. Set to `0` to turn the tier off. |
| `lifecycle.monthlyDay` | Integer | `1` | Day of the month to target, from `1` to `31`. Applies to the `calendar` strategy. |

Set an option from the command line:

```bash
pbm config --set lifecycle.dailyRetention=7
```

Or add the block to the PBM configuration file and apply the file:

```bash
pbm config --file=<PATH_TO_CONFIG_FILE>
```

## Example policies

### Recommended baseline

For most deployments, start with seven daily, four weekly, and 12 monthly backups under the `rolling` strategy, paired with `purgeFailed: true`.

Seven days cover the mistake you notice within a week. Four weeks cover silent corruption that nobody caught in time. Twelve months cover audits under standards such as SOC 2, HIPAA, and PCI DSS.

```yaml
lifecycle:
  enabled: false
  strategy: "rolling"
  minKeep: 1
  prompt: true
  purgeFailed: true
  dailyRetention: 7
  weeklyRetention: 4
  monthlyRetention: 12
```

### Calendar policy for an audit

This policy keeps every backup for 14 days, the Friday backup for eight weeks, and the backup from the 15th of the month for six months:

```yaml
lifecycle:
  enabled: false
  strategy: "calendar"
  purgeFailed: false
  dailyRetention: 14
  weeklyRetention: 8
  weeklyDay: 5
  monthlyRetention: 6
  monthlyDay: 15
```


!!! note

    Both examples start with `enabled: false`. Validate the rules 

## Run a rotation

A purge deletes data permanently, so PBM separates the decision from the action. The command uses the same connection options, environment variables, and authentication as every other PBM command.

### Step 1. Validate with a dry run

The `--dry-run` flag overrides the `enabled: false` safety switch. PBM evaluates the rules against your storage and prints a report. Nothing leaves storage:

```bash
pbm lifecycle --dry-run
```

Output:

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

Check that the oldest restore point you depend on appears under the backups to keep.

### Step 2. Turn the feature on

```bash
pbm config --set lifecycle.enabled=true
```

### Step 3. Run the rotation

```bash
pbm lifecycle
```

With `prompt: true`, PBM prints the same report and waits for your answer:

```text
Are you sure you want to permanently delete the purged backups? [y/N]: y
Starting deletion...
Purging backup 2026-03-18T04:02:01Z...
Lifecycle rotation complete.
```

To cancel, answer `N` or press `Ctrl+C`. A `Ctrl+C` during the deletion aborts the remaining purges.

## Separate retention per storage profile

A policy applies at one of two scopes.

| Scope | What it covers | Command |
| --- | --- | --- |
| Global | All backups, with the same rules for every backup type | `pbm lifecycle` |
| Profile | Only the backups routed to that profile | `pbm lifecycle --profile=<PROFILE_NAME>` |

Use the global scope when one set of rules suits all your data. Use profiles when retention lengths differ, or when buckets in separate regions carry separate compliance rules. A common split keeps physical backups for a year and logical backups for three days.

### Step 1. Configure the physical profile

Create `pbm-physical.conf` with the storage settings and the long-term rules:

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

```bash
pbm profile add physical-backup pbm-physical.conf
```

### Step 2. Configure the logical profile

Create `pbm-logical.conf` with the short-term rules. A value of `0` turns off the weekly and monthly tiers:

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

### Step 3. Rotate one profile

Add the `--profile` flag to evaluate or purge a single profile. PBM ignores the global configuration for that run and tags each kept backup with the tier that saved it, such as `[Daily]` or `[Weekly]`:

```bash
pbm lifecycle --profile=physical-backup --dry-run
pbm lifecycle --profile=logical-backup --dry-run
```

## Automate the rotation

A daily `cron` job keeps storage under control without manual work. First turn off the interactive prompt, which waits forever in the background, and confirm the circuit breaker:

```bash
pbm config --set lifecycle.prompt=false
pbm config --set lifecycle.minKeep=1
```

Schedule the run for off-peak hours, away from your backup jobs. The `--out json` flag produces output that a log collector can parse:

```bash
# Run the rotation every day at 3:00 AM
0 3 * * * /usr/bin/pbm lifecycle --out json >> /var/log/pbm-lifecycle-global.log 2>&1
```

Profiles rotate one at a time:

```bash
# Long-term physical backups at 2:00 AM
0 2 * * * /usr/bin/pbm lifecycle --profile=physical-backup --out json >> /var/log/pbm-lifecycle-phys.log 2>&1

# Short-term logical backups at 2:30 AM
30 2 * * * /usr/bin/pbm lifecycle --profile=logical-backup --out json >> /var/log/pbm-lifecycle-logi.log 2>&1
```


!!! warning

    Profiles carry their own settings. Set `prompt: false` inside each profile configuration file as well, not only in the global configuration.


## Safety checks

PBM protects recoverability during a rotation with the following guards.

| Situation | Behavior |
| --- | --- |
| A purge would leave too few backups | The rotation aborts in full when the survivors fall below `minKeep`. |
| A backup is the base for an active PITR chain | PBM refuses the deletion and logs an `ErrBaseForPITR` warning. |
| A backup is in progress | Backups in the `starting`, `running`, or `dumpDone` state stay out of the evaluation. |
| A backup matches several rules at once | PBM flags the backup for each tier and keeps a single copy until the longest window expires. |
| A backup failed or was canceled | `purgeFailed: false` protects the backup without a time limit. `purgeFailed: true` keeps the backup for the length of the daily window, then deletes it. A failed backup never serves as a weekly or monthly anchor. |
| No backup exists on a calendar anchor date | PBM keeps the closest backup from that month. |

An aborted automated run logs the reason. Review your retention settings when this message appears:


!!! WARNING
    This rotation would leave you with 0 backup(s), which is below the safety threshold of 1 (minKeep).
Automated run (prompt: false) detected. Purge aborted to protect your backups.

