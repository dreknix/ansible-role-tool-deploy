# Ansible Role - Tool Deploy

Ansible role for deploying tools form GitHub, GitLab, and other sources.

**IMPORTANT:** This role is currently in development. It is only possible to
install tools from GitHub. More possibilities must be added.

## TODO

Tools to look into:

* [walles / moar](https://github.com/walles/moar)
* [jdx / mise](https://github.com/jdx/mise)

Features:

* compile install without package install

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
    jq_version: 1.8.1
    task_version: 3.44.1

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
          src: completion/bash/task.bash
        zsh_completion:
          src: completion/zsh/_task

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
    jq_version: 1.8.1
    task_version: 3.44.1

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
      src: bat.1
    bash_completion:
      src: autocomplete/bat.bash
    zsh_completion:
      src: autocomplete/bat.zsh
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
direnv_version: 2.37.1

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
dive_version: 0.13.1

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

### docker-credential-pass

[docker / docker-credential-helpers](https://github.com/docker/docker-credential-helpers)

``` yaml
docker_credential_version: 0.9.3

tool_deploy_list:
  docker-credential-pass:
    action: download_binary
    github:
      repo: docker/docker-credential-helpers
      file: "v{{ docker_credential_version }}/docker-credential-pass_v{{ docker_credential_version }}.linux-amd64"
    version:
      args: --version
      match: "v{{ docker_credential_version | mandatory }}"
```

### eza

[eza-community / eza](https://github.com/eza-community/eza)

``` yaml
eza_version: 0.23.0

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
      src: fd.1
    bash_completion:
      src: autocomplete/fd.bash
    zsh_completion:
      src: autocomplete/_fd
```

### fx

[antonmedv / fx](https://github.com/antonmedv/fx)

``` yaml
fx_version: 38.0.0
  fx:
    action: download_binary
    github:
      repo: antonmedv/fx
      file: "{{ fx_version | mandatory }}/fx_linux_amd64"
    version:
      args: --version
      match: "{{ fx_version | mandatory }}"
    bash_completion:
      generate_args: --comp bash
    zsh_completion:
      generate_args: --comp zsh

tool_deploy_list:
```

### fzf

[junegunn / fzf](https://github.com/junegunn/fzf)

``` yaml
fzf_version: 0.65.0

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

### glow

[charmbracelet / glow](https://github.com/charmbracelet/glow)

``` yaml
glow_version: 2.1.1

tool_deploy_list:
  glow:
    action: download_archive
    github:
      repo: charmbracelet/glow
      file: "v{{ glow_version | mandatory }}/glow_{{ glow_version }}_Linux_x86_64.tar.gz"
    version:
      args: --version
      match: "glow version {{ glow_version | mandatory }}"
    man_pages:
      src:
        - manpages/glow.1.gz
    bash_completion:
      src: completions/glow.bash
    zsh_completion:
      src: completions/glow.zsh
```

### gopass

[gopasspw / gopass](https://github.com/gopasspw/gopass)

``` yaml
gopass_version: 1.15.16

tool_deploy_list:
  gopass:
    action: download_archive
    github:
      repo: gopasspw/gopass
      file: "v{{ gopass_version | mandatory }}/gopass-{{ gopass_version | mandatory }}-linux-amd64.tar.gz"
    version:
      args: version
      match: "gopass {{ gopass_version | mandatory }}"
    bash_completion:
      src: bash.completion
    zsh_completion:
      src: zsh.completion
```

### gopass-jsonapi

[gopasspw / gopass-jsonapi](https://github.com/gopasspw/gopass-jsonapi)

``` yaml
gopass_jsonapi_version: 1.15.16

tool_deploy_list:
  gopass-jsonapi:
    action: download_archive
    github:
      repo: gopasspw/gopass-jsonapi
      file: "v{{ gopass_jsonapi_version | mandatory }}/gopass-jsonapi-{{ gopass_jsonapi_version }}-linux-amd64.tar.gz"
    version:
      args: version
      match: "gopass-jsonapi version {{ gopass_jsonapi_version | mandatory }}"
```

### gum

[charmbracelet / gum](https://github.com/charmbracelet/gum)

``` yaml
gum_version: 0.16.2

tool_deploy_list:
  gum:
    action: download_archive
    github:
      repo: charmbracelet/gum
      file: "v{{ gum_version | mandatory }}/gum_{{ gum_version }}_Linux_x86_64.tar.gz"
    version:
      args: --version
      match: "gum version v{{ gum_version | mandatory }}"
    man_pages:
      src:
        - manpages/gum.1.gz
    bash_completion:
      src: completions/gum.bash
    zsh_completion:
      src: completions/gum.zsh
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

### hexyl

[sharkdp / hexyl](https://github.com/sharkdp/hexyl)

``` yaml
hexyl_version: 0.16.0

tool_deploy_list:
  hexyl:
    action: download_archive
    github:
      repo: sharkdp/hexyl
      file: "v{{ hexyl_version }}/hexyl-v{{ hexyl_version }}-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "hexyl {{ hexyl_version }}"
    man_pages:
      src: hexyl.1
```

### hyperfine

[sharkdp / hyperfine](https://github.com/sharkdp/hyperfine)

``` yaml
hyperfine_version: 1.19.0

tool_deploy_list:
  hyperfine:
    action: download_archive
    github:
      repo: sharkdp/hyperfine
      file: "v{{ hyperfine_version }}/hyperfine-v{{ hyperfine_version }}-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "hyperfine {{ hyperfine_version }}"
    man_pages:
      src: hyperfine.1
    bash_completion:
      src: autocomplete/hyperfine.bash
    zsh_completion:
      src: autocomplete/_hyperfine
```

### jq

[jqlang / jq](https://github.com/jqlang/jq)

``` yaml
jq_version: 1.8.1

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

### KeePassXC

[keepassxreboot / keepassxc](https://github.com/keepassxreboot/keepassxc)

``` yaml
keepassxc_version: 2.7.10

tool_deploy_list:
  keepassxc:
    action: download_appimage
    github:
      repo: keepassxreboot/keepassxc
      file: "{{ keepassxc_version | mandatory }}/KeePassXC-{{ keepassxc_version }}-x86_64.AppImage"
    version:
      args: --version
      match: "keepassxc v{{ keepassxc_version | mandatory }}"
    xdg_desktop:
      src: squashfs-root/org.keepassxc.KeePassXC.desktop
      svg: squashfs-root/keepassxc.svg
```

### kubectl

[kubectl](https://kubernetes.io/docs/reference/kubectl/)

``` yaml
kubectl_version: 1.33.3

tool_deploy_list:
  kubectl:
    action: download_binary
    url: https://dl.k8s.io/release/v{{ kubectl_version }}/bin/linux/amd64/kubectl
    version:
      args: version --client=true
      match: "Client Version: v{{ kubectl_version | mandatory }}"
    bash_completion:
      generate_args: completion bash
    zsh_completion:
      generate_args: completion zsh
```

### lazydocker

[jesseduffield / lazydocker](https://github.com/jesseduffield/lazydocker)

``` yaml
lazydocker_version: 0.24.1

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
lazygit_version: 0.50.0

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

### less

[less homepage](http://greenwoodsoftware.com/less/)

``` yaml
less_version: 668

tool_deploy_list:
  less:
    dependencies:
      - build-essential
    action: download_sources
    url: http://greenwoodsoftware.com/less/less-{{ less_version }}.tar.gz
    version:
      args: --version
      match: "less {{ less_version | mandatory }}"
    build_steps:
      - ./configure prefix={{ tool_deploy_prefix }}
      - make
      - make install
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

### micro

[zyedidia / micro](https://github.com/zyedidia/micro)

``` yaml
micro_version: 2.0.14

tool_deploy_list:
  micro:
    action: download_archive
    github:
      repo: zyedidia/micro
      file: "v{{ micro_version | mandatory }}/micro-{{ micro_version | mandatory }}-linux64-static.tar.gz"
    version:
      args: --version
      match: "Version: {{ micro_version | mandatory }}"
```

### minikube

[kubernetes / minikube](https://github.com/jarun/minikube)

``` yaml
minikube_version: 1.35.0

tool_deploy_list:
  minikube:
    action: download_binary
    github:
      repo: kubernetes/minikube
      file: "v{{ minikube_version | mandatory }}/minikube-linux-amd64"
    version:
      args: version
      match: "minikube version: v{{ minikube_version | mandatory }}"
    bash_completion:
      generate_args: completion bash
    zsh_completion:
      generate_args: completion zsh
```

### moar

[walles / moar](https://github.com/walles/moar)

``` yaml
moar_version: 1.31.5

tool_deploy_list:
  moar:
    action: download_binary
    github:
      repo: walles/moar
      file: "v{{ moar_version | mandatory }}/moar-v{{ moar_version }}-linux-amd64"
    version:
      args: --version
      match: "{{ moar_version | mandatory }}"
    man_pages:
      url:
        - https://raw.githubusercontent.com/walles/moar/refs/heads/master/moar.1
```

### nnn

[jarun / nnn](https://github.com/jarun/nnn)

``` yaml
nnn_version: 5.1

tool_deploy_list:
  nnn:
    action: download_archive
    github:
      repo: jarun/nnn
      file: "v{{ nnn_version | mandatory }}/nnn-nerd-static-{{ nnn_version | mandatory }}.x86_64.tar.gz"
    binary_name_in_archive: nnn-nerd-static
    version:
      args: -V
      match: "{{ nnn_version | mandatory }}"
    man_pages:
      url: "https://raw.githubusercontent.com/jarun/nnn/refs/heads/master/nnn.1"
    bash_completion:
      url: "https://raw.githubusercontent.com/jarun/nnn/refs/heads/master/misc/auto-completion/bash/nnn-completion.bash"
    zsh_completion:
      url: "https://raw.githubusercontent.com/jarun/nnn/refs/heads/master/misc/auto-completion/zsh/_nnn"
```

### neovim / nvim

[neovim / neovim](https://github.com/neovim/neovim)

``` yaml
nvim_version: 0.11.1

tool_deploy_list:
  nvim:
    action: download_appimage
    github:
      repo: neovim/neovim
      file: "v{{ nvim_version | mandatory }}/nvim-linux-x86_64.appimage"
    version:
      args: --version
      match: "NVIM v{{ nvim_version | mandatory }}"
    xdg_desktop:
      src: squashfs-root/nvim.desktop
      icon: squashfs-root/nvim.png
    man_pages:
      src: squashfs-root/usr/share/man/man1/nvim.1
```

### oh-my-posh

[JanDeDobbeleer / oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)

``` yaml
oh_my_posh_version: 26.17.0

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
      src: misc/completions/zsh/_pet
```

### restic

[restic / restic](https://github.com/restic/restic)

``` yaml
restic_version: 0.18.0

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

### sqlitebrowser

[sqlitebrowser / sqlitebrowser](https://github.com/sqlitebrowser/sqlitebrowser)

``` yaml
sqlitebrowser_version: 3.13.1

tool_deploy_list:
  sqlitebrowser:
    action: download_appimage
    github:
      repo: keepassxreboot/sqlitebrowser
      file: "v{{ sqlitebrowser_version }}/DB.Browser.for.SQLite-v{{ sqlitebrowser_version }}-x86_64-v2.AppImage"
    version:
      args: --version
      match: "DB Browser for SQLite Version {{ sqlitebrowser_version | mandatory }}"
    xdg_desktop:
      src: squashfs-root/sqlitebrowser.desktop
      svg: squashfs-root/sqlitebrowser.svg
```

### talosctl

[siderolabs / talos](https://github.com/siderolabs/talos)

``` yaml
talosctl_version: 1.9.4

tool_deploy_list:
  talosctl:
    action: download_binary
    github:
      repo: siderolabs/talos
      file: "v{{ talosctl_version | mandatory }}/talosctl-linux-amd64"
    version:
      args: version --client
      match: "{{ talosctl_version | mandatory }}"
    bash_completion:
      generate_args: completion bash
    zsh_completion:
      generate_args: completion zsh
```

### task

[go-task / task](https://github.com/go-task/task)

``` yaml
task_version: 3.44.1

tool_deploy_list:
  task:
    action: download_archive
    github:
      repo: go-task/task
      file: "v{{ task_version }}/task_linux_amd64.tar.gz"
    version:
      args: --version
      match: "{{ task_version }}"
    bash_completion:
      src: completion/bash/task.bash
    zsh_completion:
      src: completion/zsh/_task
```

### tealdeer

[tealdeer-rs / tealdeer](https://github.com/tealdeer-rs/tealdeer)

``` yaml
tealdeer_version: 1.7.2

tool_deploy_list:
  tealdeer:
    action: download_binary
    github:
      repo: tealdeer-rs/tealdeer
      file: "v{{ tealdeer_version | mandatory }}/tealdeer-linux-x86_64-musl"
    version:
      args: --version
      match: "{{ tealdeer_version | mandatory }}"
    bash_completion:
      url:
        - https://raw.githubusercontent.com/tealdeer-rs/tealdeer/refs/heads/main/completion/bash_tealdeer
    zsh_completion:
      url:
        - https://raw.githubusercontent.com/tealdeer-rs/tealdeer/refs/heads/main/completion/zsh_tealdeer
```

### tig

[jonas / tig](https://github.com/jonas/tig)

``` yaml
tig_version: 2.5.12

tool_deploy_list:
  tig:
    dependencies:
      - build-essential
      - ncurses-dev
      - pkg-config
    action: download_sources
    github:
      repo: jonas/tig
      file: "tig-{{ tig_version | mandatory }}/tig-{{ tig_version | mandatory }}.tar.gz"
    version:
      args: --version
      match: "tig version {{ tig_version | mandatory }}"
    build_steps:
      - ./configure prefix={{ tool_deploy_prefix }}
      - make
      - make install
      - make install-doc-man
```

### tmux

[tmux / tmux](https://github.com/tmux/tmux)

``` yaml
tmux_version: 3.5a

tool_deploy_list:
  tmux:
    dependencies:
      - build-essential
      - ncurses-dev
      - libevent-dev
      - pkg-config
      - bison
    action: download_sources
    github:
      repo: tmux/tmux
      file: "{{ tmux_version | mandatory }}/tmux-{{ tmux_version | mandatory }}.tar.gz"
    version:
      args: -V
      match: "tmux {{ tmux_version | mandatory }}"
    build_steps:
      - ./configure prefix={{ tool_deploy_prefix }}
      - make
      - make install
```

### uv

[astral-sh / uv](https://github.com/astral-sh/uv)

``` yaml
uv_version: 0.7.2

tool_deploy_list:
  uv:
    action: download_archive
    github:
      repo: astral-sh/uv
      file: "{{ uv_version | mandatory }}/uv-x86_64-unknown-linux-gnu.tar.gz"
    version:
      args: --version
      match: "uv {{ uv_version | mandatory }}"
    copy_files:
      - src: uvx
        mode: u=rwx,go=rx
```

### vhs

[charmbracelet / vhs](https://github.com/charmbracelet/vhs)

``` yaml
vhs_version: 0.9.0

tool_deploy_list:
  vhs:
    action: download_archive
    github:
      repo: charmbracelet/vhs
      file: "v{{ vhs_version | mandatory }}/vhs_{{ vhs_version }}_Linux_x86_64.tar.gz"
    version:
      args: --version
      match: "vhs version v{{ vhs_version | mandatory }}"
    man_pages:
      src:
        - manpages/vhs.1.gz
    bash_completion:
      src: completions/vhs.bash
    zsh_completion:
      src: completions/vhs.zsh
```

### xh

[ducaale / xh](https://github.com/ducaale/xh)

``` yaml
xh_version: 0.24.1

tool_deploy_list:
  xh:
    action: download_archive
    github:
      repo: ducaale/xh
      file: "v{{ xh_version | mandatory }}/xh-{{ xh_version }}-x86_64-unknown-linux-musl.tar.gz"
    version:
      args: --version
      match: "xh {{ xh_version | mandatory }}"
    create_links:
      - src: xh
        dest: xhs
    man_pages:
      src: doc/xh.1
    bash_completion:
      src: completions/xh.bash
    zsh_completion:
      src: completions/_xh
```

### xq

[sibprogrammer / xq](https://github.com/sibprogrammer/xq)

``` yaml
xq_version: 1.3.0

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
yq_version: 4.45.2

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
zoxide_version: 0.9.7

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
