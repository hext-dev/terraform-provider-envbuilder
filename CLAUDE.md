# terraform-provider-envbuilder

Fork of [coder/terraform-provider-envbuilder](https://github.com/coder/terraform-provider-envbuilder) with hext-dev/envbuilder integration.

## Purpose

This provider helps determine if a pre-built devcontainer image exists in a cache registry, enabling instant workspace starts when the cache is hit.

**Why fork?** The upstream provider uses `coder/envbuilder` for hash computation. Our fork uses `hext-dev/envbuilder` which has fixes for ARG substitution in Dockerfiles, affecting cache hash generation.

## Repository Links

| Resource | URL |
|----------|-----|
| This repo | https://github.com/hext-dev/terraform-provider-envbuilder |
| Envbuilder fork | https://github.com/hext-dev/envbuilder |
| Upstream provider | https://github.com/coder/terraform-provider-envbuilder |
| Terraform Registry | https://registry.terraform.io/providers/hext-dev/envbuilder |

## Development Setup

### Local Override (Current)

The Coder provisioner uses a local binary via `dev_overrides`:

```
/home/coder/.terraformrc:
  provider_installation {
    dev_overrides {
      "coder/envbuilder" = "/home/coder/.terraform.d/plugins"
    }
    direct {}
  }

/home/coder/.terraform.d/plugins/terraform-provider-envbuilder_v1.0.3
```

### Updating the Local Binary

```bash
# Download latest release
cd /tmp
gh release download v1.0.X -R hext-dev/terraform-provider-envbuilder -p '*linux_amd64*'

# Extract and install
python3 -c "import zipfile; zipfile.ZipFile('terraform-provider-envbuilder_1.0.X_linux_amd64.zip').extractall('.')"
sudo cp terraform-provider-envbuilder_v1.0.X /home/coder/.terraform.d/plugins/
sudo chmod +x /home/coder/.terraform.d/plugins/terraform-provider-envbuilder_v1.0.X
sudo chown coder:coder /home/coder/.terraform.d/plugins/terraform-provider-envbuilder_v1.0.X
```

## Release Workflow

### When to Release

Release a new version when:
1. `hext-dev/envbuilder` has a new release you want to use
2. Upstream `coder/terraform-provider-envbuilder` has changes to merge
3. Bug fixes or improvements to the provider itself

### Release Steps

```bash
cd ~/terraform-provider-envbuilder

# 1. Update go.mod if envbuilder changed
# Edit: replace github.com/coder/envbuilder => github.com/hext-dev/envbuilder vX.X.X
go mod tidy

# 2. Commit changes
git add -A && git commit -m "chore: bump envbuilder to vX.X.X"
git push origin main

# 3. Wait for Tests workflow to pass
gh run watch

# 4. Tag and push release
git tag v1.0.X
git push origin v1.0.X

# 5. Release workflow runs automatically (goreleaser + GPG signing)
gh run watch

# 6. Update local binary (see above)
```

### GPG Signing

Releases are signed with GPG key `C1469A531E975F1C` (terraform@hext.dev).

- **GitHub Secrets**: `GPG_PRIVATE_KEY`, `GPG_PASSPHRASE` (empty)
- **Terraform Registry**: Public key added for version verification
- **Backup**: Key backed up in password manager

To export public key (if needed for registry):
```bash
gpg --armor --export C1469A531E975F1C
```

## Terraform Registry Publishing

Once the GPG key is registered at https://registry.terraform.io/settings/gpg-keys, releases should sync automatically.

To use from registry instead of local override:
1. Remove dev_overrides from `/home/coder/.terraformrc`
2. Update template source from `coder/envbuilder` to `hext-dev/envbuilder`
3. Run `terraform init -upgrade`

## Template Integration

The `hext-devcontainer` template uses this provider:

```hcl
# In main.tf
terraform {
  required_providers {
    envbuilder = {
      source = "coder/envbuilder"  # Redirected via dev_overrides
      # OR after registry publish:
      # source = "hext-dev/envbuilder"
    }
  }
}

# Cache probe resource
resource "envbuilder_cached_image" "cached" {
  count         = var.cache_repo != "" ? 1 : 0
  builder_image = local.devcontainer_builder_image
  git_url       = local.git_url
  cache_repo    = var.cache_repo
  extra_env     = local.envbuilder_probe_env
}
```

## Debugging

Check if cache probe found a cached image:
```bash
coder state pull <workspace> | jq '.resources[] | select(.type == "envbuilder_cached_image")'
```

Expected output when cache hits:
```json
{
  "exists": true,
  "id": "<image-digest>"
}
```

## Syncing with Upstream

```bash
# Add upstream remote (one-time)
git remote add upstream https://github.com/coder/terraform-provider-envbuilder.git

# Fetch and merge
git fetch upstream
git merge upstream/main

# Resolve conflicts, keeping our go.mod replace directive
# Test and release as normal
```
