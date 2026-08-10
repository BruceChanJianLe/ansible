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

| File                | Role                                                    |
| ------------------- | ------------------------------------------------------- |
| `vm-test.yml`       | Reusable (`workflow_call`). All the logic lives here.    |
| `ubuntu-<ver>.yml`  | Thin caller, one per Ubuntu release.                     |
| `archlinux.yml`     | Thin caller for Arch.                                    |

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
- **macOS: unsupported, no workflow.** The Darwin task files live on the
  `macos` branch and have never been merged. Note that this VM approach would
  not transfer: GitHub's macOS runners do not expose nested virtualisation, so
  a macOS target would have to run `ansible-pull` directly on a `macos-*`
  runner, which is a different design. Leave the README row as "not supported"
  until that work is actually done.

## Gotchas

- **Badge URLs are pinned to a branch.** They carry `?branch=u26-test`. When
  this merges to master, update the query string in both the image URL and the
  link URL of every row, or the table will keep reporting a stale branch.
- **The become password goes in via `-e @file`, and must stay that way.**
  `--become-password-file` / `ANSIBLE_BECOME_PASSWORD_FILE` arrived in
  ansible-core 2.12, but Ubuntu 22.04 ships ansible 2.10.8, where they are
  silently ignored - every privileged task then fails with
  "sudo: a password is required". The oldest supported target sets the floor
  here; re-check this if the support window ever moves.
- **`/dev/kvm` is not on every runner.** The hosted pool is not homogeneous, so
  a job can land on a host without nested virtualisation and fail the very
  first step in ~10s with a silent exit 1 (that is `test -w /dev/kvm`). Re-run
  the job; it is a lottery, not a code change.
- **`ansible-pull` fetches from github.com, not from the runner's checkout.**
  A fix has to be pushed before CI can test it. There is no way around this
  without abandoning `ansible-pull` as the thing under test.
- **`paths-ignore` skips doc-only pushes.** Editing `**.md` or `LICENSE` alone
  will not spin up the VMs, and the badges keep their previous state. Remove it
  if you ever need a run on a docs change.
- **KVM is asserted, not assumed.** `test -w /dev/kvm` fails the job early;
  without it QEMU would silently fall back to software emulation, run ~20x
  slower, and simply hit the 90 minute timeout instead of telling you why.
