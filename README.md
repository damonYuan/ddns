# Ansible Role: ddns

Keeps Cloudflare A records pointed at a host's current public IPv4 address. The
role creates or updates the configured records, installs a small updater script,
and schedules it with cron. It can optionally send `mail` notifications when the
IP changes or an update fails.

Based on the script from
[K0p1-Git/cloudflare-ddns-updater](https://github.com/K0p1-Git/cloudflare-ddns-updater).

## Requirements

- Ansible 2.15+
- Ubuntu 22.04 / 24.04 (or another cron-based distribution)
- `ddns_user` must already exist (default `app`)
- `curl` on the target host
- `mail` on the target host if `ddns_notify_email` is set
- A Cloudflare API token with `Zone.DNS:Edit` permission for the zone

## Role Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `ddns_state` | `present` | no | `present`, `absent`, or `purged` |
| `ddns_zone` | `""` | yes | Cloudflare zone, e.g. `example.com` |
| `ddns_records` | `[]` | yes | List of records; each item has `name` (subdomain) and optional `proxied` (bool, default `false`) |
| `ddns_cloudflare_zone_id` | `""` | yes | Zone ID from the Cloudflare dashboard |
| `ddns_cloudflare_api_token` | `""` | yes | Scoped API token (vault it) |
| `ddns_user` | `app` | no | Existing local user that owns and runs the cron job |
| `ddns_base_dir` | `/data/apps/ddns` | no | Application directory |
| `ddns_logs_dir` | `/data/logs/ddns` | no | Log directory |
| `ddns_cron_minute` | `*/10` | no | Cron minute field |
| `ddns_ttl` | `3600` | no | DNS TTL used by the updater script |
| `ddns_notify_email` | `""` | no | Recipient for mail notifications |

The role creates `<ddns_base_dir>/logs` as a symlink to `ddns_logs_dir` and
installs `/etc/logrotate.d/ddns-logrotate.conf`.

## Dependencies

None.

## Example Playbook

```yaml
- name: Configure DDNS
  hosts: all
  become: true
  vars:
    ddns_zone: example.com
    ddns_cloudflare_zone_id: "your-zone-id"
    ddns_cloudflare_api_token: "{{ vault_cloudflare_api_token }}"
    ddns_records:
      - name: home
        proxied: false
  roles:
    - ddns
```

## Removal

```bash
ansible-playbook playbook.yml -e ddns_state=absent
ansible-playbook playbook.yml -e ddns_state=purged
```

`absent` removes the cron jobs and scripts. `purged` also removes the application
and log directories and the logrotate configuration. Cloudflare records are not
deleted.

## License

MIT

## Author

Damon Yuan
