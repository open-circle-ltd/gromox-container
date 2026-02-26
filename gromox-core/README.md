# grommunio Docker Image

Docker image for [grommunio](https://grommunio.com/) groupware, based on openSUSE Leap 15.6 with systemd. This image packages the full grommunio stack into a single system container.

## Included Services

- **Gromox** - Mail transport (IMAP, POP3, SMTP, HTTP/MAPI)
- **Grommunio Admin** - Admin API and web interface
- **Grommunio Web** - Webmail client
- **Grommunio Sync** - ActiveSync (mobile device sync)
- **Grommunio DAV** - CalDAV/CardDAV
- **Grommunio Chat** - Messaging (Mattermost-based)
- **Grommunio Antispam** - Spam filtering (rspamd)
- **Grommunio Keycloak** - Identity provider
- **Grommunio Auth** - Authentication
- **Postfix** - MTA for inbound/outbound email
- **Certbot** - Automated TLS certificate management

## Usage

```bash
docker run --runtime=sysbox-runc -it --rm \
  --name=grommunio \
  -v /path/to/var.env:/home/vars/var.env \
  your-registry/grommunio:latest
```

The container uses systemd as its entrypoint and runs two oneshot services on first boot:

1. **db.service** - Initializes the Gromox database schema (or resets it if `CLEAR_DBS=true`)
2. **entry.service** - Configures all grommunio services, SSL, Postfix, and nginx

Both services read their configuration from the environment file mounted at `/home/vars/var.env`.

## Environment Variables

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
| `ENABLE_KEYCLOAK` | Set to `true` to enable Keycloak identity provider. Generates OIDC import config for Keycloak in `/etc/gromox` |
| `KEYCLOAK_REALM` | Keycloak realm name |
| `KEYCLOAK_URL` | URL of the Keycloak server |
| `KEYCLOAK_CLIENT_ID` | OAuth client ID for grommunio in Keycloak |
| `KEYCLOAK_CLIENT_SECRET` | OAuth client secret for grommunio in Keycloak |

## Project Structure

```
.
├── Dockerfile                  # Image definition (openSUSE Leap 15.6 + systemd)
└── gromox-core/
    ├── entrypoint.sh           # Main setup script (SSL, services, Postfix, nginx)
    ├── common/
    │   ├── helpers             # Shared shell functions (setconf, DB config generation)
    │   ├── repo                # Grommunio repository setup
    │   └── ssl_setup           # SSL certificate handling (self-signed, Let's Encrypt)
    ├── config/
    │   ├── autodiscover.ini    # Autodiscover configuration template
    │   ├── certificate.conf    # Nginx SSL certificate config
    │   ├── config.json         # Admin web config template
    │   ├── mysql_adaptor.cfg   # MySQL adaptor config template
    │   └── smtp                # PAM config for SMTP auth
    └── scripts/
        ├── db.sh               # Database initialization script
        ├── db.service          # Systemd unit for DB setup (runs once)
        ├── entry.service       # Systemd unit for entrypoint (runs once, after db.service)
        ├── enable.sh           # Enables the systemd units at build time
        ├── firewall.sh         # Firewall rules (currently disabled)
        └── helpers             # Additional script helpers
```

## Building

```bash
docker build -t grommunio .
```

Build arguments can be used to pin package versions:

```bash
docker build \
  --build-arg GROMOX_VERSION=3.4.0.m43e234e-lp156.6.2 \
  --build-arg GROMMUNIO_ADMIN_API_VERSION=1.17.60.ma2b3a27-lp156.19.1 \
  --build-arg GROMMUNIO_ADMIN_WEB_VERSION=4.0.0.32.fbc7470-lp156.148.1 \
  -t grommunio .
```

## Ports

| Port | Service |
|---|---|
| 25 | SMTP (Postfix) |
| 587 | SMTP Submission (Postfix) |
| 80 | HTTP (nginx / Certbot) |
| 443 | HTTPS (nginx - webmail, admin, sync, DAV) |
| 993 | IMAPS |
| 995 | POP3S |
| 8443 | Grommunio Admin Web UI |

## After Setup

Once the container is running, access the admin interface at:

```
https://<FQDN>:8443
```

Login with username `admin` and the password set via `ADMIN_PASS`.
