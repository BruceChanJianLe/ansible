> This script is a quick way to setup your brand new laptop,
> or your docker containers in a swift manner. It currently supports
> Ubuntu and Arch!

tags:
  - setup (all)
  - core
  - ssh
  - base16
  - nvim
  - tmux
  - zsh
  - dotfiles
  - private_aliases
  - brave
  - nix
  - ghostty
  - additional (not meant for containers)

# Install dependencies

## Ubuntu

```bash
sudo apt install ansible git -y
```

## Arch

```bash
sudo pacman -S ansible git --noconfirm
```

# Setting Up New Laptop

## Full Setup without decryption

(For anyone who wants to try) Note that additional setup will be skipped for
arm architecture.

```bash
ansible-pull -U https://github.com/brucechanjianle/ansible --ask-become-pass
```

## Full Setup with decryption

(For you know who you are:)

```bash
ANSIBLE_ASK_VAULT_PASS=True ansible-pull -U https://github.com/brucechanjianle/ansible --ask-vault-pass -e "enable_decryption=true" --ask-become-pass
```

## Partial Setup

Partial setup, to be discreet! Works for docker/podman as well!

```bash
ansible-pull -U https://github.com/brucechanjianle/ansible --skip-tags additional --ask-become-pass
```

## For Others

Setting up for a specific group of users defined in inventory file.
Please clone the repository locally to use this feature as you will need to
edit the inventory file.

```bash
ansible-playbook local.yml --ask-become-pass -l others
```

# Post-Installation (Optional)

Set ghostty as your default terminal!

```bash
sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator /usr/local/bin/ghostty 100
```

Set nvim as your default editor!
```bash
sudo update-alternatives --install /usr/bin/editor editor /usr/local/bin/nvim 100
```

# Reference
- [A good read](https://wearenotch.com/speed-up-ansible-playbook-execution/#:~:text=The%20first%20time%20a%20playbook,due%20to%20Ansible's%20idempotence%20checking.)
- [ansible_os_family var](https://groups.google.com/g/ansible-project/c/OZPu-b17n_w)
