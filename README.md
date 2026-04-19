# Arch Btrfs + LUKS Ansible Playbook

An Ansible playbook that installs Arch Linux in a VM with a GPT disk layout, LUKS-encrypted root, Btrfs subvolumes, and systemd-boot. It is designed to be run from an Arch live environment against a target VM disk.

## What it does
- Partitions `/dev/{{ diskdevice }}` with GPT and creates an EFI system partition plus an encrypted root partition.
- Creates a LUKS container, formats it as Btrfs, and creates subvolumes `@`, `@home`, `@snapshots`.
- Installs a base Arch system with `pacstrap`, generates `fstab`, and enables NetworkManager.
- Sets hostname, timezone, locale, and console settings.
- Creates a user with wheel sudo access and sets a password.
- Configures `mkinitcpio` hooks for an encrypted root and rebuilds initramfs.
- Installs systemd-boot and writes boot entries for the encrypted root.
- Installs `yay` from AUR and cleans up build artifacts.

## Requirements
- Arch Linux live environment with `ansible` available.
- Target disk is empty or disposable. This playbook **destroys all data** on `/dev/{{ diskdevice }}`.
- Required collections:
  - `community.general`
  - `community.crypto`
  - `ansible.posix`

You can install collections with:

```bash
ansible-galaxy collection install community.general community.crypto ansible.posix
```

## Usage
Run the playbook from the Arch live environment, targeting the VM host (often `localhost` for local execution):

```bash
ansible-playbook -i localhost, -c local archbtrfssetup.yml
```

You will be prompted for:
- `luks_passhrase`
- `setup_username`
- `setup_password`

## Variables
Defined in `archbtrfssetup.yml`:
- `diskdevice`: default `vda`
- `hostname`: default `archbtrfs`

Override with `-e`:

```bash
ansible-playbook -i localhost, -c local archbtrfssetup.yml -e diskdevice=sda -e hostname=myarch
```

## Structure
- `archbtrfssetup.yml`: top-level playbook
- `roles/10.filesystems`: partitioning, LUKS, Btrfs subvolumes, mounts
- `roles/20.arch-initial`: mirrorlist, pacstrap, fstab, NetworkManager
- `roles/30.hostname`: hostname and hosts file
- `roles/40.locale`: timezone, locale, console settings
- `roles/50.user`: user creation and sudo
- `roles/60.mkinitcpio`: mkinitcpio hooks for encrypted root
- `roles/65.bootloader`: systemd-boot entries
- `roles/70.yay`: build and install yay

## Notes and caveats
- The playbook is partially written by an LLM, but with oversight of an experienced in a professional setting Ansible user.
- The initial README is completely written by an LLM to reduce time and have more fun with Ansible.
- The EFI partition mount uses `/dev/vda1` in `roles/10.filesystems`. If you change `diskdevice`, update that task or it will still mount `vda1`.
- `reflector` is configured for the Netherlands and mirrors within the last 6 hours.
- The playbook assumes systemd-boot and does not install GRUB.
- `MAKEFLAGS` is set to `-j4` in `/etc/makepkg.conf`.
