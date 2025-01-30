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
    - role: dreknix.tool_deploy
```

The variable `tool_deploy_list` is a dictionary, so that this dictionary can be
defined in `group_vars`, but also be overwritten partially in `host_vars`. This
allows different versions of tools to be installed.

All tools described in the [example](#configuration-examples) section can be
referenced with `tool_deploy_default_config_XXXX`. The above playbook is
equivalent to the following playbook definition:

``` yaml
- name: Deploying tools
  hosts: '{{ target | default("all") }}'
  vars:
    delta_version: 0.18.2
    jq_version: 1.7.1
    task_version: 3.40.1

  pre_tasks:
    - name: Set dictionary 'tool_deploy_list'
      ansible.builtin.set_fact:
        tool_deploy_list: >-
          {{ tool_deploy_list | combine(tool) }}
      loop_control:
        loop_var: tool
        label: "{{ tool.keys() }}"
      loop:
        - "{{ tool_deploy_default_config_delta }}"
        - "{{ tool_deploy_default_config_jq }}"
        - "{{ tool_deploy_default_config_task }}"

  roles:
    - role: dreknix.tool_deploy
```

## Install role via `roles/requirements.yml`

### Read-Only copy

Configure the role as read-only copy:

``` yaml
---
roles:
  - name: dreknix.tool_deploy
    src: https://github.com/dreknix/ansible-role-tool-deploy.git
    scm: git
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
roles:
  - name: dreknix.tool_deploy
    src: git@github.com:dreknix/ansible-role-tool-deploy.git
    scm: git
...
```

Install the working copy:

``` console
ansible-galaxy role install --force --keep-scm-meta -r roles/requirements.yml
```

## Configuration Examples

All this configuration examples are also defined in `vars/main/` for referencing
in playbooks.

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
    man_pages:
      src: "bat.1"
    bash_completion:
      src: "autocomplete/bat.bash"
    zsh_completion:
      src: "autocomplete/bat.zsh"
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

### dive

[wagoodman / dive](https://github.com/wagoodman/dive)

``` yaml
dive_version: 0.12.0

tool_deploy_list:
  dive:
    action: download_archive
    github:
      repo: wagoodman/dive
      file: "v{{ dive_version | mandatory }}/dive_{{ dive_version | mandatory }}_linux_amd64.tar.gz"
    version:
      args: --version
      match: "dive {{ dive_version | mandatory }}"
```

### eza

[eza-community / eza](https://github.com/eza-community/eza)

``` yaml
eza_version: 0.20.19

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
    man_pages:
      src: "fd.1"
    bash_completion:
      src: "autocomplete/fd.bash"
    zsh_completion:
      src: "autocomplete/_fd"
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
    man_pages:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/man/man1/fzf.1
    bash_completion:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/shell/completion.bash
    zsh_completion:
      url: https://raw.githubusercontent.com/junegunn/fzf/refs/heads/master/shell/completion.zsh
```

### hadolint

[hadolint / hadolint](https://github.com/hadolint/hadolint)

``` yaml
hadolint_version: 2.12.0

tool_deploy_list:
  hadolint:
    action: download_binary
    github:
      repo: hadolint/hadolint
      file: "v{{ hadolint_version | mandatory }}/hadolint-Linux-x86_64"
    version:
      args: --version
      match: "Haskell Dockerfile Linter {{ hadolint_version | mandatory }}"
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

### less

**TODO:** download archive and compile tools

### lazydocker

