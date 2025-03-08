---
title: Caddy monitoring with Zabbix
date: 2025-03-08 04:48:15
tags: [zabbix, template, monitoring, caddy, website]
---

There is a way to monitor [Caddy](https://caddyserver.com/) with Prometheus, but my setup is using Zabbix and I could not find a ready made template for Caddy, so I made one. The template uses HTTP agent to get the data from exposed [Caddy metrics](https://caddyserver.com/docs/metrics).

## Enable Caddy metrics

[Caddy documentation](https://caddyserver.com/docs/metrics)

Enable metrics in `Caddyfile` and expose them on port `8080` on the `/metrics` path:

> When exposing metrics on a different port, make sure that port is exposed also in docker container.
{: .prompt-warning }

```conf
{
  servers {
    metrics
  }
}

:8080 {
  metrics /metrics
}
```

## Zabbix

> Created on **Zabbix ver. 7.2**
{: .prompt-info }

### Import template

Download the template file `zabbix-caddy.yaml` from my [Github repo](https://github.com/z0gst/zabbix-caddy).

Navigate to `Data collection -> Templates` and click `Import` in top right corner in Zabbix.

Choose the downloaded file to import and click `Import`.

### Using template

There are three macros to set up the template:

- `{$CADDY.HOST}` - default: **127.0.0.1**
- `{$CADDY.PATH}` - default: **/metrics**
- `{$CADDY.PORT}` - default: **8080**
