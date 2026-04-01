# Secure credential management with systemd

## Overview

This guide describes how to securely store the **MongoDB connection URI** (`mongodb-uri`) for `pbm-agent` using **systemd service credentials**. The same technique can be applied to other sensitive PBM credentials, such as object storage credentials in the PBM configuration.

By default, credentials are often stored in plaintext in environment variables or configuration files. This introduces security risks such as credential leakage and unauthorized access.

## Why use systemd credentials?

Storing credentials in plaintext significantly increases the risk of compromise. Secrets placed in configuration files or environment variables can be exposed through:

- File access
- Process inspection (e.g., `ps`, `/proc`)
- Unauthorized system access.

**`systemd` credentials** mitigate these risks by:

- Encrypting credentials at rest
- Decrypting them only at service runtime
- Storing them in a temporary, non-swappable directory
- Restricting access to the service itself


## Prerequisites

- systemd version 250 or higher. 

    To check the installed `systemd` version on your system, run:

    ```bash
    systemctl --version
    ```

    ??? example "Output"

        ```sh
        systemd 252 (252.5-2ubuntu3)
        +PAM +AUDIT +SELINUX ...
        ```

      The following operating systems meet this requirement:

        - RHEL/OL/Rocky Linux 9
        - Ubuntu 24.04
        - Debian 12
        - Amazon Linux 2023

- Root or sudo privileges
- (Optional) TPM2 support for hardware-backed encryption
- Kernel 5.4+


## Procedure

Here are the steps to integrate PBM with systemd's [System and service credentials :octicons-link-external-16:](https://systemd.io/CREDENTIALS/)
{.power-number}

1. Create a **temporary PBM agent YAML config file** containing the `mongodb-uri` key with your connection string:

    ```sh
    echo "mongodb-uri: mongodb://pbmuser:secretpwd@localhost:27017/?authSource=admin" > pbm_uri.yaml
    ```

2. Encrypt the credential using the local system key (and Trusted Platform Module version 2.0 if available):

    ```sh
    systemd-creds encrypt --name=pbm_connection.yaml pbm_uri.yaml pbm_connection.yaml.cred
    ```

3. Securely delete the plain text file:

    ```sh
    shred -u pbm_uri.yaml
    ```

4. Edit the systemd unit file (for example, `/lib/systemd/system/pbm-agent.service` on Debian/Ubuntu or `/usr/lib/systemd/system/pbm-agent.service` on RHEL-based distributions). In the `[Service]` section, add the `LoadCredentialEncrypted` and `PrivateMounts` directives, and update `ExecStart` to pass the decrypted credential file to `pbm-agent`:

    ```
    [Service]
    LoadCredentialEncrypted=pbm_connection.yaml:/path/to/pbm_connection.yaml.cred
    PrivateMounts=yes
    ExecStart=/usr/bin/pbm-agent -f %d/pbm_connection.yaml
    ```

5. Reload the systemd manager configuration:

    ```sh
    sudo systemctl daemon-reload
    ```

6. Restart the PBM agent:

    ```sh
    sudo systemctl restart pbm-agent
    ```

    ??? info "What happens under the hood"
        Systemd automatically decrypts the credential during service startup and places it in a temporary, non-swappable directory.

        - The path to the decrypted plaintext is stored in the `$CREDENTIALS_DIRECTORY` environment variable.
        - In the example above, the PBM agent will read the YAML configuration file at `%d/pbm_connection.yaml`, which contains the `mongodb-uri` setting with its connection string.


## How to verify?

Run the following command to ensure the service can see the credential. This file is only accessible while the service is running and is stored in a secure, temporary directory.

```sh
sudo cat /run/credentials/pbm-agent.service/pbm_connection.yaml
```

