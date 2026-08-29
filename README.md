# AMP Template: RimWorld Multiplayer (Zetrith)

An [AMP](https://cubecoders.com/AMP) Generic Module template for the **standalone dedicated server**
shipped with [Zetrith's Multiplayer mod](https://github.com/rwmt/Multiplayer) for RimWorld.

> **Credit.** The mod and the dedicated server are the work of **Zetrith** and the
> [rwmt/Multiplayer](https://github.com/rwmt/Multiplayer) contributors. Nothing in this repository is
> their code — this is only an AMP template that downloads and runs their release. All credit for the
> server itself belongs to them; please report server bugs to their tracker, not here.

> **Status: verified on a live AMP instance (2026-08-29).** Install, start, ready-state detection,
> player count, graceful stop and re-running the installer have all been exercised on Linux x86_64.
> Windows and arm64 are untested and not declared. See [Known risks](#known-risks).

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

Release version and .NET runtime version (used during installation).

**Server settings (multifaction, async time, max players, password, autosave…) are NOT exposed
in the AMP panel** — AMP cannot parse TOML. Edit `settings.toml` directly through AMP's File
Manager instead. The file is generated automatically the first time a client completes setup.

## Known risks

| # | Risk | Fallback |
|---|---|---|
| 1 | `UseLinuxIOREDIR=False` may not actually provide a pty | Wrap in `script` (see above) |
| 2 | `settings.toml` stores `directAddress = "ip:port"` as a **combined string**, which AMP's separate port/IP fields cannot map onto. The port is currently fixed at 30502 | Add a pre-start stage that rewrites `directAddress` with the AMP-assigned port |
| 3 | ~~AMP's support for `ConfigType: toml`~~ **CONFIRMED UNSUPPORTED** — 0 of 240 official templates use `toml` (ini: 58, json: 36, xml: 10), and AMP returns *"No meta configuration manifest data is available"*. Config takeover has been removed | Edit `settings.toml` directly via AMP's File Manager. The bootstrap flow generates it on first run |

## Credits

The installer structure is adapted from the official `rimworld-together` template in
[CubeCoders/AMPTemplates](https://github.com/CubeCoders/AMPTemplates).

Server software: [rwmt/Multiplayer](https://github.com/rwmt/Multiplayer) — not affiliated with this template.
