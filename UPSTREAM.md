# Upstream

This repository is a private fork of the official Braze Cordova SDK.

| Field | Value |
|---|---|
| Upstream repo | https://github.com/braze-inc/braze-cordova-sdk |
| Upstream version at fork time | **17.0.0** |
| Fork created | 2026-09-05 |
| Upstream branch tracked | `17.0.0` tag |

## Identifying company-specific commits

All company changes are prefixed with `COMPANY:` in the commit message:

```bash
git log --oneline | grep "COMPANY:"
```

## Upstream diff

To see only the changes we made on top of upstream:

```bash
git diff 17.0.0..HEAD
```

## Upgrade procedure

See [docs/upgrading-braze.md](docs/upgrading-braze.md).
