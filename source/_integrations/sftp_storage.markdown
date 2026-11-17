---
description: Allows storing backups created with Backup system onto remote SFTP host.
ha_release: '2025.10'
ha_domain: sftp_storage
related:
  - docs: /common-tasks/general/backups/
    title: Backups
  - docs: /more-info/backup-emergency-kit/
    title: Backup emergency kit
---

The **SFTP Storage** {% term integration %} works with the core [Backup](/integrations/backup) integration. When you enable **SFTP Storage** in the **Locations** section of your [Backup](/integrations/backup) integration settings, Home Assistant automatically stores manual and automatic backups on your remote SSH/SFTP server.

To learn how to create and restore a backup, refer to the backup section under [common tasks](/common-tasks/general/backups/).

{% include integrations/config_flow.md %}

{% configuration_basic %}
Host:
  description: Hostname or IP address of SSH/SFTP Server to connect to.
  required: true
  type: string
Port:
  description: Port of your SSH/SFTP Server. This is usually 22.
  required: true
  type: integer
  default: 22
Username:
  description: Username to authenticate with.
  required: true
  type: string
Password:
  description: Password to authenticate with. Provide this or `Private Key File`.
  required: true
  type: string
Private Key File:
  description: Upload a private key file used for authentication. The key must not be protected by a passphrase. Provide this or `Password`.
  required: true
  type: selector
Remote path:
  description: Remote path where to upload backups. Directory must already exist and user provided in `Username` must have write access to it.
  required: true
  type: string
{% endconfiguration_basic %}

If both `Password` and `Private Key File` are provided, service will try to log in with private key first, then fallback to password-based authentication if private key authentication fails.

## Reconfiguration

You can change the login without removing the integration. The host, port and remote path can't be changed, because they identify where your backups are stored. To use a different server or folder, add the integration again.

1. Go to {% my integrations title="**Settings** > **Devices & services**" %} and select the **SFTP Storage** integration.
2. Next to the entry, select the three dots {% icon "mdi:dots-vertical" %} menu, then select **Reconfigure**.
3. Enter the **Username**, and a new **Password** or **Private Key File**. To keep the current password and private key, leave both empty.
4. Select **Submit**.

A new password or private key replaces both of the old ones. For example, if you enter only a password, the private key file the integration stored is removed.

## Troubleshooting

If no tip here resolves your issue, [open a bug](https://github.com/home-assistant/core/issues/new?template=bug_report.yml&integration_name=SFTP%20Storage&integration_link=https%3A%2F%2Fwww.home-assistant.io%2Fintegrations%2Fsftp_storage) and include the [debug logs](/docs/configuration/troubleshooting/#debug-logs-and-diagnostics).

### The SSH server rejects the login

#### Symptom: "Authentication failed for _username_@_host_"

Backups can't be uploaded or listed, and the integration shows this error.

#### Description

The SSH server refused the username, password, or private key. This usually happens after the password was changed on the server, or the public key was removed from the user's `authorized_keys` file.

#### Resolution

To resolve this issue, try the following steps:

1. From another computer, check that you can log in with the same username and password or private key, for example with `sftp -P 22 username@host`.
2. If the password or key has changed, [reconfigure](#reconfiguration) the integration with the new one.
3. If you use a private key, make sure it is not protected by a passphrase.

### Home Assistant can't reach the SSH server

#### Symptom: "Cannot connect to the SSH server _host_"

The integration doesn't load, or backups to this location fail.

#### Description

Home Assistant can't open a connection to the server. The host or port may be wrong, the server may be off or asleep, or a firewall may block the connection. If this happens while Home Assistant starts, it keeps retrying in the background until the server can be reached.

#### Resolution

To resolve this issue, try the following steps:

1. Check that the server is on, and that its address hasn't changed.
2. Check that SSH is running on the port you entered. This is usually 22.
3. Check that no firewall blocks connections from Home Assistant to that port.

### The backup folder can't be opened

#### Symptom: "Cannot open the backup location _path_ on _host_"

#### Description

Home Assistant logged in, but can't open the **Remote path**. The folder may have been removed or renamed, or the user may not have access to it.

#### Resolution

To resolve this issue, try the following steps:

1. Make sure the folder exists on the server, at exactly the path you entered.
2. Make sure the user can read and write in that folder.
3. If the backups have moved to another folder, add the integration again with the new path, and then remove the old entry.

### The server doesn't allow SFTP

#### Symptom: "Cannot open an SFTP session on _host_"

#### Description

The server accepted the login but didn't start SFTP. SFTP may be turned off on the server, or the account may only be allowed a shell or another file protocol.

#### Resolution

To resolve this issue, try the following steps:

1. Make sure SFTP is turned on. On a NAS, this is usually a setting next to SSH. On a Linux server, check that `sshd_config` has a `Subsystem sftp` line.
2. Make sure the user is allowed to use SFTP.

## Removing the integration

This integration follows standard integration removal. No extra steps are required.

{% include integrations/remove_device_service.md %}

- After you remove the integration, all Home Assistant backups stored on the remote server are not automatically deleted. You need to manually delete them from the remote filesystem.
