# Grommunio Containers (Open Circle Fork)

Forked from [grommunio/gromox-container](https://github.com/grommunio/gromox-container) by [Open Circle](https://github.com/open-circle-ltd).

## Scope

This fork only actively develops and maintains the **gromox-core** and **gromox-archive** containers. Other containers (e.g. gromox-office) are kept as-is from upstream.

## Changes from Upstream

- **No firewalld**: We do not use firewalld inside the containers. Port exposure is handled entirely via Docker port mappings in `docker-compose.yml`.
- **Let's Encrypt (Certbot)**: Fixed certificate generation to support multiple domains (FQDN + autodiscover + custom mail domains), added certificate reuse logic to skip Certbot when a valid certificate already exists, and corrected the admin email address used for registration
- **Compose file (Volumes)**: Fixed volume mounts and removed firewalld dependency

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

### Environment Variables

Please edit the variables in the `var.env` file to suit your deployment.

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
