---
title: Tailscale in Alpine Linux
date: 2025-05-08 03:54:14
tags: [tailscale, vm, alpine]
---

I have been running [Tailscale](https://tailscale.com/) in an [Ubuntu](https://ubuntu.com/) [VM](https://en.wikipedia.org/wiki/Virtual_machine), and it has been stable and working as expected for months. Lately however I have been replacing homelab services to [Alpine Linux](https://alpinelinux.org/) based [Docker](https://www.docker.com/) containers to save up on some precious resources in my resource poor homelab. This made me wonder, if I can change the VM where Tailscale is running to Alpine? Turns out I can, and in the process save about 500 MB of memory ([Proxmox](https://www.proxmox.com/en) has been reporting the memory usage to be between approximately 550 - 750 MB for the Ubuntu VM) that I can use for other services. Win-win!

These steps are right after a clean Alpine Linux install. Installation steps can be found [here](https://wiki.alpinelinux.org/wiki/Installation#Installation_Step_Details).

## Preparation

Update packages and install [nano](https://www.nano-editor.org/) for the convenience.

```shell
doas apk update
doas apk add nano
```

Enable community repos

```shell
doas setup-apkrepos -c -1
```

## Tailscale setup

Install Tailscale

```shell
doas apk add tailscale
```

Enable Tailscale service after boot

```shell
doas rc-update add tailscale default
```

Start Tailscale service

```shell
doas rc-service tailscale start
```

Check that Tailscale service is running

```shell
doas rc-service tailscale status
```

### Enable IP forwarding

Enable IP forwarding to be able to use the VM as an [exit node](https://tailscale.com/kb/1103/exit-nodes) and access other VMs and services in the same network.

Edit `/etc/sysctl.conf` file

```shell
doas nano /etc/sysctl.conf
```

Add these lines to the file

```conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

Load these new settings

```shell
doas sysctl -p /etc/sysctl.conf
```

Verify that the new settings were loaded

```shell
sysctl net.ipv4.ip_forward net.ipv6.conf.all.forwarding
```

## Run Tailscale

It is time to fire up Tailscale...

```shell
doas tailscale up --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

Login and approve the exit-node and routes in Tailscale admin panel.

Reboot. If everything is configured correctly, Tailscale service should be running and can be used as an exit-node as well as subnet router for advertised routes.

To check that Tailscale service is running after the boot, run

```shell
rc-service tailscale status
```

Also Tailscale status can be checked with

```shell
doas tailscale status
```

**_Voilà!_**