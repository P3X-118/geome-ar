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

# Setting up Bar Assistant

This is an [Ansible](https://www.ansible.com/) role which installs [Bar Assistant](https://github.com/karlomikus/bar-assistant/) API server and the web client to run as [Docker](https://www.docker.com/) containers wrapped in systemd services.

Bar Assistant is a service for managing cocktail recipes at your home bar with a lot of cocktail-oriented features like ingredient substitutes.

The role is configured to set up the Bar Assistant's API server and its web client software [Salt Rim](https://github.com/karlomikus/vue-salt-rim).

See the project's [documentation](https://docs.barassistant.app/) to learn what Bar Assistant does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Bar Assistant with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# barassistant                                                         #
#                                                                      #
########################################################################

barassistant_enabled: true

########################################################################
#                                                                      #
# /barassistant                                                        #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Bar Assistant you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
barassistant_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Bar Assistant under a subpath (by configuring the `barassistant_path_prefix` variable) does not seem to be possible due to Bar Assistant's technical limitations.

### Enabling signing up

By default this role is configured to disable signing up for an account on the service. To enable it, add the following configuration to your `vars.yml` file:

```yaml
barassistant_server_environment_variables_allow_registration: true
```

### Connecting to a Meilisearch instance (optional)

To enable the search and filtering functions, you can optionally have the Bar Assistant instance connect to a Meilisearch instance by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the Meilisearch server instance URL
barassistant_saltrim_environment_variables_meilisearch_url: YOUR_MEILISEARCH_INSTANCE_URL_HERE

# Specify a Meilisearch hostname
barassistant_server_environment_variables_meilisearch_host: YOUR_MEILISEARCH_HOSTNAME_HERE

# Specify a Meilisearch API key
barassistant_server_environment_variables_meilisearch_key: YOUR_MEILISEARCH_KEY_HERE
```

You can set the same value to `barassistant_saltrim_environment_variables_meilisearch_url` and `barassistant_server_environment_variables_meilisearch_host` if the Meilisearch is not hosted under a subpath.

>[!NOTE]
>
> - The Meilisearch instance needs to be exposed to the internet.
> - The default Admin API Key is sufficient for using Meilisearch on a Bar Assistant instance. It is [not recommended](https://www.meilisearch.com/docs/learn/security/basic_security) to use the master key for operations anything but managing other API keys.

If you are looking for an Ansible role for Meilisearch, you can check out [ansible-role-meilisearch](https://github.com/mother-of-all-self-hosting/ansible-role-meilisearch) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

### Configuring a Redis database (optional)

You can optionally enable a [Redis](https://redis.io/) database for the Bar Assistant server. [Valkey](https://valkey.io/) can also be used instead.

To enable the Redis database for Bar Assistant server, add the following configuration to your `vars.yml` file:

```yaml
barassistant_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` with your own value.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The role for Valkey ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) is available as well.

### Configuring a SMTP mailer (optional)

You can configure a SMTP mailer to enable email functions such as password recovery.

To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
barassistant_mailer_enabled: true

barassistant_server_environment_variables_mail_mailer: smtp

# Specify SMTP server hostname
barassistant_server_environment_variables_mail_host: ""

# Specify SMTP server port
barassistant_server_environment_variables_mail_port: 587

# Specify SMTP server encryption
# Set `tls` to enable TLS encryption
barassistant_server_environment_variables_mail_encryption: ""

# Specify SMTP server username
barassistant_server_environment_variables_mail_username: ""

# Specify SMTP server password
barassistant_server_environment_variables_mail_password: ""

# Specify the email address that emails will be sent from
barassistant_server_environment_variables_mail_from_address: ""

# Specify the name that emails will be sent from
barassistant_server_environment_variables_mail_from_name: ""
```

See [this page](https://docs.barassistant.app/setup/mailing/) on the official documentation for details.

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Integrating with Prometheus (optional)

Bar Assistant server can natively expose metrics to Prometheus.

If you are looking for an integration, you can check out the MASH playbook. See [this section of the documentation on the playbook](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/services/barassistant.md#integrating-with-prometheus-optional) for more information.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `barassistant_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Bar Assistant's API server becomes available at the specified hostname with the subpath like `https://example.com/api`, and the Salt Rim instance becomes available at `https://example.com`.

To get started, open the Salt Rim's URL with a web browser, and register the account. **Note that the first registered user becomes an administrator automatically.**

Since account registration is disabled by default, you need to enable it first by setting `barassistant_server_environment_variables_allow_registration` to `true` temporarily in order to create your own account.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH, and running `journalctl -fu barassistant-server` (or how you/your playbook named the service, e.g. `mash-barassistant-server`) for the API server and `journalctl -fu barassistant-saltrim` (or how you/your playbook named the service, e.g. `mash-barassistant-saltrim`) for the Salt Rim instance, respectively.
