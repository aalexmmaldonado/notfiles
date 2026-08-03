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

## Tasks

[`just`][just] runs the routine tasks in this repo. Run it with no arguments to list every recipe:

```
just
```

Most recipes act on a single host.
The `host` variable resolves on its own: macOS reads `scutil --get LocalHostName`, Linux uses `hostname`.
That value has to match a key under `nixosConfigurations` or `darwinConfigurations` in `flake.nix`.
To act on a different machine, override it:

```
just host=whitegrizzly rebuild
```

`rebuild`, `clean`, and `diff` choose `darwin-rebuild` or `nixos-rebuild` from the current OS, so one command covers every machine.

Three recipes (`fmt`, `lint`, `diff`) use tools that live in the flake's dev shell rather than a host profile.
With direnv the shell loads on entry to the repo (see [Dev shell](#dev-shell)); without it, prefix the command, for example `nix develop -c just lint`.

| Recipe | What it does |
| :----- | :----------- |
| `rebuild` | Build the current host's config and switch to it now |
| `diff` | Build the current host's config and show what would change, without switching |
| `update` | Update `flake.lock` to the latest inputs |
| `clean` | Delete generations older than 7 days, optimize the store, stage the result for next boot |
| `fmt` | Format every `.nix` file with `nixfmt` |
| `lint` | Run `statix` and `deadnix` over the tree |
| `gen-keys` | Generate Ed25519 SSH keys for the named services |
| `fetch-iso` | Download the latest NixOS installer ISO into `iso/` |
| `make-usb` | Write a bootable installer USB |
| `register-host` | Scaffold `hosts/<name>/` and add a `flake.nix` entry |
| `capture-hardware` | Write the current machine's `hardware.nix` |
| `hosts` | List the hosts registered in `flake.nix` |
| `docker-creds` | Seed `~/.docker/config.json` and set up a credential store |

## Dev shell

Editing the repo (as opposed to running it) calls for a formatter, two linters, and a pair of inspectors: `nixfmt`, `statix`, `deadnix`, `nvd`, and `nix-tree`.
They live in `devShells.default` in `flake.nix`, deliberately outside every host's system profile, so they show up only while you work on this repo.

A repo-root `.envrc` containing `use flake` loads the shell through [direnv] on entry and drops it on exit.
Run `direnv allow` once after cloning.
Without direnv, enter it by hand:

```
nix develop
```

Either path puts `just fmt`, `just lint`, and `just diff` in reach.

## Common workflows

### The edit loop

Change a module, preview the effect, then switch:

```
$EDITOR home/modules/helix.nix
just diff
just rebuild
```

`just diff` builds the target system to `/tmp/notfiles-next` and runs `nvd diff` against the running one, so the added, removed, and upgraded packages are visible before anything changes.
The switch happens only at `just rebuild`.

Before committing, format and lint:

```
just fmt
just lint
```

`lint` exits nonzero when `statix` or `deadnix` find something, which makes it a usable pre-commit or CI gate.
The first run on an established config tends to surface unused bindings and a few anti-patterns; clear them once and later runs go quiet.

### Staying current

```
just update
just diff
just rebuild
```

`just update` refreshes every flake input.
Read the changes with `just diff` before switching.
Reclaim disk now and then:

```
just clean
```

`clean` drops generations older than a week, deduplicates the store, and stages the rebuilt system for the next boot (a plain `switch` on macOS).

### Adding a machine

Register the host from any checkout, then capture hardware on the machine itself:

```
just register-host blackbear      # scaffolds hosts/blackbear/, edits flake.nix
# then, on blackbear:
just capture-hardware blackbear    # writes hosts/blackbear/hardware.nix
just host=blackbear rebuild
```

`just hosts` shows what is already registered.

### Building install media

For a fresh NixOS box, fetch an installer image and write it to a USB stick:

```
just fetch-iso
just make-usb --iso iso/latest-nixos-graphical-x86_64-linux.iso
```

`fetch-iso` accepts channel and edition overrides, for example `just fetch-iso --channel nixos-unstable --edition minimal`.
`make-usb` self-elevates with sudo when it needs to.

### Keys and credentials

Generate signing and auth keys, then seed Docker's credential store:

```
just gen-keys
just docker-creds
```

`gen-keys` writes one Ed25519 key per service and defaults to the set this repo expects (GitHub, Gradescope, the HPC clusters, and the two Git signing keys).
`docker-creds` defaults to a `pass`-backed store and sets up the GPG key behind it; pass `secretservice` to use the desktop keyring instead.

[just]: https://github.com/casey/just
[direnv]: https://direnv.net/
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

