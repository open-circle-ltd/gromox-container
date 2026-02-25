# Grommunio Containers (Open Circle Fork)

Forked from [grommunio/gromox-container](https://github.com/grommunio/gromox-container) by [Open Circle](https://github.com/open-circle-ltd).

## Scope

This fork only actively develops and maintains the **gromox-core** and **gromox-archive** containers. Other containers (e.g. gromox-office) are kept as-is from upstream.

## Changes from Upstream

- **No firewalld**: We do not use firewalld inside the containers. Port exposure is handled entirely via Docker port mappings in `docker-compose.yml`.
- **Let's Encrypt (Certbot)**: Fixed certificate generation to support multiple domains (FQDN + autodiscover + custom mail domains), added certificate reuse logic to skip Certbot when a valid certificate already exists, and corrected the admin email address used for registration
- **Compose file (Volumes)**: Fixed volume mounts and removed firewalld dependency
- **Keycloak Support**: Fixed Keycloak Configuration
- 
## About

The container uses an [OpenSuse Leap 15.6 base](https://hub.docker.com/r/opensuse/leap) and [Sysbox](https://github.com/nestybox/sysbox) enabled for PID 1 Init capabilities.

*This is an incredibly complex piece of software that tries to get you up and running with sane defaults, you will need to switch eventually over to manually configuring the configuration file when depending on your usage case.*

**Do not use our defaults in production environment! Change them**

## Maintainer

- [Open Circle](https://github.com/open-circle-ltd) (fork)
- [Grommunio Team](https://github.com/grommunio) (upstream)

## Table of Contents

- [Quick Start](#quick-start)
- [Configuration](#configuration)
    - [Persistent Volumes](#persistent-volumes)
    - [Environment Variables](#environment-variables)

## Quick Start

* The quickest way to get started is using [docker-compose](https://docs.docker.com/compose/).
The docker compose file can be modified (and **should be**) for development or production use.

* Set various [environment variables](#environment-variables) to understand the capabilities of this image.

## Configuration

## Environment Variables

Please edit the variables in the `var.env` file to suit your deployment.

### Required

| Variable | Description |
|---|---|
| `FQDN` | Fully qualified domain name of the server (e.g. `mail.example.com`) |
| `DOMAIN` | Mail domain (e.g. `example.com`) |
| `ADMIN_PASS` | Password for the grommunio admin user |
| `MYSQL_HOST` | Hostname of the MySQL/MariaDB server |
| `MYSQL_USER` | Database user for gromox |
| `MYSQL_PASS` | Database password for gromox |
| `MYSQL_DB` | Database name for gromox |

### Chat

| Variable | Description |
|---|---|
| `CHAT_MYSQL_HOST` | MySQL host for grommunio-chat |
| `CHAT_MYSQL_USER` | MySQL user for grommunio-chat |
| `CHAT_MYSQL_PASS` | MySQL password for grommunio-chat |
| `CHAT_MYSQL_DB` | Database name for grommunio-chat |
| `CHAT_ADMIN_PASS` | Admin password for grommunio-chat |
| `CHAT_CONFIG` | Path to the chat config file |

### SSL / TLS

| Variable | Description |
|---|---|
| `SSL_INSTALL_TYPE` | `0` = self-signed certificate, `2` = Let's Encrypt |
| `SSL_BUNDLE` | Path to custom TLS certificate bundle (if providing your own) |
| `SSL_KEY` | Path to custom TLS private key (if providing your own) |

### Optional

| Variable | Description |
|---|---|
| `ORGANIZATION` | Organization name for autodiscover |
| `TIMEZONE` | Timezone for autodiscover (e.g. `Europe/Zurich`) |
| `RELAYHOST` | SMTP relay host for Postfix |
| `LOGFILE` | Path to the setup log file |
| `CLEAR_DBS` | Set to `true` to drop and recreate the gromox database on startup |
| `MYSQL_ROOT_PASS` | MySQL root password (required when `CLEAR_DBS=true`) |
| `ENABLE_FILES` | Set to `true` to enable grommunio Files proxy |
| `ENABLE_OFFICE` | Set to `true` to enable grommunio Office proxy |
| `ENABLE_ARCHIVE` | Set to `true` to enable grommunio Archive proxy |
| `OFFICE_HOST` | Hostname of the grommunio Office/Files server |
| `ARCHIVE_HOST` | Hostname of the grommunio Archive server |

### Keycloak

| Variable | Description |
|---|---|
| `ENABLE_KEYCLOAK` | Set to `true` to enable Keycloak integration |
| `KEYCLOAK_REALM` | Keycloak realm name |
| `KEYCLOAK_URL` | Keycloak auth server URL (e.g. `https://keycloak.example.com/auth/`) — must end with `/` |
| `KEYCLOAK_CLIENT_ID` | OAuth2 client ID (resource) |
| `KEYCLOAK_CLIENT_SECRET` | OAuth2 client secret |

When enabled, the entrypoint will:
- Fetch the RS256 public key from the Keycloak JWKS endpoint and write it to `/etc/gromox/bearer_pubkey`
- Generate `/etc/grommunio-common/keycloak.json` with the OIDC client config
- Generate `/etc/gromox/oidc-import.json` — a Keycloak client import template that can be imported into the Keycloak realm

## Caveats
* If you are testing without a valid domain, please ensure you perform the action below otherwise your file service will be unreachable.
  ```
  docker exec -it <your office container name> bash
  cd /usr/share/grommunio-files
  sudo -u grofiles ./occ -q -n config:system:set trusted_domains 3 --value="<YOUR SERVER IP>"
  ```

* You need to run the pre-launch script `pre-launch.sh` to set up the docker volume and pre-populate it
  * The certificates are shared in the volume created by the `gromox-core` container. Please ensure you have the `gromox-core` container running before the other containers.

* Grommunio-files will fail with self-signed certificates. Use Letsencrypt certificates instead.
