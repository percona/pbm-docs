# View detailed information about a backup

To view a detailed information about a backup, run the following command:

```{.bash data-prompt="$"}
$ pbm describe-backup <backup-name>
```

The output provides the backup name, type, status, size and the information about the cluster topology it was taken in. For [selective backups](../features/selective-backup.md), it also shows the namespaces that were backed up:

Output:

```{.text .no-copy}
name: "2022-08-17T10:49:03Z"
type: logical
last_write_ts: 1662039300,2
last_transition_ts: "1662039304"
namespaces:
- Invoices.*
mongodb_version: 5.0.10-9
pbm_version: 2.0.0
status: done
size: 10234670
error: ""
replsets:
- name: rs1
  status: done
  iscs: false
  last_write_ts: 1662039300,2
  last_transition_ts: "1662039304"
  error: ""
```