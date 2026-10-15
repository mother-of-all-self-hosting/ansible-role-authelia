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

This is an [Ansible](https://www.ansible.com/) role which installs [Authelia](https://www.authelia.com/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Authelia is an open-source [authentication](https://www.authelia.com/overview/authentication/introduction/) and [authorization](https://www.authelia.com/overview/authorization/access-control/) server and portal fulfilling the identity and access management (IAM) role of information security in providing [multi-factor authentication](https://www.authelia.com/overview/authentication/introduction/) and single sign-on (SSO) for your applications via a web portal.

See the project's [documentation](https://www.authelia.com/integration/prologue/get-started/) to learn what Authelia does and why it might be useful to you.

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

### Set random strings

You also need to set random secure strings. To do so, add the following configuration to your `vars.yml` file. The values can be generated with `pwgen -s 64 1` or in another way.

```yaml
authelia_config_jwt_secret: YOUR_SECRET_JWT_KEY_HERE

authelia_config_session_secret: YOUR_SESSION_SECRET_HERE

authelia_config_storage_encryption_key: YOUR_SECRET_KEY_HERE
```

The value of `authelia_config_storage_encryption_key` needs to be at least 20 characters long.

>[!WARNING]
> Changing `authelia_config_storage_encryption_key` subsequently will cause trouble unless you use the CLI to migrate.

### Configuring database

#### Specify database

It is necessary to select database used by DocuSeal from a MySQL compatible database, Postgres, and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
authelia_database_type: postgres
```

Set `mysql` to use a MySQL compatible database and `sqlite` to use SQLite, respectively. The SQLite database is stored in the directory specified with `authelia_data_path`.

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

### Configuring a Redis database (optional)

You can optionally enable a [Redis](https://redis.io/) database for the Authelia instance. [Valkey](https://valkey.io/) can also be used instead.

>[!WARNING]
> If Redis configuration is not specified, session information is stored in memory, and restarting the process will destroy all user sessions.

To enable the Redis database for Authelia, add the following configuration to your `vars.yml` file. Note that the role is by default configured to establish connection with the Redis database via the Unix socket.

```yaml
# Specify the path to the Redis Unix socket path on the host (bind-mount source)
authelia_redis_socket_path_host: ""
```

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to Redis via a Unix socket
authelia_redis_socket_enabled: false

authelia_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` with your own value.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The role for Valkey ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) is available as well.

### Configuring access control (optional)

You can configure [access control](https://www.authelia.com/configuration/security/access-control/), a list of rules defining the authorizations applied for one resource to users or group of users, by adding the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
authelia_config_access_control_rules:
 - domain: 'service1.example.com'
   policy: one_factor
```

Refer to [`templates/configuration.yml.j2`](../templates/configuration.yml.j2) for details about how to set it up.

### Configuring operation modes

Authelia can be used in two different modes, which can be enabled simultaneously:

- **Forward-Auth**: Forward-Auth can act as a replacement for [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication). It does this by acting as a companion to [common reverse proxies](https://www.authelia.com/overview/prologue/supported-proxies/) like Traefik.

- **OpenID Connect**: experimental OpenID Connect support support, so that services which are OpenID Connect-compatible can use Authelia as an identity provider.

#### Protecting a service with Authelia's forward-auth

If Traefik runs as a reverse proxy for Authelia, you can easily protect services running on the same host by adding additional labels to them to configure [Forward-Auth](https://doc.traefik.io/traefik/middlewares/http/forwardauth/). Forward-Auth is useful for protecting services which are not aware of authentication at all or which can receive authentication/authorization data via [HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers).

Here's an example configuration for protecting a static website ([Hubsite](https://github.com/moan0s/hubsite) installed with [ansible-role-hubsite](https://github.com/mother-of-all-self-hosting/ansible-role-hubsite)) service:

```yaml
hubsite_container_labels_additional_labels_custom:
  - traefik.http.routers.{{ hubsite_identifier }}.middlewares={{ authelia_identifier }}@docker
```

The Hubsite component does not use any Traefik middlewares, so defining a `.middlewares` configuration key and pointing it to the Authelia middleware works well.

For most other components, middlewares are in use in their default Traefik labels, so adding an additional `.middlewares` key will not work. You may need to inject additional middlewares on top of the default ones.

#### Protecting a service with OpenID Connect

For services which support OpenID Connect, you can enable the [experimental OpenID Connect identity provider](https://www.authelia.com/configuration/identity-providers/open-id-connect/) in Authelia.

Here's an example configuration for protecting [Grafana](https://grafana.com/) with a single OpenID Connect client (application) called `grafana`:

```yaml
authelia_config_identity_providers_oidc_clients:
  - id: grafana
    description: Grafana
    secret: HASH_OF_THE_SHARED_SECRET
    public: false
    authorization_policy: one_factor
    redirect_uris:
      - https://example.com/grafana/login/generic_oauth
    scopes:
      - openid
      - profile
      - groups
      - email
    userinfo_signing_algorithm: none

authelia_config_identity_providers_oidc_issuer_private_key: |
  KEY CONTENT GOES HERE.
  TO GENERATE A KEY, USE: `openssl genpkey -algorithm RSA -out FILE_NAME`
```

You will need to create a shared secret and hash its value (e.g. `php -r 'echo password_hash("PASSWORD_HERE",  PASSWORD_ARGON2ID);'`). Feel free to use another language (or tool) for creating a hash as well. A few different hash algorithms are supported besides Argon2id.

Finally, configure your application, hooking it to Authelia's OpenID Connect identity provider. Refer to the [sample configuration](https://github.com/mother-of-all-self-hosting/ansible-role-grafana/blob/main/docs/configuring-grafana.md#authelia) for details.

### Configuring authentication storage providers

Authelia supports [file-based](https://www.authelia.com/configuration/first-factor/file/) and [LDAP](https://www.authelia.com/configuration/first-factor/ldap/) storage providers for the user database.

To enable the file-based provider, add the following configuration to your `vars.yml` file (adapt to your needs):

```yaml
authelia_config_authentication_backend_file_content: |
  ---
  users:
    john:
      disabled: false
      displayname: John Doe
      password: PASSWORD_HASH_HERE
      email: john@example.com
      groups:
        - admins
        - dev
    peter:
      disabled: false
      displayname: Peter Johnson
      password: PASSWORD_HASH_HERE
      email: peter@example.com
      groups:
        - dev
  ...
```

To use LDAP, define `authelia_config_authentication_backend_ldap_*` variables instead.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `authelia_environment_variables_additional_variables` variable

Refer to [this page](https://www.authelia.com/configuration/) on the official documentation for a complete list of Authelia's config options that you can put in `authelia_configuration_extension_yaml`.

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

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
authelia_config_log_level: debug
```
