# [proserver-ansible-redis](https://github.com/punktDe/proserver-ansible-mariadb)

Ansible role to configure Redis on a proServer.

## Requirements

- proServer or Debian/Ubuntu Linux
- Ansible >=2.9.0
- Ansible option `hash_behaviour` set to `merge`

## Configuration

**1)** Add the role to your playbook.
You could add this repository as submodule to your Ansible project's Git repository.

```
git submodule add https://github.com/punktDe/proserver-ansible-redis.git roles/redis
```

```yaml
- name: redis
  hosts: all
  become: yes
  roles:
    - redis
```

**2)** (Optional) Reconfigure Redis (e.g. in group_vars)

[List of available redis.conf options](https://raw.githubusercontent.com/antirez/redis/unstable/redis.conf)

```yaml
redis:
  redis.conf:
    databases: 64
```

## Replication

See [REPLICATION.md](documentation/REPLICATION.md).
