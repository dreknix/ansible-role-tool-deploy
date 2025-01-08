# Ansible Role - Tool Deploy

Ansible role for deploying tools form GitHub, GitLab, and other sources.

**IMPORTANT:** This role is currently in development. It is only possible to
install tools from GitHub. More possibilities must be added.

## Using in a Playbook

``` yaml
```

A more complex example:

``` yaml
```

## Install role via `roles/requirements.yml`

### Read-Only copy

Configure the role as read-only copy:

``` yaml
---
- name: dreknix.tool_deploy
  src: https://github.com/dreknix/ansible-role-tool-deploy.git
  type: git
  version: main
...
```

Install the role:

``` console
ansible-galaxy role install --force -r roles/requirements.yml
```

### Working copy

Configure the role as a working copy:

``` yaml
---
- name: dreknix.tool_deploy
  src: git@github.com:dreknix/ansible-role-tool-deploy.git
  type: git
  version: main
...
```

Install the working copy:

``` console
ansible-galaxy role install --force --keep-scm-meta -r roles/requirements.yml
```

## License

[MIT](https://github.com/dreknix/ansible-role-tool-deploy/blob/main/LICENSE)
