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
SPDX-FileCopyrightText: 2023 Alejandro AR
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 MASH project contributors
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Sergio Durigan Junior
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Authelia

This is an [Ansible](https://www.ansible.com/) role which installs [Authelia](https://authelia.org) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Authelia is a self-hosted lightweight software forge (Git hosting service, etc.), an alternative to [Gitea](https://gitea.io/).

See the project's [documentation](https://authelia.org/docs/latest/) to learn what Authelia does and why it might be useful to you.

## Prerequisites

To run a Authelia instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). By default it is configured to use SQLite.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Authelia with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# authelia                                                             #
#                                                                      #
########################################################################

authelia_enabled: true

########################################################################
#                                                                      #
# /authelia                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Authelia you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `authelia.example.com` with your own value.

```yaml
authelia_hostname: "authelia.example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

### Set the base domain for session cookies

You also need to specify the base domain that session cookies related to authentication will be set on as below:

```yaml
authelia_config_session_domain: example.com
```

>[!NOTE]
> This needs to be a subdomain of `authelia_hostname`.

Forward-auth services that you will protect need to be hosted on subdomains under this base domain (e.g. service1.example.com, service2.example.com). For OpenID Connect-protected services, this value does not matter.

### Configuring database

#### Specify database (optional)

You can specify a database used by Authelia. By default it is configured to use SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
authelia_database_type: postgres
```

Set `mysql` to use a MySQL compatible database.

For other settings, check variables such as `authelia_database_mysql_*` and `authelia_database_postgres_*` on [`defaults/main.yml`](../defaults/main.yml).

#### Configuring connection to the database server (optional)

By default the role is configured to establish the connection to the database server via a Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the MySQL compatible server's Unix socket path on the host (bind-mount source)
authelia_database_mysql_socket_path_host: ""

# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
authelia_database_postgres_socket_path_host: ""
```

Setting it enables to connect to the database server via Unix socket mounted in the container.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to the MySQL compatible server via a Unix socket
authelia_database_mysql_socket_enabled: false

# Disable the connection to the Postgres server via a Unix socket
authelia_database_postgres_socket_enabled: false
```

### Configuring cache (optional)

```yaml
authelia_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
authelia_redis_port: 6379
authelia_redis_password: YOUR_REDIS_SERVER_PASSWORD_HERE
authelia_redis_database: 0
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` and `YOUR_REDIS_SERVER_PASSWORD_HERE` with your own values.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The role for Valkey ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) is available as well.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `authelia_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the Authelia instance becomes available at the URL specified with `authelia_hostname`. With the configuration above, the service is hosted at `https://authelia.example.com`.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu authelia` (or how you/your playbook named the service, e.g. `mash-authelia`).
