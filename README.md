# Ansible Role - Tool Deploy

Ansible role for deploying tools form GitHub, GitLab, and other sources.

**IMPORTANT:** This role is currently in development. It is only possible to
install tools from GitHub. More possibilities must be added.

## Using in a Playbook

This Ansible will deploy tools specified in the variable `tool_deploy_list` to
your infrastructure. For each tool the source URL will be given by `url` or
`github.repo` and `github.file`. The role checks if the binary is present and
when `version.match` is provided, if the correct version of the tool is
installed. The installation can be forced with the flag `force_install`.

``` yaml
- name: Deploying tools
  hosts: '{{ target | default("all") }}'
  vars:

    delta_version: 0.18.2
    jq_version: 1.7.1
    task_version: 3.40.1

    tool_deploy_list:

      delta:
        # force_install: true
        action: download_archive
        github:
          repo: dandavison/delta
          file: >-
            {{ delta_version }}/delta-{{ delta_version }}-x86_64-unknown-linux-gnu.tar.gz
        version:
          args: --version
          match: "delta {{ delta_version }}"
        bash_completion:
          generate_args: --generate-completion bash
        zsh_completion:
          generate_args: --generate-completion zsh

      jq:
        # force_install: true
        action: download_binary
        github:
          repo: jqlang/jq
          file: >-
            jq-{{ jq_version }}/jq-linux-amd64
        version:
          args: --version
          match: "jq-{{ jq_version }}"

      task:
        # force_install: true
        action: download_archive
        github_repo: go-task/task
        release_file: >-
          v{{ task_version }}/task_linux_amd64.tar.gz
        version:
          args: --version
          match: "Task version: v{{ task_version }}"
        bash_completion:
          src: "completion/bash/task.bash"
        zsh_completion:
          src: "completion/zsh/_task"

  roles:
    - role: tool_deploy
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
