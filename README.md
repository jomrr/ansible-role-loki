# Ansible Role: loki

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-loki)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-loki)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-loki)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-loki/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-loki/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-loki/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-loki/actions/workflows/main.yml?query=branch%3Amain)

Install Grafana Loki from Grafana's package repositories and configure its
systemd service.

## Purpose

Run a single Loki process that accepts and queries logs on 127.0.0.1:3100. The
default uses TSDB schema v13, a local ring and persistent filesystem storage
under /var/lib/loki.

## Scope

### Managed

- Grafana's stable signed package repository, the loki package and its systemd
  service.
- A dedicated service account, persistent state directory and validated Loki
  configuration.

### Not Managed

- Firewall rules, log shippers, authentication proxies, TLS certificates and
  object storage provisioning.

## Requirements

- Target hosts need systemd and access to apt.grafana.com or rpm.grafana.com.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
  - name: containers.podman
    version: '>=1.7.0'
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

## Managed Files

- `/etc/loki/loki.yaml`
- `/etc/systemd/system/loki.service.d/10-ansible.conf`
- `/var/lib/loki`

## Check Mode

Run check mode on a provisioned host. A first installation needs a normal run to
install the validator and service unit.

## Service Behavior

Loki validates a candidate configuration before it replaces the live file.
Changes to configuration or the systemd drop-in restart the service. An
unchanged run leaves a running instance in place and starts a stopped service.

## Security Notes

- HTTP and gRPC listen on loopback by default. Place an authenticating reverse
  proxy in front of a remotely accessible endpoint.
- Loki's auth_enabled setting enforces tenant headers. The proxy must
  authenticate clients and supply a trusted X-Scope-OrgID.
- The configuration is readable by root and the loki group. Store credentials
  supplied in loki_config in Ansible Vault.

## Operational Notes

- Set native YAML keys in loki_config. The default merge mode recursively merges
  mappings and replaces whole lists.
- Use replace mode for a complete configuration, for example when switching the
  storage backend or deploying cluster components. Provide the required schema,
  ring and storage settings, and provision external resources and credentials.
- When extending schema_config.configs, include all historical periods and add a
  future-dated period for the new schema.

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

### Start a local Loki instance

Apply the role and check readiness with `curl http://127.0.0.1:3100/ready`.

```yaml
- name: Start Loki
  hosts: log_servers
  gather_facts: true
  roles:
    - role: jomrr.loki
```

### Change the HTTP port and retain logs for 30 days

Keep the local store and enable compactor retention. Retention deletes expired logs.

```yaml
- name: Configure log retention
  hosts: log_servers
  gather_facts: true
  roles:
    - role: jomrr.loki
      loki_config:
        server:
          http_listen_port: 3101
          log_level: warn
        limits_config:
          retention_period: 720h
        compactor:
          retention_enabled: true
          delete_request_store: filesystem
```

### Use an S3 bucket

Use an existing bucket and grant access through the host’s IAM role.

```yaml
- name: Store Loki logs in S3
  hosts: log_servers
  gather_facts: true
  roles:
    - role: jomrr.loki
      loki_config_mode: replace
      loki_config:
        auth_enabled: false
        server:
          http_listen_address: 127.0.0.1
          grpc_listen_address: 127.0.0.1
        common:
          instance_addr: 127.0.0.1
          path_prefix: /var/lib/loki
          replication_factor: 1
          ring:
            kvstore:
              store: inmemory
          storage:
            s3:
              region: eu-central-1
              bucketnames: my-loki-logs
        schema_config:
          configs:
            - from: '2024-01-01'
              store: tsdb
              object_store: s3
              schema: v13
              index:
                prefix: index_
                period: 24h
```

## References

- [Grafana package repositories](https://apt.grafana.com/)
- [Loki configuration reference](https://grafana.com/docs/loki/latest/reference/loki-config-ref/)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
