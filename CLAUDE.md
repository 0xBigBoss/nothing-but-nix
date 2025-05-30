# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nothing but Nix is a GitHub Action that maximizes available disk space for Nix on GitHub Actions runners by removing pre-installed software. It creates a BTRFS volume at `/nix` that dynamically expands from ~65GB to ~130GB as space is reclaimed.

## Architecture

### Core Components

1. **action.yml** - GitHub Action definition with the following inputs:
   - `hatchet-protocol`: Controls purging level (holster/carve/cleave/rampage)
   - `witness-carnage`: Shows purge progress in real-time (default: false)
   - `root-safe-haven`: MB to reserve on root filesystem (default: 2048)
   - `mnt-safe-haven`: MB to reserve on /mnt filesystem (default: 1024)

2. **The Hatchet Protocol** - Four levels of disk space reclamation:
   - **Holster (0)**: Only use free space from /mnt (~65GB)
   - **Carve (1)**: Combine free space from / and /mnt (~85GB)
   - **Cleave (2)**: Remove Docker, Snap, minimal apt packages (~115GB)
   - **Rampage (3)**: Aggressive removal of all non-essential software (~130GB)

### Implementation Flow

1. **Environment Check**: Validates Ubuntu runner without existing Nix installation
2. **Initial Volume Creation**: Creates BTRFS RAID0 volume using /mnt space
3. **Background Purge Script**: Executes space reclamation based on protocol level
4. **Dynamic Expansion**: Monitors freed space and adds expansion disks to BTRFS pool
5. **Post-Action Reporting**: Displays final disk configuration

### Key Technical Details

- Uses `rmz` from Fast Unix Commands (FUC) for high-performance file deletion
- Creates loop devices for disk images to avoid direct partition manipulation
- BTRFS filesystem with compression (zstd:1) and performance optimizations
- Expansion state tracked in `${HOME}/.expansion/` directory
- Background execution by default to avoid blocking workflows

## Common Development Tasks

Since this is a GitHub Action project, there are no traditional build/test commands. Development involves:

1. Testing changes in a GitHub Actions workflow that uses the action
2. Monitoring disk space reclamation with `df -h` commands
3. Checking expansion progress via files in `~/.expansion/`

## Testing the Action

To test changes, create a workflow that uses the action:

```yaml
- uses: wimpysworld/nothing-but-nix@main
  with:
    hatchet-protocol: 'cleave'
    witness-carnage: true  # Watch progress in real-time
```

## Debugging

- Check `${HOME}/.expansion/expansion.log` for background purge output
- Look for `*_done` files in `${HOME}/.expansion/` to track completion stages
- Use `sudo btrfs filesystem show` to inspect volume configuration