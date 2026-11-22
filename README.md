# Ansible Role: loki

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-loki)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-loki)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-loki)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-loki/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-loki/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-loki/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-loki/actions/workflows/main.yml?query=branch%3Amain)

Install Grafana Loki from Grafana's package repositories and configure its
systemd service.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `loki_config`

Type: `dict`. Required: `false`.

Native Loki YAML settings, including listeners, limits, schema periods, storage,
ring and ruler.

Default:

```yaml
loki_config: {}
```

### `loki_config_mode`

Type: `str`. Required: `false`.

Recursively merge settings into the standalone defaults or replace the entire
configuration. Lists are replaced.

Default:

```yaml
loki_config_mode: merge
```

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Simple example playbook

Minimal example for applying this role.

```yaml
---
- name: "Configure loki"
  hosts: "loki"
  gather_facts: true
  roles:
    - role: "jomrr.loki"
```

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