[jesseduffield / lazydocker](https://github.com/jesseduffield/lazydocker)

``` yaml
lazydocker_version: 0.45.2

tool_deploy_list:
  lazydocker:
    action: download_archive
    github:
      repo: jesseduffield/lazydocker
      file: "v{{ lazydocker_version | mandatory }}/lazydocker_{{ lazydocker_version | mandatory }}_Linux_x86_64.tar.gz"
    version:
      args: --version
      match: "version={{ lazydocker_version | mandatory }}"
```

### lazygit

[jesseduffield / lazygit](https://github.com/jesseduffield/lazygit)

``` yaml
lazygit_version: 0.45.2

tool_deploy_list:
  lazygit:
    action: download_archive
    github:
      repo: jesseduffield/lazygit
      file: "v{{ lazygit_version | mandatory }}/lazygit_{{ lazygit_version | mandatory }}_Linux_x86_64.tar.gz"
    version:
      args: --version
      match: "version={{ lazygit_version | mandatory }}"
```

### lsd

[lsd-rs / lsd](https://github.com/lsd-rs/lsd)

``` yaml
lsd_version: 1.1.5

tool_deploy_list:
  lsd:
    action: download_archive
    github:
      repo: lsd-rs/lsd
      file: v{{ lsd_version }}/lsd-v{{ lsd_version }}-x86_64-unknown-linux-gnu.tar.gz
    version:
      args: --version
      match: "lsd {{ lsd_version }}"
    man_pages:
      src: lsd.1
    bash_completion:
      src: autocomplete/lsd.bash-completion
    zsh_completion:
      src: autocomplete/_lsd
```

### oh-my-posh

[JanDeDobbeleer / oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)

``` yaml
oh_my_posh_version: 24.19.0

tool_deploy_list:
  oh-my-posh:
    action: download_binary
    github:
      repo: JanDeDobbeleer/oh-my-posh
      file: "v{{ oh_my_posh_version | mandatory }}/posh-linux-amd64"
    version:
      args: --version
      match: "{{ oh_my_posh_version | mandatory }}"
```

### pet

[knqyf263 / pet](https://github.com/knqyf263/pet)

``` yaml
pet_version: 1.0.1

tool_deploy_list:
  pet:
    action: download_archive
    github:
      repo: knqyf263/pet
      file: "v{{ pet_version | mandatory }}/pet_{{ pet_version | mandatory }}_linux_amd64.tar.gz"
    version:
      args: version
      match: "pet version {{ pet_version | mandatory }}"
    zsh_completion:
      src: "misc/completions/zsh/_pet"
```

### restic

[restic / restic](https://github.com/restic/restic)

``` yaml
restic_version: 0.17.3

tool_deploy_list:
  restic:
    action: download_binary
    github:
      repo: restic/restic
      file: "v{{ restic_version | mandatory }}/restic_{{ restic_version | mandatory }}_linux_amd64.bz2"
    version:
      args: version
      match: "restic {{ restic_version | mandatory }}"
    man_pages:
      url:
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-backup.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-cache.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-cat.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-check.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-copy.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-diff.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-dump.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-features.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-find.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-forget.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-generate.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-init.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-key-add.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-key-list.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-key-passwd.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-key-remove.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-key.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-list.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-ls.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-migrate.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-mount.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-options.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-prune.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-recover.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-repair-index.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-repair-packs.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-repair-snapshots.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-repair.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-restore.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-rewrite.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-self-update.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-snapshots.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-stats.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-tag.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-unlock.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic-version.1
        - https://raw.githubusercontent.com/restic/restic/refs/heads/master/doc/man/restic.1
    bash_completion:
      generate_args: generate --bash-completion /dev/stdout
    zsh_completion:
      generate_args: generate --zsh-completion /dev/stdout
```

### ripgrep / rg

[BurntSushi / ripgrep](https://github.com/BurntSushi/ripgrep)

``` yaml
rg_version: 14.1.1

tool_deploy_list:
  rg:
    action: download_archive
    github:
      repo: BurntSushi/ripgrep
      file: "{{ rg_version }}/ripgrep-{{ rg_version }}-x86_64-unknown-linux-musl.tar.gz"
    version:
      args: --version
      match: "ripgrep {{ rg_version }} ("
    man_pages:
      src: doc/rg.1
    bash_completion:
      src: complete/rg.bash
    zsh_completion:
      src: complete/_rg
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

### uv

[astral-sh / uv](https://github.com/astral-sh/uv)

``` yaml
uv_version: 0.5.25

tool_deploy_list:
  uv:
    action: download_archive
    github:
      repo: astral-sh/uv
      file: "{{ uv_version | mandatory }}/uv-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "uv {{ uv_version | mandatory }}"
```

### xq

[sibprogrammer / xq](https://github.com/sibprogrammer/xq)

``` yaml
xq_version: 4.45.1

tool_deploy_list:
  xq:
    action: download_archive
    github:
      repo: sibprogrammer/xq
      file: "v{{ xq_version | mandatory }}/xq_{{ xq_version | mandatory }}_linux_amd64.tar.gz"
    version:
      args: --version
      match: "xq version {{ xq_version | mandatory }}"
```

### yq

[mikefarah / yq](https://github.com/mikefarah/yq)

``` yaml
yq_version: 4.45.1

tool_deploy_list:
  yq:
    action: download_binary
    github:
      repo: mikefarah/yq
      file: "v{{ yq_version | mandatory }}/yq_linux_amd64"
    version:
      args: --version
      match: "version v{{ yq_version | mandatory }}"
```

### zoxide

[ajeetdsouza / zoxide](https://github.com/ajeetdsouza/zoxide)

``` yaml
zoxide_version: 0.9.6

tool_deploy_list:
  zoxide:
    action: download_archive
    github:
      repo: ajeetdsouza/zoxide
      file: "v{{ zoxide_version }}/zoxide-{{ zoxide_version }}-x86_64-unknown-linux-musl.tar.gz"
    version:
      args: --version
      match: "zoxide {{ zoxide_version }}"
    man_pages:
      src:
        - man/man1/zoxide.1
        - man/man1/zoxide-add.1
        - man/man1/zoxide-import.1
        - man/man1/zoxide-init.1
        - man/man1/zoxide-query.1
        - man/man1/zoxide-remove.1
    bash_completion:
      src: completions/zoxide.bash
    zsh_completion:
      src: completions/_zoxide
```

## License

[MIT](https://github.com/dreknix/ansible-role-tool-deploy/blob/main/LICENSE)
