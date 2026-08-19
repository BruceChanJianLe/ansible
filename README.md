[![macOS](https://github.com/brucechanjianle/ansible/actions/workflows/macos.yml/badge.svg?branch=master)](https://github.com/brucechanjianle/ansible/actions/workflows/macos.yml?query=branch%3Amaster)
[![Arch Linux](https://github.com/brucechanjianle/ansible/actions/workflows/archlinux.yml/badge.svg?branch=master)](https://github.com/brucechanjianle/ansible/actions/workflows/archlinux.yml?query=branch%3Amaster)
[![Ubuntu 26.04](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-26.04.yml/badge.svg?branch=master)](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-26.04.yml?query=branch%3Amaster)
[![Ubuntu 24.04](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-24.04.yml/badge.svg?branch=master)](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-24.04.yml?query=branch%3Amaster)
[![Ubuntu 22.04](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-22.04.yml/badge.svg?branch=master)](https://github.com/brucechanjianle/ansible/actions/workflows/ubuntu-22.04.yml?query=branch%3Amaster)

> This script is a quick way to setup your brand new laptop,
> or your docker containers in a swift manner. It currently supports
> Ubuntu, Arch, and macOS!

# Quick Start

**1. Install dependencies**

| OS     | Command                                |
| ------ | -------------------------------------- |
| Ubuntu | `sudo apt install ansible git -y`      |
| Arch   | `sudo pacman -S ansible git --noconfirm` |
| macOS  | `brew install ansible git`             |

**2. Run it**

```bash
ansible-pull -U https://github.com/brucechanjianle/ansible --ask-become-pass
```

Note that additional setup is skipped on arm architecture.

<details>
<summary><strong>macOS the nix way (nix-darwin)</strong></summary>

Packages and macOS defaults come from `~/.config/nix-darwin` instead of ansible.
Install nix first:

```bash
curl -fsSL https://install.determinate.systems/nix | sh -s -- install
. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
```

Then one liner - no need for `brew install ansible git`, the temporary shell
provides both:

```bash
nix-shell -p ansible git --run \
    'ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "decrypt_cjl=true" --tags nix_darwin --ask-become-pass'
```

The `nix_darwin` tag is required - the track is tagged `never`, so it is opt-in
and a plain `ansible-pull` will not touch it.

Rebuild after editing the flake:

```bash
sudo darwin-rebuild switch --flake ~/.config/nix-darwin#$USER
```

</details>

<details>
<summary><strong>Other setup options</strong></summary>

**Partial setup**, to be discreet! Works for docker/podman as well:

```bash
ansible-pull -U https://github.com/brucechanjianle/ansible --skip-tags additional --ask-become-pass
```

**Full setup with decryption** (for all you supporters out there):

```bash
ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "decrypt_bri=true" --ask-become-pass
```

**For others** - setting up for a specific group of users defined in the
inventory file. Clone the repository locally to use this, as you will need to
edit `inventory`:

```bash
ansible-playbook local.yml --ask-become-pass -l others
```

</details>

<details>
<summary><strong>Tags</strong></summary>

| Tag                | Notes                        |
| ------------------ | ---------------------------- |
| `setup`            | all                          |
| `core`             |                              |
| `ssh`              |                              |
| `base16`           |                              |
| `nvim`             |                              |
| `tmux`             |                              |
| `zsh`              |                              |
| `dotfiles`         |                              |
| `private_aliases`  |                              |
| `brave`            |                              |
| `nix`              |                              |
| `ghostty`          |                              |
| `spotify`          | Arch only                    |
| `hyprland`         | Arch only                    |
| `additional`       | not meant for containers     |
| `nix_darwin`       | macOS nix-darwin track, opt-in |

</details>

<details>
<summary><strong>Post-installation (optional, Linux)</strong></summary>

Set ghostty as your default terminal:

```bash
sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/local/bin/ghostty 100
```

Set nvim as your default editor:

```bash
sudo update-alternatives --install /usr/bin/editor editor /usr/local/bin/nvim 100
```

</details>

<details>
<summary><strong>Reference</strong></summary>

- [A good read](https://wearenotch.com/speed-up-ansible-playbook-execution/#:~:text=The%20first%20time%20a%20playbook,due%20to%20Ansible's%20idempotence%20checking.)
- [ansible_os_family var](https://groups.google.com/g/ansible-project/c/OZPu-b17n_w)
- [CI notes](.github/workflows/README.md)

</details>
