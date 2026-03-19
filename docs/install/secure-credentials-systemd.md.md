# Secure credential management with systemd

## Overview

Percona Backup for MongoDB (PBM) requires access to sensitive credentials such as:

- MongoDB connection URI (PBM_MONGODB_URI)

- Object storage credentials defined in PBM configuration (pbm config)

By default, these credentials are often stored in plaintext in environment variables or configuration files. This introduces security risks such as credential leakage and unauthorized access.

This section describes how to securely manage PBM credentials using systemd service credentials.

## Why use systemd credentails

Storing credentials in plaintext significantly increases the risk of compromise. Secrets placed in configuration files or environment variables can be exposed through file access, process inspection, or unauthorized system access.

`systemd` credentials mitigate these risks by enforcing strict runtime security controls.
