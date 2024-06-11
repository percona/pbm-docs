# Troubleshoot backup management via Percona Monitoring and Management

If you are facing issues with physical backups and restores via PMM, follow these troubleshooting steps:

1. Ensure that the user running the `pmm` process has sufficient permissions within PBM. You can find detailed instructions in the [PMM documentation :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/setting-up/client/mongodb.html#create-pmm-account-and-set-permissions).
2. Make sure you have specified the user from step 1 in the MongoDB connection URI string for `pbm-agent` processes.
3. Confirm that the system user who runs the `pbm-agent` process (by default, `mongod`) has the read / write access to both the MongoDB dbpath and the backup storage location.
4. Examine the `systemd` restart policy for the user mentioned in step 4. Ensure it is not set to `always` or `on-success`. During physical restores, the database must not be automatically restarted as this is controlled by the `pbm-agent`.

