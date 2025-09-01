Multi-variant Dev Containers
============================

This directory provides multiple development container definitions so you can pick
the base OS image that matches your needs (Rocky 8/9 or Ubuntu Focal/Noble), all
sharing a common baseline config.

Structure:

```
.devcontainer/
  base/devcontainer.json        # Shared settings (mounts, features, extensions, etc.)
  rocky-8/devcontainer.json     # Uses image m0rf30/yap-rocky-8:1.44
  rocky-9/devcontainer.json     # Uses image m0rf30/yap-rocky-9:1.44
  ubuntu-focal/devcontainer.json# Uses image m0rf30/yap-ubuntu-focal:1.44
  ubuntu-noble/devcontainer.json# Uses image m0rf30/yap-ubuntu-noble:1.44
```

Usage:
1. In VS Code run: “Dev Containers: Reopen in Container”.
2. When prompted, select the variant folder you want.

To add or customize settings common to all images, edit `base/devcontainer.json`.
To add image-specific tweaks, edit only that variant's file.
