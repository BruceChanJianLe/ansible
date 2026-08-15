# CI notes

Read this before adding, removing or changing a target.

## What these workflows do

Each target boots a **pristine cloud image under QEMU/KVM** and runs the
documented one-liner inside it:

```bash
ansible-pull -U https://github.com/brucechanjianle/ansible -C <branch>
```

as a normal user in the admin group whose `sudo` **asks for a password** (there
is deliberately no `NOPASSWD` rule). That is the closest thing to a brand-new
laptop that CI can produce.

Containers are not an option here and should not be reintroduced: the playbook
installs snaps (ghostty, nvim, tmux) and the multi-user nix daemon, both of
which need a real kernel and systemd.

## Layout

| File                | Role                                                                        |
| ------------------- | --------------------------------------------------------------------------- |
| `vm-test.yml`       | Reusable (`workflow_call`). All the Linux VM logic lives here.              |
| `ubuntu-<ver>.yml`  | Thin caller, one per Ubuntu release.                                        |
| `archlinux.yml`     | Thin caller for Arch.                                                       |
| `macos.yml`         | Standalone macOS workflow. Runs `ansible-pull` directly on `macos-latest`. |

**Why one file per target instead of one matrix job:** a GitHub status badge is
scoped to a workflow *file*, not to a matrix leg. A single matrixed workflow can
only ever produce one badge for all targets. The status table in the top-level
`README.md` needs a badge per row, so each row needs its own workflow file. The
callers are ~20 lines each and contain no logic, so the duplication is cosmetic.

## Adding a target

1. Copy the nearest `ubuntu-<ver>.yml` and change the version, codename and
   label. Check the image URL actually resolves first:

   ```bash
   curl -sIL -o /dev/null -w '%{http_code}\n' <image_url>
   ```

2. Add a row to the status table in the top-level `README.md`, pointing the
   badge at the new workflow filename.
3. If the target is a new *distro* rather than a new version, `vm-test.yml`
   also needs a `case` arm in the **Seed cloud-init** step (admin group,
   whether to full-upgrade first) and in the **Verify installed programs** step
   (what is expected to be installed).

## Support policy and housekeeping

- **Ubuntu: the three most recent LTS releases.** Non-LTS releases are not
  tested - they are short-lived and nobody sets up a laptop on them.
  When a new LTS ships, add it and **delete the oldest** in the same change, so
  the window stays at three. Next rotation: when **28.04** ships (April 2028),
  drop **22.04** - delete `ubuntu-22.04.yml` and its README row.
- **Arch: rolling, always latest image.** No version to pin, nothing to rotate.
  Arch does *not* install brave: it is commented out of
  `setup-additional-Archlinux.yml` in favour of building `brave-bin` with paru,
  so the Arch checks assert ghostty, nix and spotify-launcher only.
- **macOS: `macos-latest` runner, no VM.** GitHub's macOS runners do not
  expose nested virtualisation, so `macos.yml` runs `ansible-pull` directly
  on the runner instead of inside QEMU. The runner user has passwordless sudo,
  so no become-password setup is needed. It is a peer of the Linux callers, not
  a caller of `vm-test.yml`, because there is no VM to boot.

## Gotchas

- **On master these run weekly, not on push.** All five carry
  `schedule: - cron: "0 12 * * 5"` - Friday 20:00 SGT, since GitHub cron is
  always UTC. Fast feedback comes from the branch-level workflows on
  `u26-test` / `macos-test`, which keep their push triggers; master's job is to
  catch upstream drift in things nobody here controls (the nix installer,
  Brave's apt repo, the snap channels, Arch rolling forward).
- **`schedule` only ever fires on the default branch.** A cron on any other
  branch is inert. That is also why `workflow_dispatch` finally works now -
  it has the same default-branch requirement.
- **Scheduled runs are best-effort.** GitHub delays them under load and can
  drop them outright, so a missing Friday run is not a signal. Worse, it
  **disables scheduled workflows entirely after 60 days of repo inactivity** -
  if this repo goes quiet, the weekly net stops silently and you re-enable it
  from the Actions tab.
- **Badges are pinned to a branch, and must agree with the triggers.** All five
  `?branch=` / `?query=branch%3A` query strings say `master` here. Repoint both
  halves of every row whenever this content moves, or the table silently
  reports a branch it is not running on.
- **Arch must never bootstrap through cloud-init.** cloud-init is a Python
  program. On a rolling release its `package_upgrade` pulls in a new `python`
  (and a new `cloud-init`) and replaces them *underneath the running process*
  mid-boot; anything Python launched during that window dies with
  `LookupError: no codec search functions registered`. That is why Arch gets
  its own `Bootstrap Arch` step running `pacman -Syu --noconfirm --needed
  ansible git` over SSH after cloud-init has finished - one transaction, never
  `-Sy` followed by an install, and driven by pacman, which is C and can
  therefore replace python without breaking itself.
- **Wait on `/var/lib/cloud/instance/boot-finished`, not `cloud-init status
  --wait`.** sshd is up long before cloud-init finishes, and that CLI is Python,
  so it hits the same trap as above. The marker file is pure filesystem.
- **The become password goes in via `-e @file`, and must stay that way.**
  `--become-password-file` / `ANSIBLE_BECOME_PASSWORD_FILE` arrived in
  ansible-core 2.12, but Ubuntu 22.04 ships ansible 2.10.8, where they are
  silently ignored - every privileged task then fails with
  "sudo: a password is required". The oldest supported target sets the floor
  here; re-check this if the support window ever moves.
- **Enabling KVM is a race unless you `udevadm settle`.** `/dev/kvm` starts as
  `root:kvm 0660` and the runner user is not in the `kvm` group, so it must be
  opened up. `udevadm trigger` merely queues the event, so a bare
  `test -w /dev/kvm` immediately afterwards intermittently fails in ~70ms with
  a silent exit 1. Hence `settle`, plus a direct `chmod` backstop that cannot
  race, plus an explicit up-front check for the device being absent - which is
  the one real "this runner has no nested virtualisation" case.
  Order matters too: host tooling is installed **first**, because
  `qemu-system-common` ships its own `/dev/kvm` udev rules and can otherwise
  undo the permissions.
- **`ansible-pull` fetches from github.com, not from the runner's checkout.**
  A fix has to be pushed before CI can test it. There is no way around this
  without abandoning `ansible-pull` as the thing under test.
- **`paths-ignore` is gone from master, deliberately.** Path filters only apply
  to `push` / `pull_request` and are silently ignored by `schedule`, so keeping
  one here would have implied a doc-only exemption that does not exist. The
  branch-level copies still carry it, where it does work.
- **KVM is asserted, not assumed.** `test -w /dev/kvm` fails the job early;
  without it QEMU would silently fall back to software emulation, run ~20x
  slower, and simply hit the 90 minute timeout instead of telling you why.
