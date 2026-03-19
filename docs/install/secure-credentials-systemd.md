# Secure credential management with systemd

## Overview

Percona Backup for MongoDB (PBM) requires access to sensitive credentials such as:

- **MongoDB connection URI** (`PBM_MONGODB_URI`)

- **Object storage credentials** defined in PBM configuration (pbm config)

By default, these credentials are often stored in plaintext in environment variables or configuration files. This introduces security risks such as credential leakage and unauthorized access.

This section describes how to securely manage PBM credentials using **systemd service credentials**.

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

- systemd version 250 or higher
- Root or sudo privileges
- (Optional) TPM2 support for hardware-backed encryption
- Kernel 5.4+


## Procedure

Here are the steps to integrate PBM with systemd's [System and service credentials :octicons-link-external-16:](https://systemd.io/CREDENTIALS/)
{.power-number}

1. Create a **temporary file** containing your connection string:

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

4. Edit the systemd unit file (default location at `/lib/systemd/system/pbm-agent.service)` and in the `[Service]` section, add the `LoadCredentialEncrypted` and `PrivateMounts directives:`

    ```
    [Service]
    LoadCredentialEncrypted=pbm_connection.yaml:/path/to/pbm_connection.yaml.cred
    PrivateMounts=yes
    ExecStart=/usr/bin/pbm-agent -f /run/credentials/%n/pbm_connection.yaml
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
        
        - The path to the decrypted plaintext is stored in the $CREDENTIALS_DIRECTORY environment variable.
        - In the example above, the PBM agent will read the contents of the file at `%d/pbm_connection.yaml` as its connection string.


## How to verify?

Run the following command to ensure the service can see the credential. This file is only accessible while the service is running and is stored in a secure, temporary directory.

```sh
sudo cat /run/credentials/pbm-agent.service/pbm_connection.yaml
```

