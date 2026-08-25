# AMP Template: RimWorld Multiplayer (Zetrith)

An [AMP](https://cubecoders.com/AMP) Generic Module template for the **standalone dedicated server**
shipped with [Zetrith's Multiplayer mod](https://github.com/rwmt/Multiplayer) for RimWorld.

> **Status: untested pre-release.** The template is complete and syntax-valid, but has not yet been
> verified end-to-end on a live AMP instance. See [Known risks](#known-risks) before using it.

## What this gives you

A dedicated RimWorld Multiplayer server managed by AMP — start/stop, console, config editing,
automatic restarts, and automatic installation of both the server and its .NET runtime.

## Installing

In AMP: **Configuration → Instance Deployment → Add a Configuration Repository**, then enter:

```
luna61ouo/amp-templates:main
```

Click *Fetch*, refresh your browser, and the application will appear (prefixed `Luna`) when
creating a new instance.

## What the installer does

| Stage | Action |
|---|---|
| 1 | Create `Dotnet/` directory |
| 2–4 | Download the **.NET 8 runtime** from the Microsoft CDN into the instance's own `Dotnet/` folder (Linux x64 / Linux arm64 / Windows) |
| 5 | Fetch `Server-beta.zip` from the `rwmt/Multiplayer` GitHub releases |
| 6–7 | Flatten `Server/Linux/` (or `Server/Windows/`) into the instance root |

The instance carries its own .NET runtime, so the host does not need .NET installed.

Steps 6–7 exist because the upstream zip contains **both** platform folders in a single archive,
rather than one archive per platform.

## Important behaviour of this server

**1. The server needs a real TTY.**
It polls `Console.KeyAvailable`; if stdin is redirected or piped it throws
`InvalidOperationException` and exits immediately on startup. The template sets
`App.UseLinuxIOREDIR=False`. If your instance dies the moment it starts, wrap the executable
in a pty:

```
App.ExecutableLinux=/usr/bin/script
App.LinuxCommandLineArgs=-qec "./Server" /dev/null
```

**2. The server stops itself after first-time setup — this is by design, not a crash.**
On a fresh instance the server waits for the first client to connect and create the world.
Once the world is uploaded it writes `save.zip`, disconnects everyone with reason
`BootstrapCompleted`, and shuts down. It must be **restarted** to come back up in normal mode.

→ **Enable "restart automatically when the application stops unexpectedly" on the instance**,
otherwise the server will appear to die right after you finish creating the world.

**3. The server does not simulate the game.**
It stores world/map data and relays commands; all simulation happens client-side
(deterministic lockstep). With no players connected, the world does not advance.

**4. Only the `continuous` release ships a server.**
Tagged stable releases do **not** include `Server-beta.zip`. The default `Release Version`
setting is therefore `continuous`, a rolling alpha build. All players must run a client from
the *same* build.

## Configurable settings

Server name, max players, password, multifaction, async time, autosave interval/unit,
pause-on-join, mod config syncing, desync traces, plus the release/runtime versions used
during installation.

## Known risks

| # | Risk | Fallback |
|---|---|---|
| 1 | `UseLinuxIOREDIR=False` may not actually provide a pty | Wrap in `script` (see above) |
| 2 | `settings.toml` stores `directAddress = "ip:port"` as a **combined string**, which AMP's separate port/IP fields cannot map onto. The port is currently fixed at 30502 | Add a pre-start stage that rewrites `directAddress` with the AMP-assigned port |
| 3 | AMP's support for `ConfigType: toml` is unconfirmed | Drop `metaconfig.json`; the bootstrap flow generates `settings.toml` on its own anyway |

## Credits

The installer structure is adapted from the official `rimworld-together` template in
[CubeCoders/AMPTemplates](https://github.com/CubeCoders/AMPTemplates).

Server software: [rwmt/Multiplayer](https://github.com/rwmt/Multiplayer) — not affiliated with this template.
