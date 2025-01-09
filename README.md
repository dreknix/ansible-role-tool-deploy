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

The variable `tool_deploy_list` is a dictionary, so that this dictionary can be
defined in `group_vars`, but also be overwritten partially in `host_vars`. This
allows different versions of tools to be installed.

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

## Configuration Examples

### bat

[sharkdp / bat](https://github.com/sharkdp/bat)

``` yaml
bat_version: 0.25.0

tool_deploy_list:
  bat:
    action: download_archive
    github:
      repo: sharkdp/bat
      file: "v{{ bat_version }}/bat-v{{ bat_version }}-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "bat {{ bat_version }}"
    bash_completion:
      src: "autocomplete/bat.bash"
    zsh_completion:
      src: "autocomplete/bat.zsh"
    man_pages:
      src: "bat.1"
```

### delta

[dandavison / delta](https://github.com/dandavison/delta)

``` yaml
delta_version: 0.18.2

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
```

### direnv

[direnv / direnv](https://github.com/direnv/direnv)

``` yaml
direnv_version: 2.35.0

tool_deploy_list:
  direnv:
    action: download_binary
    github:
      repo: direnv/direnv
      file: "v{{ direnv_version }}/direnv.linux-amd64"
    version:
      args: version
      match: "{{ direnv_version }}"
    man_pages:
      url:
        - https://raw.githubusercontent.com/direnv/direnv/refs/heads/master/man/direnv.1
        - https://raw.githubusercontent.com/direnv/direnv/refs/heads/master/man/direnv-fetchurl.1
        - https://raw.githubusercontent.com/direnv/direnv/refs/heads/master/man/direnv-stdlib.1
        - https://raw.githubusercontent.com/direnv/direnv/refs/heads/master/man/direnv.toml.1
```

### eza

[eza-community / eza](https://github.com/eza-community/eza)

``` yaml
eza_version: 0.20.16

tool_deploy_list:
  eza:
    action: download_archive
    github:
      repo: eza-community/eza
      file: "v{{ eza_version }}/eza_x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "v{{ eza_version }}"
```

### fd

[sharkdp / fd](https://github.com/sharkdp/fd)

``` yaml
fd_version: 10.2.0

tool_deploy_list:
  fd:
    action: download_archive
    github:
      repo: sharkdp/fd
      file: "v{{ fd_version }}/fd-v{{ fd_version }}-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "fd {{ fd_version }}"
    bash_completion:
      src: "autocomplete/fd.bash"
    zsh_completion:
      src: "autocomplete/_fd"
    man_pages:
      src: "fd.1"
```

### fzf

[junegunn / fzf](https://github.com/junegunn/fzf)

``` yaml
fzf_version: 0.57.0

tool_deploy_list:
  fzf:
    action: download_archive
    github:
      repo: junegunn/fzf
      file: "v{{ fzf_version }}/fzf-{{ fzf_version }}-linux_amd64.tar.gz"
    version:
      args: --version
      match: "{{ fzf_version }} ("
    bash_completion:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/shell/completion.bash
    zsh_completion:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/shell/completion.zsh
    man_pages:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/man/man1/fzf.1
```

### jq

[go-task / task](https://github.com/go-task/task)

``` yaml
jq_version: 1.7.1

tool_deploy_list:
  jq:
    action: download_binary
    github:
      repo: jqlang/jq
      file: "jq-{{ jq_version }}/jq-linux-amd64"
    version:
      args: --version
      match: "jq-{{ jq_version }}"
```

### task

[go-task / task](https://github.com/go-task/task)

``` yaml
task_version: 3.40.1

tool_deploy_list:
  task:
    action: download_archive
    github:
      repo: go-task/task
      file: "v{{ task_version }}/task_linux_amd64.tar.gz"
    version:
      args: --version
      match: "Task version: v{{ task_version }}"
    bash_completion:
      src: "completion/bash/task.bash"
    zsh_completion:
      src: "completion/zsh/_task"
```

## License

[MIT](https://github.com/dreknix/ansible-role-tool-deploy/blob/main/LICENSE)
