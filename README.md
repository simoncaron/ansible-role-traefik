# Ansible Role: Traefik

[![ubuntu-18](https://img.shields.io/badge/ubuntu-18.x-orange?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![ubuntu-20](https://img.shields.io/badge/ubuntu-20.x-orange?style=flat&logo=ubuntu)](https://ubuntu.com/)
[![debian-9](https://img.shields.io/badge/debian-9.x-orange?style=flat&logo=debian)](https://www.debian.org/)
[![debian-10](https://img.shields.io/badge/debian-10.x-orange?style=flat&logo=debian)](https://www.debian.org/))

[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg?style=flat)](https://opensource.org/licenses/MIT)
[![GitHub issues](https://img.shields.io/github/issues/OnkelDom/ansible-role-traefik?style=flat)](https://github.com/OnkelDom/ansible-role-traefik/issues)
[![GitHub tag](https://img.shields.io/github/tag/OnkelDom/ansible-role-traefik.svg?style=flat)](https://github.com/OnkelDom/ansible-role-traefik/tags)
[![GitHub action](https://github.com/OnkelDom/ansible-role-traefik/workflows/ansible-lint/badge.svg)](https://github.com/OnkelDom/ansible-role-traefik)

## Description

Deploy [Traefik](https://github.com/traefik/traefik) Webserver.

You can define a custom Rule with `<name>.yml` in files folder. The role copy this to the rules folder on server. I have added one example for unifi controler in unifi.yml

## Requirements

- Ansible >= 2.5 (It might work on previous versions, but we cannot guarantee it)

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `proxy_env` | {} | Proxy environment variables |
| `traefik_version` | 2.4.11 | Traefik package version. Also accepts `latest` as parameter. |
| `traefik_config_dir` | /etc/traefik | Path to directory with traefik configuration |
| `traefik_log_dir` | /var/log/traefik | Path to directory with traefik logfiles |
| `traefik_binary_install_dir` | /usr/local/bin | Path to directory with traefik binaries |
| `traefik_system_user` | traefik | Traefik system user |
| `traefik_system_group` | traefik | Traefik system group |
| `traefik_limit_nofile` | 8192 | nofile limit in systemd unit |
| `traefik_default_domain` | "{{ ansible_domain }}" | Set default domain |
| `traefik_config_flags_extra` | {} | Set additional startup params in systemd unit |
| `traefik_config` | {} | traefik config file entries |
| `traefik_rules_config` | [] | traefik custom rules in array - see exmaple |

## Example

```yml
traefik_config:
  global:
    checkNewVersion: true
    sendAnonymousUsage: true
    entryPoints:
      http:
        address: :80
      https:
        address: :443
traefik_rules_config:
# Traefik rules config files with two exmaples
traefik_rules_config:
  - name: dashboard
    content:
      http:
        routers:
          dashboard:
            rule: Host(`traefik.{{ traefik_default_domain }}`)
            service: api@internal # This is the defined name for api. You cannot change it.
            tls:
              certresolver: dns-cloudflare
  - name: unifi
    content:
      http:
        routers:
          unifi-rtr:
            rule: "Host(`unifi.{{ traefik_default_domain }}`)" # will only work with cloudflare Full SSL (not Strict)
            entryPoints:
              - https
            service: unifi-svc
            middlewares:
              - rate-limit
              - secure-headers
              - https-redirect
            tls:
              certResolver: dns-cloudflare
        services:
          unifi-svc:
            loadBalancer:
              servers:
                - url: "https://192.168.1.2:8443" # or whatever your external host's IP:port is
  - name: adguard
    content:
      http:
        routers:
          adguard-rtr:
            rule: "Host(`adguard.{{ traefik_default_domain }}`)"  # will only work with cloudflare Full SSL (not Strict)
            entryPoints:
              - https
            service: adguard-svc
            middlewares:
              - rate-limit
              - secure-headers
              - https-redirect
            tls:
              certResolver: dns-cloudflare
      #        passthrough: true
        services:
          adguard-svc:
            loadBalancer:
              servers:
                - url: "http://192.168.1.2:3000" # or whatever your external host's IP:port is
                  scheme: https
```

## Playbook

```yaml
---
- hosts: all
  roles:
  - onkeldom.traefik
  vars:
    traefik_config:
      global:
        checkNewVersion: true
        sendAnonymousUsage: true
        entryPoints:
          http:
            address: :80
          https:
            address: :443
    traefik_rules_config:
    # Traefik rules config files with two exmaples
    traefik_rules_config:
      - name: dashboard
        content:
          http:
            routers:
              dashboard:
                rule: Host(`traefik.{{ traefik_default_domain }}`)
                service: api@internal # This is the defined name for api. You cannot change it.
                tls:
                  certresolver: dns-cloudflare
      - name: unifi
        content:
          http:
            routers:
              unifi-rtr:
                rule: "Host(`unifi.{{ traefik_default_domain }}`)" # will only work with cloudflare Full SSL (not Strict)
                entryPoints:
                  - https
                service: unifi-svc
                middlewares:
                  - rate-limit
                  - secure-headers
                  - https-redirect
                tls:
                  certResolver: dns-cloudflare
            services:
              unifi-svc:
                loadBalancer:
                  servers:
                    - url: "https://192.168.1.2:8443" # or whatever your external host's IP:port is
                      scheme: https
      - name: adguard
        content:
          http:
            routers:
              adguard-rtr:
                rule: "Host(`adguard.{{ traefik_default_domain }}`)"  # will only work with cloudflare Full SSL (not Strict)
                entryPoints:
                  - https
                service: adguard-svc
                middlewares:
                  - rate-limit
                  - secure-headers
                  - https-redirect
                tls:
                  certResolver: dns-cloudflare
            services:
              adguard-svc:
                loadBalancer:
                  servers:
                    - url: "http://192.168.1.2:3000" # or whatever your external host's IP:port is
```

## Contributing

See [contributor guideline](CONTRIBUTING.md).

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.
