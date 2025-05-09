---
title: Self-hosted website analytics - Umami
date: 2025-05-09 04:56:50
tags: [website, analytics, monitoring, umami, podman, degoogle, privacy, open-source, self-hosted]
---

When I was researching for open-source and privacy-focused website analytics, as an alternative to Google Analytics, I ended up choosing [Umami](https://github.com/umami-software/umami) to deliver me some valuable insights into my website's performance. It is easy enough to set up and doesn't have unnecessary clutter.

The original docker-compose.yaml file is available in [Umami's Github repository](https://github.com/umami-software/umami/blob/master/docker-compose.yml).

> The compose file is modified to work in the Debian LXC container with Podman instead of Docker. Although Podman does support internal DNS, in the LXC setup, this does not work and as a workaround the IP addresses have been set manually. **This setup will work with Docker as well.**
{: .prompt-info }

## Files

### podman-compose.yaml

```yaml
services:
  umami:
    image: ghcr.io/umami-software/umami:${UMAMI_VERSION}
    container_name: umami
    restart: always
    ports:
      - 3000:3000
    environment:
      DATABASE_URL: postgresql://${PG_USER}:${PG_PASSWORD}@${DB_IP}:5432/${PG_DB}
      DATABASE_TYPE: postgresql
      APP_SECRET: ${SECRET}
      DISABLE_TELEMETRY: 1
    depends_on:
      postgres:
        condition: service_healthy
    init: true
    healthcheck:
      test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      umami:
        ipv4_address: ${UMAMI_IP}

  postgres:
    image: postgres:${PG_VERSION}
    container_name: postgres
    restart: always
    environment:
      POSTGRES_DB: ${PG_DB}
      POSTGRES_USER: ${PG_USER}
      POSTGRES_PASSWORD: ${PG_PASSWORD}
    volumes:
      - umami-db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      umami:
        ipv4_address: ${DB_IP}

volumes:
  umami-db-data:

networks:
  umami:
    driver: bridge
    ipam:
      config:
        - subnet: ${SUBNET}

```

### .env

> `UMAMI_VERSION` can be replaced with any specific version to have control over the version upgrades. E.g. `postgresql-v2.18.0`.
{: .prompt-tip }

```conf
# Network
SUBNET=172.19.0.0/24
UMAMI_IP=172.19.0.5
DB_IP=172.19.0.6

# Database setup
PG_DB=umami
PG_USER=umami
PG_PASSWORD=<replace_with_your_password>

# App secret
SECRET=<replace_with_your_secret>

# Version control
PG_VERSION=15-alpine
UMAMI_VERSION=latest
```

## Tips & Tricks

### Default login

Username: `admin` \
Password: `umami`

### Delete unwanted sessions

While Umami does not have a built-in way of deleting individual sessions, there is a workaround and they can be deleted manually from the database. 

> Although I have deleted my own sessions directly from the database without noticing any complications, it is advised to back up your data and proceed with caution.
{: .prompt-warning }

Enter the database container. This should be run as `sudo`, however in my example, I am already a `root` user in an unprivileged LXC container.

> For Docker, replace `podman` with `docker`.
{: .prompt-tip }

```shell
# Podman
podman exec -it postgres bash 
```

Access the database:

```shell
psql -U umami -d umami
```

Delete the session records:

> The `session_id` can be found in the Umami web interface under Sessions by clicking on a specific session.
{: .prompt-tip }

```sql
DELETE FROM session WHERE session_id = '45f57d86-f452-4d2d-8e0f-b7b7165e2f70';
DELETE FROM website_event WHERE session_id = '45f57d86-f452-4d2d-8e0f-b7b7165e2f70';
```

**_Voilà!_**