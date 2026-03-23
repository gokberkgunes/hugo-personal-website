---
title: "Pacman and Makepkg "
date: 2026-03-23T01:15:00-05:00
authors: ["Gokberk Gunes", ]
tags: ["Linux", "Artix", "Pacman"]
draft: true
---


## Package Inquiry and Ownership
Use these commands to identify file origins and query the local database.

| Action | Command | Context |
| :--- | :--- | :--- |
| **Global Search** | `pacman -F /path/to/file` | Identify which package contains a file (requires `pacman -Fy`). |
| **Local Ownership** | `pacman -Qo /path/to/file` | Find which installed package owns a specific binary. |
| **List Contents** | `pacman -Ql <pkg>` | List every file installed by a specific package. |
| **Explicitly Installed** | `pacman -Qe` | List packages you manually installed (excluding dependencies). |
| **Foreign (AUR)** | `pacman -Qm` | List packages not found in official sync databases. |

## Maintenance and Cleanup
Keep the system lean by pruning orphans and managing the local cache.

| Action | Command | Description |
| :--- | :--- | :--- |
| **Clean Remove** | `pacman -Rns <pkg>` | Removes package and its unneeded dependencies. |
| **Force Remove** | `pacman -Rdd <pkg>` | Skips all dependency version checks (use with caution). |
| **Show Orphans** | `pacman -Qdt` | Lists packages installed as dependencies but no longer required. |
| **Clear Old Cache** | `pacman -Sc` | Removes all cached packages except those currently installed. |
| **Wipe Cache** | `pacman -Scc` | Deletes the entire cache directory. |

## System Optimization

### Tmpfs Cache (SSD/HDD Longevity)
Redirect the Pacman cache to RAM to minimize disk I/O. Beware that some distros do not create /tmp as `tmfs`, check `df`.
Edit `/etc/pacman.conf`:
```text
CacheDir = /tmp
