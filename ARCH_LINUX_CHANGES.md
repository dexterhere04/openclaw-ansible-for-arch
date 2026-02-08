# Arch Linux Support Implementation

This document details the changes made to add Arch Linux support to the clawdbot-ansible project while maintaining full backwards compatibility with Debian/Ubuntu and macOS.

## Overview

Arch Linux support has been added with complete feature parity including:
- System tools installation
- Docker installation and configuration
- Node.js and pnpm setup
- Tailscale VPN support
- UFW firewall with DOCKER-USER isolation
- Security hardening (fail2ban)

## Files Modified

### 1. install.sh
**Lines Changed: 2**

- **Line 68**: Fixed syntax error
  ```bash
  # Before: if [["OS_TYPE"=="arch"]]; then
  # After:  if [[ "$OS_TYPE" == "arch" ]]; then
  ```

- **Line 73**: Added missing `fi` to complete the if statement

- **Line 45**: Updated error message
  ```bash
  # Before: "This installer supports: Debian/Ubuntu and macOS"
  # After:  "This installer supports: Debian/Ubuntu, Arch Linux, and macOS"
  ```

### 2. roles/clawdbot/defaults/main.yml
**Lines Changed: 1**

- **Line 22**: Updated `package_manager` variable to handle three OS types
  ```yaml
  # Before: package_manager: "{{ 'brew' if ansible_os_family == 'Darwin' else 'apt' }}"
  # After:  Conditional logic for Darwin/Archlinux/Debian
  package_manager: >-
    {%- if ansible_os_family == 'Darwin' -%}
    brew
    {%- elif ansible_os_family == 'Archlinux' -%}
    pacman
    {%- else -%}
    apt
    {%- endif -%}
  ```

### 3. roles/clawdbot/tasks/system-tools.yml
**Lines Changed: 4**

- **Lines 8-10**: Added Arch Linux condition
- **Line 14**: Updated unsupported OS message to include Arch

### 4. roles/clawdbot/tasks/docker.yml
**Lines Changed: 3**

- **Lines 7-9**: Added Arch Linux condition

### 5. roles/clawdbot/tasks/nodejs.yml
**Lines Changed: 11**

- **Completely restructured** to use conditional includes like other task files
- **Lines 4-11**: Added conditions for Debian, Arch, and macOS

### 6. roles/clawdbot/tasks/tailscale.yml
**Lines Changed: 3**

- **Lines 7-9**: Added Arch Linux condition

### 7. roles/clawdbot/tasks/firewall.yml
**Lines Changed: 3**

- **Lines 7-9**: Added Arch Linux condition

## Files Created

### 1. roles/clawdbot/tasks/system-tools-arch.yml
**New file: 119 lines**

- Uses `ansible.builtin.pacman` module
- Package mappings for Arch Linux repositories
- Vim configuration path: `/etc/vimrc` (Debian uses `/etc/vim/vimrc.local`)
- Identical shell and user configuration

### 2. roles/clawdbot/tasks/docker-arch.yml
**New file: 17 lines**

- Simplified installation using official Arch repositories
- No external GPG keys or repositories needed
- Install packages: `docker`, `docker-compose`, `docker-buildx`
- Same systemd service and user management

### 3. roles/clawdbot/tasks/nodejs-arch.yml
**New file: 22 lines**

- Uses official Arch Node.js packages
- No external NodeSource repository needed
- Install packages: `nodejs`, `npm`
- Same pnpm global installation

### 4. roles/clawdbot/tasks/tailscale-arch.yml
**New file: 31 lines**

- Uses official Arch Tailscale package
- Same systemd service management
- Arch-specific user guidance

### 5. roles/clawdbot/tasks/firewall-arch.yml
**New file: 96 lines**

- Install: `ufw`, `fail2ban` from official repos
- **Skip unattended-upgrades** (Arch is rolling release, user opted out)
- Same UFW + DOCKER-USER security model
- Same systemd service management

### 6. roles/clawdbot/tasks/nodejs-linux.yml
**New file: 65 lines**

- Moved original Debian/Ubuntu Node.js logic from `nodejs.yml`
- Maintains existing NodeSource repository setup

### 7. roles/clawdbot/tasks/nodejs-macos.yml
**New file: 28 lines**

- macOS-specific Node.js installation via Homebrew
- Extracted from restructuring process

## Package Mappings

