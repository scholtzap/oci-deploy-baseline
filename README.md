# OCI deploy baseline (shared)

**Repo:** https://github.com/scholtzap/oci-deploy-baseline  
**Composite action:** `scholtzap/oci-deploy-baseline@v1` (root `action.yml`)

## Why

Prefer **CI checkout + rsync/SCP over SSH** so production VMs never hold a GitHub PAT. Same pattern for Metabase, Avocado legal pipeline, OpenClaw, etc.

| Approach | Credentials on VM | Recommended |
|----------|-------------------|-------------|
| CI → rsync/SCP over SSH | SSH public key only | **Yes (baseline)** |
| Deploy key + `git fetch`/`archive` on VM | Deploy key | Acceptable interim |
| PAT / `GH_TOKEN` on VM | Long-lived token | **No** |

## Usage

```yaml
- uses: actions/checkout@v4

- uses: scholtzap/oci-deploy-baseline@v1
  with:
    host: ${{ secrets.ORACLE_VM_IP }}
    ssh_key: ${{ secrets.ORACLE_SSH_KEY }}
    ssh_passphrase: ${{ secrets.ORACLE_SSH_PASSPHRASE }}  # optional
    remote_path: /opt/example/
    local_path: ./
    delete: "true"   # metabase-style; use "false" to mirror git-archive (avocado)
    excludes: |
      .git/
      .env
      .env.*
      backups/
      .venv/

# Then SSH for rebuild / systemd / health checks only
```

## Consumers

- `scholtzap/metabase-3bo` — `/opt/metabase-3bo` (`delete: true`)
- `scholtzap/avocado` — selective paths under `/opt/dify` (`delete: false`)
- `scholtzap/openclaw-personal` — already rsync; can adopt action for parity

## Versioning

Tag releases as `v1`, `v1.0.1`, … Pin workflows to `@v1` or a commit SHA.
