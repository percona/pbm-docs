# Download Percona Backup for MongoDB from Percona website

You can download Percona Backup for MongoDB from [Percona website :octicons-link-external-16:](https://www.percona.com/downloads/percona-backup-mongodb/) and install it:

* [From binary tarballs](#install-from-binary-tarballs).
* Manually, from the installation packages using `dpkg` (Debian and Ubuntu) or `rpm` (Red Hat Enterprise Linux and CentOS). However, you must make sure that all dependencies are satisfied.

--8<-- "pbm-install-nodes.md"

## Before you start

Check the [system requirements](../system-requirements.md) and [supported MongoDB versions](../details/versions.md).

## Install from binary tarballs

Find the link to the binary tarballs under the **Generic Linux** menu item on [Percona website :octicons-link-external-16:](https://www.percona.com/downloads/percona-backup-mongodb/).
{.power-number}

1. Fetch the binary tarball. Replace the version in the URL with your required version.

    ```bash
    wget https://downloads.percona.com/downloads/percona-backup-mongodb/percona-backup-mongodb-{{release}}/binary/tarball/percona-backup-mongodb-{{release}}-x86_64.tar.gz
    ```

2. Extract the tarball

    ```bash
    tar -xf percona-backup-mongodb-{{release}}-x86_64.tar.gz
    ```

3. Export the location of the binaries to the `PATH` variable

    For example, if you’ve extracted the tarball to your `home` directory, the command would be the following:

    ```bash
    export PATH=~/percona-backup-mongodb-{{release}}/:$PATH
    ```

--8<-- "install-result.md"

## Next steps

[Initial setup :material-arrow-right:](initial-setup.md){.md-button}