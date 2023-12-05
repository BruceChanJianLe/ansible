tags:
  - setup
  - ssh
  - dotfiles

tasks:
  - ssh.yml
    - setup
    - ssh
    - dotfiles
  - dotfiles.yml

ansible-playbook local.yaml -e "enable_decryption=true" --ask-vault-pass
