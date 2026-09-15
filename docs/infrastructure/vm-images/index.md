# VM Images

VM/disk images are too large for git. For each image, document rather than
commit the binary:

| Image | Purpose | Build source | Download | Checksum |
|---|---|---|---|---|
| e.g. `ccdc-vuln-ubuntu-22.04` | Vulnerable box for CCDC practice | `infra/packer/ccdc-vuln-ubuntu/` | (bucket/NAS link) | `sha256:...` |

Prefer building images from a Packer/Terraform definition checked into this
repo so the image itself is reproducible and disposable — store the
*build recipe* in git, not the multi-GB artifact.
