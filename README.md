<h1 align="center">notfiles</h1>
<p align="center"><b>N</b>ixOS <s>d</s>otfiles</p>

My declarative NixOS configuration.

## Structure

```
notfiles/
├── flake.nix               # Flake inputs and host definitions
├── hosts/                  # Host-specific config (hardware, hostname, etc.)
│   ...
├── modules/                # NixOS system-level modules (shared across hosts)
│   ...
└── home/                   # User-level entry point and user-specific config
    ├── alex/
    │   ...
    └── modules/            # Reusable Home Manager modules
        ...
```

## Flake inputs

| Input | Purpose |
| :---- | :------ |
| [`nixpkgs`][nixpkgs] | NixOS 26.05 |
| [`home-manager`][home-manager] | User environment management |
| [`plasma-manager`][plasma-manager] | Declarative KDE Plasma configuration |
| [`nix4vscode`][nix4vscode] | VSCode/VSCodium extension management |
| [`globalprotect-openconnect`][globalprotect-openconnect] | VPN client |

[nixpkgs]: https://github.com/NixOS/nixpkgs
[home-manager]: https://github.com/nix-community/home-manager
[plasma-manager]: https://github.com/nix-community/plasma-manager
[nix4vscode]: https://github.com/nix-community/nix4vscode
[globalprotect-openconnect]: https://github.com/yuezk/GlobalProtect-openconnect


## Where this project lives

The real home for this project is a self-hosted Gitea instance: **[git.scient.ing/alexm/notfiles](https://git.scient.ing/alexm/notfiles)**, where the code, issues, and history live on infrastructure I control.
GitHub only holds a pointer, so open issues, send pull requests, and clone from the link above.
I moved because I would rather own where my code lives than rent it under terms a platform can rewrite by announcement, and self-hosting puts backups, uptime, and data location back in my hands.
Old GitHub links still resolve here, so nothing breaks.

