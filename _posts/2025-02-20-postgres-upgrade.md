---
title: Upgrading PostgreSQL 14 to 17
date: 2025-02-20 07:17:50
tags: [postgres, database, upgrade]
---

> Upgrading from `postgres:14.2-alpine` based database to `postgres:17-alpine`.
{: .prompt-info }

> Remember to back up your data before proceeding.
{: .prompt-warning }

Create a directory for the database dump.

```shell
mkdir ~/db_dump
```

Create the database dump.

```shell
docker exec -t <pg14_container_name> pg_dumpall -U postgres > ~/db_dump/dump.sql
```

Stop and remove the old database container or stop the whole service, e.g., in Portainer `Stop this stack`.

```shell
docker stop <pg14_container_name>
docker rm <pg14_container_name>
```

Create a new volume for the database.

> Make sure it has the same naming convention as the previous database volume. For example, when using Portainer, the stack name will be used as a prefix. Use `docker volume ls` to find out exactly how the previous volume was named.
{: .prompt-info }

```shell
docker volume create <stack-name>_pg17_data
```

Create a new database container with a new database volume. This is temporary; however, use the same database **password**, **username**, and **database** name as on the previous database.

```shell
docker run -d \
  --name pg17 \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=your_db_name \
  -v <stack-name>_pg17_data:/var/lib/postgresql/data \
  postgres:17-alpine
```

Restore data from the backup dump to the new database.

```shell
docker exec -i pg17 /bin/bash -c "PGPASSWORD=password psql --username postgres your_db_name" < ~/db_dump/dump.sql
```

Stop and remove the temporary container.

```shell
docker stop pg17
docker rm pg17
```

Update the previous setup in `docker-compose.yaml` to run with the new database image `postgres:17-alpine` and new database volume.

```diff
...

db:
-  image: postgres:14.2-alpine
+  image: postgres:17-alpine
  volumes:
-    - db_data:/var/lib/postgresql/data
+    - pg17_data:/var/lib/postgresql/data
...

volumes:
-  db_data:
+  pg17_data:
```

Once verified that everything works as expected, the old volume and database dump can be removed.

```shell
docker volume rm <old_db_volume>
rm -rf ~/db_dump
```

**_Voilà!_**