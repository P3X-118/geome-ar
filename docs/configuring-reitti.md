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

# Setting up Reitti

This is an [Ansible](https://www.ansible.com/) role which installs [Reitti](https://www.dedicatedcode.com/projects/reitti/) and its tile cache server to run as [Docker](https://www.docker.com/) containers wrapped in systemd services.

Reitti is a personal location tracking and analysis application.

See the project's [documentation](https://www.dedicatedcode.com/projects/reitti/) to learn what Reitti does and why it might be useful to you.

## Prerequisites

To run a Reitti instance it is necessary to prepare a [Postgres](https://www.postgresql.org/) database server with [PostGIS](https://postgis.net/) extensions installed and [Redis](https://redis.io/) database for managing cache data.

If you are looking for Ansible roles for them, you can check out [ansible-role-postgis](https://github.com/mother-of-all-self-hosting/ansible-role-postgis) and [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The role for [Valkey](https://valkey.io/) ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) is available as well.

## Adjusting the playbook configuration

To enable Reitti with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# reitti                                                               #
#                                                                      #
########################################################################

reitti_enabled: true

########################################################################
#                                                                      #
# /reitti                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Reitti you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
reitti_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Reitti under a subpath (by configuring the `reitti_path_prefix` variable) does not seem to be possible due to Reitti's technical limitations.

### Set variables for the database server

To have the Reitti instance connect to your Postgres server, add the following configuration to your `vars.yml` file.

```yaml
reitti_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
reitti_database_port: 5432
reitti_database_username: YOUR_POSTGRES_SERVER_USERNAME_HERE
reitti_database_password: YOUR_POSTGRES_SERVER_PASSWORD_HERE
reitti_database_name: YOUR_POSTGRES_SERVER_DATABASE_NAME_HERE
```

Make sure to replace the placeholders with your own values.

### Configure a Redis database

It is necessary to set up a Redis database for the Reitti instance. Valkey can also be used instead.

To enable the Redis database for Reitti, add the following configuration to your `vars.yml` file:

```yaml
reitti_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
reitti_redis_port: 6379
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` with your own value.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `reitti_environment_variables_additional_variables` variable

See the [documentation](https://github.com/dedicatedcode/reitti/blob/main/README.md#environment-variables) for a complete list of Reitti's config options that you could put in `reitti_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Reitti becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and register the account. **Note that the first registered user becomes an administrator automatically.**

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH, and running `journalctl -fu reitti` (or how you/your playbook named the service, e.g. `mash-reitti`) for the Reitti instance and `journalctl -fu reitti-tilecache` (or how you/your playbook named the service, e.g. `mash-reitti-tilecache`) for the tile cache server, respectively.
