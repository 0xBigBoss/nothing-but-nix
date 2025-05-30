# Nothing but Space - Example Workflows

## 1. Docker Build with Large Images

```yaml
name: Build Large Docker Images
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Maximize space for Docker
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'docker'
          hatchet-protocol: 'rampage'  # Maximum space reclamation
          
      - name: Build multi-stage image
        run: |
          docker build -t myapp:latest .
          docker images
          df -h /var/lib/docker
```

## 2. Node.js Monorepo with Heavy Dependencies

```yaml
name: Node.js Monorepo Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create large workspace
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'workspace'
          hatchet-protocol: 'cleave'
          symlink-config: |
            [
              {"from": "${{ github.workspace }}", "to": "/workspace/repo"},
              {"from": "$HOME/.npm", "to": "/workspace/npm-cache"},
              {"from": "$HOME/.pnpm-store", "to": "/workspace/pnpm-store"}
            ]
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        working-directory: /workspace/repo
        run: |
          npm ci
          npx lerna bootstrap
          
      - name: Build all packages
        working-directory: /workspace/repo
        run: npm run build:all
```

## 3. Machine Learning with Large Datasets

```yaml
name: ML Training Pipeline
on: [push]

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create ML data volume
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'custom'
          mount-path: '/ml-workspace'
          hatchet-protocol: 'rampage'
          compression: 'lzo'  # Faster for ML workloads
          
      - name: Download datasets
        run: |
          mkdir -p /ml-workspace/data
          wget -P /ml-workspace/data https://example.com/dataset1.tar.gz
          wget -P /ml-workspace/data https://example.com/dataset2.tar.gz
          tar -xzf /ml-workspace/data/*.tar.gz -C /ml-workspace/data
          
      - name: Train model
        run: |
          python train.py \
            --data-dir /ml-workspace/data \
            --output-dir /ml-workspace/models \
            --cache-dir /ml-workspace/cache
```

## 4. Android Build with SDK and Gradle Cache

```yaml
name: Android Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Maximize build space
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'workspace'
          hatchet-protocol: 'cleave'
          symlink-config: |
            [
              {"from": "${{ github.workspace }}", "to": "/workspace/repo"},
              {"from": "$HOME/.gradle", "to": "/workspace/gradle-cache"},
              {"from": "$HOME/.android", "to": "/workspace/android-sdk"}
            ]
            
      - name: Setup JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          
      - name: Build APK
        working-directory: /workspace/repo
        run: |
          ./gradlew assembleRelease
          ls -la app/build/outputs/apk/
```

## 5. Rust Project with Large Dependencies

```yaml
name: Rust Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create cargo cache volume
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'cache'
          hatchet-protocol: 'carve'  # Moderate space, keep some tools
          auto-symlinks: true  # Will auto-link common cache dirs
          
      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
        
      - name: Build project
        run: |
          cargo build --release
          cargo test
          
      - name: Show cache usage
        run: |
          du -sh ~/.cargo
          du -sh target/
          df -h /cache
```

## 6. Multi-Purpose Build Environment

```yaml
name: Complex Multi-Tool Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup build environment
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'custom'
          mount-path: '/build'
          hatchet-protocol: 'cleave'
          symlink-config: |
            [
              {"from": "${{ github.workspace }}", "to": "/build/source"},
              {"from": "$HOME/.cargo", "to": "/build/cargo"},
              {"from": "$HOME/.npm", "to": "/build/npm"},
              {"from": "$HOME/.cache", "to": "/build/cache"},
              {"from": "/tmp", "to": "/build/tmp"}
            ]
            
      - name: Run complex build
        working-directory: /build/source
        run: |
          # Frontend build
          cd frontend && npm ci && npm run build
          
          # Backend build  
          cd ../backend && cargo build --release
          
          # Package everything
          cd .. && ./scripts/package.sh
```

## 7. Database Testing with Large Datasets

```yaml
name: Database Integration Tests
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          
    steps:
      - uses: actions/checkout@v4
      
      - name: Create data volume
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'custom'
          mount-path: '/dbdata'
          hatchet-protocol: 'cleave'
          
      - name: Load test data
        run: |
          # Download large test datasets
          wget -P /dbdata https://example.com/test-data.sql.gz
          gunzip /dbdata/test-data.sql.gz
          
          # Import into PostgreSQL
          PGPASSWORD=postgres psql -h localhost -U postgres < /dbdata/test-data.sql
          
      - name: Run integration tests
        run: |
          npm test:integration
```

## 8. Legacy Nix Compatibility

```yaml
name: Nix Build (Backward Compatible)
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Drop-in replacement for nothing-but-nix
      - name: Maximize Nix store space
        uses: orgname/nothing-but-space@main
        with:
          space-target: 'nix'  # Equivalent to original behavior
          hatchet-protocol: 'cleave'
          
      - name: Install Nix
        uses: DeterminateSystems/nix-installer-action@main
        
      - name: Build with Nix
        run: |
          nix build
          nix-store --query --requisites --include-outputs ./result
```

## Tips for Choosing Options

### Hatchet Protocol Selection
- **holster**: When you need other GitHub Actions tools to work
- **carve**: When you want more space but still need apt packages
- **cleave**: Best balance for most use cases (default)
- **rampage**: Maximum space, when nothing else matters

### Space Target Selection
- **workspace**: General builds, compilation, artifacts
- **docker**: Container image builds
- **cache**: Dependency management, faster CI
- **custom**: Specific needs with full control
- **nix**: Backward compatibility with nothing-but-nix

### Compression Selection
- **zstd:1**: Best balance (default)
- **lzo**: Faster but less compression
- **none**: Maximum performance, no compression