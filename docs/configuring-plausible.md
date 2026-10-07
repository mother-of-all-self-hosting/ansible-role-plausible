<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Plausible Analytics

This is an [Ansible](https://www.ansible.com/) role which installs [Plausible Analytics](https://plausible.io/) ([Community Edition](https://plausible.io/blog/community-edition)) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Plausible Analytics is intuitive, lightweight and open-source web analytics. No cookies and fully compliant with GDPR, CCPA and PECR.

See the project's [documentation](https://plausible.io/docs) to learn what Plausible Analytics does and why it might be useful to you.

## Prerequisites

To run a Plausible Analytics instance it is necessary to prepare [ClickHouse](https://clickhouse.com/) and [Postgres](https://www.postgresql.org/) database servers.

If you are looking for an Ansible role for them, you can check out [ansible-role-clickhouse](https://github.com/mother-of-all-self-hosting/ansible-role-clickhouse) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Plausible Analytics with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# plausible                                                            #
#                                                                      #
########################################################################

plausible_enabled: true

########################################################################
#                                                                      #
# /plausible                                                           #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Plausible Analytics you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
plausible_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Plausible Analytics under a subpath (by configuring the `plausible_path_prefix` variable) does not seem to be possible due to Plausible Analytics's technical limitations.

### Set variables for the database servers

To have the Plausible Analytics instance connect to your ClickHouse and Postgres servers, add the following configuration to your `vars.yml` file.

```yaml
plausible_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
plausible_database_port: 5432
plausible_database_username: YOUR_POSTGRES_SERVER_USERNAME_HERE
plausible_database_password: YOUR_POSTGRES_SERVER_PASSWORD_HERE
plausible_database_name: YOUR_POSTGRES_SERVER_DATABASE_NAME_HERE

plausible_clickhouse_database_hostname: YOUR_CLICKHOUSE_SERVER_HOSTNAME_HERE
plausible_clickhouse_database_port: 8123
plausible_clickhouse_database_username: YOUR_CLICKHOUSE_SERVER_USERNAME_HERE
plausible_clickhouse_database_password: YOUR_CLICKHOUSE_SERVER_PASSWORD_HERE
plausible_clickhouse_database_name: YOUR_CLICKHOUSE_SERVER_DATABASE_NAME_HERE
```

Make sure to replace the placeholders with your own values.

>[!NOTE]
> Plausible also requires the following grants on ClickHouse:
>
> 1. `GRANT SELECT ON system.replicas TO plausible;`
> 2. `GRANT SELECT ON system.parts TO plausible;`

### Set random strings

You also need to set random secure strings. To do so, add the following configuration to your `vars.yml` file:

```yaml
# Generate this with: `openssl rand -base64 48`
plausible_environment_variable_secret_key_base: YOUR_SECRET_KEY_HERE

# Generate this with: `openssl rand -base64 32`
plausible_environment_variable_totp_vault_key: YOUR_SECRET_KEY_FOR_TOTP_VAULT_HERE
```

### Setting user IDs for administrators (optional)

It is possible to specify which user IDs will be system admins by adding the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
plausible_environment_variable_admin_user_ids: '1,2,3'
```

By default, only the first user (`1`) to be registered will be made an administrator.

### Configuring the mailer (optional)

You can configure a mailer for activating accounts, resetting password, and sending reports. For the mailer you can use a SMTP server or services like Mailgun and Postmark.

To configure the SMTP mailer, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
plausible_environment_variable_mailer_adapter: Bamboo.SMTPAdapter

# Specify SMTP server hostname
plausible_environment_variable_smtp_host_addr: ""

# Specify SMTP server port
plausible_environment_variable_smtp_host_port: 587

# Specify SMTP server username
plausible_environment_variable_smtp_user_name: ""

# Specify SMTP server password
plausible_environment_variable_smtp_user_pwd: ""

# Specify the email address that emails will be sent from
plausible_environment_variable_mailer_email: ""

# Set to `true` to enable SMTPS
plausible_environment_variable_smtp_host_ssl_enabled: ""
```

>[!NOTE]
> As of 2024-06-28, only `Bamboo.SMTPAdapter` behaves well when no SMTP username/password AUTH is required (as is the case for exim-relay). The Bamboo.Mua SMTP adapter is more modern, but always sends authentication, even when the SMTP user is empty.

Refer to [this section](https://github.com/plausible/community-edition/wiki/configuration#email) on the official documentation for details about how to configure the mailer.

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `plausible_environment_variables_additional_variables` variable

Refer to [this page](https://github.com/plausible/community-edition/wiki/configuration) on the official documentation for a complete list of Plausible Analytics's config options that you can put in `plausible_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Plausible Analytics becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and register the administrator account (see the details about `plausible_environment_variable_admin_user_ids` above).

After logging in with your user account you can create properties (websites) and invite other users by email. By default, the service is configured to allow registrations that are coming from an explicit invitation, while public registrations are disabled. This can be controlled with the `plausible_environment_variable_disable_registration` variable.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu plausible` (or how you/your playbook named the service, e.g. `mash-plausible`).
