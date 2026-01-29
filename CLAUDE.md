# terraform-provider-envbuilder

Fork of coder/terraform-provider-envbuilder using hext-dev/envbuilder for cache hash computation.

## Links

- **Registry**: https://registry.terraform.io/providers/hext-dev/envbuilder
- **Envbuilder fork**: https://github.com/hext-dev/envbuilder

## Current Setup

Local dev_overrides redirect `coder/envbuilder` → `/home/coder/.terraform.d/plugins/terraform-provider-envbuilder_v1.0.3`

## Release

```bash
# 1. Update go.mod replace directive if envbuilder changed, then:
go mod tidy && git commit -am "bump envbuilder" && git push

# 2. Tag (GitHub Actions handles goreleaser + GPG signing)
git tag v1.0.X && git push origin v1.0.X

# 3. Update local binary
cd /tmp
gh release download v1.0.X -R hext-dev/terraform-provider-envbuilder -p '*linux_amd64*'
unzip terraform-provider-envbuilder_*.zip
sudo cp terraform-provider-envbuilder_v1.0.X /home/coder/.terraform.d/plugins/
sudo chmod +x /home/coder/.terraform.d/plugins/terraform-provider-envbuilder_v1.0.X
```

GPG key: `C1469A531E975F1C` (terraform@hext.dev)

## Debug Cache

```bash
coder state pull <workspace> | jq '.resources[] | select(.type == "envbuilder_cached_image") | {exists, image}'
```