### System Tools (Debian → Arch)
```yaml
# Build tools
build-essential → base-devel

# Network tools
netcat-openbsd → openbsd-netcat
iputils-ping → iputils
dnsutils → bind-tools
procps → procps-ng
inetutils → inetutils (telnet replacement)

# All other packages have same names
zsh, vim, nano, git, git-lfs, curl, wget, net-tools
traceroute, tcpdump, nmap, socat, strace, lsof, gdb
htop, iotop, iftop, sysstat, tmux, tree, jq, unzip, rsync, less, file
```

### Docker (Debian external repo → Arch official repo)
```yaml
# Debian: docker-ce, docker-ce-cli, containerd.io, docker-buildx-plugin, docker-compose-plugin
# Arch:   docker, docker-compose, docker-buildx
```

### Node.js (NodeSource repo → Arch official repo)
```yaml
# Debian: nodejs (from NodeSource)
# Arch:   nodejs, npm (from official repos)
```

### Firewall (Skip some packages on Arch)
```yaml
# Debian: ufw, fail2ban, unattended-upgrades, apt-listchanges
# Arch:   ufw, fail2ban (unattended-upgrades skipped per user decision)
```

## Security Considerations

### Maintained Security Model
- **UFW + DOCKER-USER**: Identical security isolation pattern preserved
- **Package Trust**: Uses Arch's package signing instead of external GPG keys
- **Service Management**: Same systemd approach ensures consistent security
- **Container Security**: Non-root containers and localhost binding maintained

### Security Simplifications
- **No External Repositories**: Reduces attack surface by using official Arch repos
- **Package Signing**: Leverages Arch's trusted package signing infrastructure
- **Simplified GPG**: No external key management needed

## OS Detection Logic

The implementation uses Ansible's built-in OS detection:
```yaml
is_arch: "{{ ansible_os_family == 'Archlinux' }}"
```

This works for:
- Arch Linux
- Arch-based distributions (Manjaro, EndeavourOS, etc.)

## Testing Checklist

### Syntax Validation
- [x] Ansible playbook syntax check: `ansible-playbook playbook.yml --syntax-check`
- [x] Install script syntax check: `bash -n install.sh`

### OS Detection
- [x] Conditional logic for all three OS families
- [x] Correct task file inclusion based on OS family

### Package Availability
- [x] All Arch packages verified in official repositories
- [x] Package name mappings tested and confirmed

### Backwards Compatibility
- [x] Existing Debian/Ubuntu functionality preserved
- [x] Existing macOS functionality preserved
- [x] No breaking changes introduced

## Migration Impact

### For Existing Users
- **Zero Impact**: Existing Debian/Ubuntu and macOS users see no changes
- **No Migration Required**: Existing installations continue to work
- **Same Commands**: All management commands remain identical

### For New Arch Users
- **Same Installation Process**: `curl | bash` works identically
- **Same User Experience**: All documentation and commands apply
- **Same Security Model**: Identical firewall and container isolation

## File Structure After Changes

```
roles/clawdbot/tasks/
├── main.yml
├── system-tools.yml           # Updated with Arch condition
├── system-tools-linux.yml
├── system-tools-arch.yml     # NEW
├── system-tools-macos.yml
├── docker.yml                 # Updated with Arch condition
├── docker-linux.yml
├── docker-arch.yml           # NEW
├── docker-macos.yml
├── nodejs.yml                 # Restructured for includes
├── nodejs-linux.yml          # NEW (moved content)
├── nodejs-arch.yml           # NEW
├── nodejs-macos.yml          # NEW
├── tailscale.yml              # Updated with Arch condition
├── tailscale-linux.yml
├── tailscale-arch.yml        # NEW
├── tailscale-macos.yml
├── firewall.yml               # Updated with Arch condition
├── firewall-linux.yml
├── firewall-arch.yml         # NEW
├── firewall-macos.yml
├── user.yml
└── clawdbot.yml
```

## Risk Assessment

### LOW RISK
- **Additive Changes**: All modifications are new functionality
- **Conditional Execution**: Arch code only runs on Arch systems
- **Isolation**: Existing OS paths completely unaffected
- **Testing**: Comprehensive syntax and logic validation completed

### Breaking Changes: None
- **Backwards Compatible**: 100% compatibility maintained
- **No Required Updates**: Existing users don't need to change anything
- **Same Interface**: All documentation and commands remain identical

## Summary

This implementation successfully adds full Arch Linux support to the clawdbot-ansible project while maintaining complete backwards compatibility. The security model, user experience, and feature parity are preserved across all three supported platforms (Debian/Ubuntu, Arch Linux, and macOS).

The implementation leverages Arch Linux's official repositories and package signing infrastructure, reducing complexity while maintaining security. Users on Arch Linux now have access to the same automated, hardened Clawdbot installation experience as users on other platforms.