# Nothing but Space - Generalized Space Reclamation for GitHub Actions

## Executive Summary

Transform the Nix-specific space reclamation approach into a general-purpose GitHub Action that provides 65-130GB of additional space for any workflow need.

## Core Concept

Leverage the proven space reclamation techniques from "Nothing but Nix" but make the destination configurable:
- **Workspace builds**: Large compilation projects, monorepos
- **Docker storage**: Building large container images
- **Dependency cache**: Node modules, Maven, pip packages
- **Custom paths**: User-defined mount points

## Proposed Action Configuration

```yaml
- uses: orgname/nothing-but-space@main
  with:
    # Where to put the reclaimed space
    space-target: 'workspace'  # Options: workspace, docker, cache, custom
    
    # Custom mount path (only for 'custom' target)
    mount-path: '/build'
    
    # Paths to symlink into the new volume
    symlink-paths:
      - from: '${{ github.workspace }}'
        to: '/workspace/repo'
    
    # Existing options from Nothing but Nix
    hatchet-protocol: 'cleave'
    witness-carnage: false
    root-safe-haven: '2048'
    mnt-safe-haven: '1024'
```

## Target Modes

### 1. Workspace Mode (Default)
- Mount point: `/workspace`
- Use case: Build artifacts, compilation, large repos
- Auto-symlinks: `$GITHUB_WORKSPACE` → `/workspace/repo`

### 2. Docker Mode
- Mount point: `/var/lib/docker`
- Use case: Building large container images
- Special handling:
  - Stop Docker daemon
  - Move existing Docker data
  - Mount new volume
  - Restart Docker daemon

### 3. Cache Mode
- Mount point: `/cache`
- Use case: Dependency caching, build caches
- Common symlinks:
  - `~/.npm` → `/cache/npm`
  - `~/.m2` → `/cache/maven`
  - `~/.cache` → `/cache/general`

### 4. Custom Mode
- Mount point: User-specified
- Use case: Specific workflow needs
- Full control over symlinks

## Implementation Strategy

### Phase 1: Core Refactoring
1. Fork "Nothing but Nix" repository
2. Abstract volume creation logic
3. Make mount point configurable
4. Keep all purging logic intact

### Phase 2: Target-Specific Logic
1. Implement Docker mode with daemon management
2. Add intelligent symlink creation
3. Create post-mount hooks for service restarts
4. Add validation for target modes

### Phase 3: Enhanced Features
1. Multiple volume support (e.g., both workspace AND cache)
2. Compression options per target
3. Performance profiles per use case
4. Space usage reporting

## Example Workflows

### Large Docker Build
```yaml
- uses: orgname/nothing-but-space@main
  with:
    space-target: 'docker'
    hatchet-protocol: 'rampage'  # Maximum space for images

- name: Build Large Image
  run: |
    docker build -t myapp:latest .
    docker images  # Will show available space
```

### Node.js Monorepo
```yaml
- uses: orgname/nothing-but-space@main
  with:
    space-target: 'workspace'
    symlink-paths:
      - from: '${{ github.workspace }}'
        to: '/workspace/repo'
      - from: '~/.npm'
        to: '/workspace/npm-cache'

- name: Install Dependencies
  run: |
    cd /workspace/repo
    npm install  # Plenty of space for node_modules
```

### Machine Learning Pipeline
```yaml
- uses: orgname/nothing-but-space@main
  with:
    space-target: 'custom'
    mount-path: '/ml-data'
    hatchet-protocol: 'rampage'

- name: Download Dataset
  run: |
    wget -P /ml-data https://example.com/large-dataset.tar.gz
    tar -xzf /ml-data/large-dataset.tar.gz -C /ml-data
```

## Benefits Over Current Solution

1. **Broader Appeal**: Not limited to Nix users
2. **Flexible Targets**: Users choose where space goes
3. **Docker Integration**: Solves common "no space left" errors
4. **Backward Compatible**: Can still create /nix for Nix users
5. **Progressive Enhancement**: Start simple, add features as needed

## Migration Path

For existing "Nothing but Nix" users:
```yaml
# Old way
- uses: wimpysworld/nothing-but-nix@main

# New way (equivalent)
- uses: orgname/nothing-but-space@main
  with:
    space-target: 'custom'
    mount-path: '/nix'
```

## Technical Considerations

1. **BTRFS on Loop Devices**: Keep this approach - it's elegant and avoids repartitioning
2. **Dynamic Expansion**: Maintain background expansion capability
3. **Service Management**: Carefully handle Docker/other service restarts
4. **Error Handling**: Graceful fallbacks if target mode setup fails
5. **Compatibility**: Test across Ubuntu runner versions

## Success Metrics

- Adoption by non-Nix projects
- Reduction in "disk space" workflow failures  
- GitHub marketplace ratings
- Community contributions for new target modes

## Next Steps

1. Validate approach with potential users
2. Create proof-of-concept for Docker mode
3. Benchmark performance across different targets
4. Write comprehensive documentation
5. Plan migration strategy for existing users