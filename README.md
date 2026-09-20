# Ultimate CS2 RCON and Server Management Tool

Version: 1.0.4

Developer: ShAgGy

ShAgGy's Ultimate CS2 RCON and Server Management Tool is a cross-platform desktop application for administering Counter-Strike 2 dedicated servers. It provides RCON commands, local and remote lifecycle management, SteamCMD maintenance, add-on installation, scheduling, backups, player monitoring, and authenticated chat integration from one shared Avalonia application.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## Table of Contents

- [Choose Your Platform](#choose-your-platform)
- [Release Packages](#release-packages)
- [Features](#features)
- [Quick Start](#quick-start)
- [Lifecycle Modes](#lifecycle-modes)
- [Server Installation And Updates](#server-installation-and-updates)
- [Main Areas](#main-areas)
- [PlayerPunishments](#playerpunishments)
- [Settings](#settings)
- [Frameworks And Plugins](#frameworks-and-plugins)
	- [Install Frameworks And Plugins](#install-frameworks-and-plugins)
- [Backups And Restore](#backups-and-restore)
- [Data Storage And Security](#data-storage-and-security)
- [Updating Or Migrating Data](#updating-or-migrating-data)
- [Troubleshooting](#troubleshooting)
- [About](#about)
- [Application Screenshots](#application-screenshots)

## Choose Your Platform

The root README is the project overview. The platform guides are the complete start-to-finish procedures for preparing the host, configuring a profile, installing or updating CS2, and starting the server.

| Application package | Guide | Local CS2 hosting | Remote targets |
| --- | --- | --- | --- |
| Windows x64 | [Windows guide](README-WINDOWS.md) | Local Windows | Linux and Windows |
| Linux x64 | [Linux guide](README-LINUX.md) | Local Linux | Linux and Windows |
| macOS Intel or Apple Silicon | [macOS guide](README-MACOS.md) | Not supported by Valve | Linux and Windows |

macOS 12.0 or later is required. Remote Linux and Remote Windows management use SSH/SFTP; Remote Windows does not require WinRM or a separate SFTP server. Before configuring a Remote Windows profile, prepare OpenSSH and the remote Windows account using the [Remote Windows SSH setup](README-WINDOWS.md#create-the-windows-ssh-user) instructions.

## Release Packages

- `cs2-rcon-tool-universal-v1.0.4-windows-x64.zip`
- `cs2-rcon-tool-universal-v1.0.4-linux-x64.tar.gz`
- `cs2-rcon-tool-universal-v1.0.4-macos-x64.zip`
- `cs2-rcon-tool-universal-v1.0.4-macos-arm64.zip`
- `cs2-rcon-tool-universal-v1.0.4-all-platforms.zip`

Each platform build contains a self-contained, single-file native application. A separate .NET installation is not required. GeoLite data and flag images remain external runtime assets by design.

## Features

- Query CS2 server information and connected players.
- Run categorized RCON commands with suggestions and autocomplete.
- Start, stop, restart, install, update, and validate supported servers.
- Manage local servers on Windows and Linux.
- Manage remote Linux and Windows servers over SSH.
- Choose Direct process or Service commands lifecycle ownership.
- Configure startup maps, Workshop content, CFG, player passwords, tickrate, VAC, and LAN mode.
- Install or upgrade Metamod, CounterStrikeSharp, and tracked optional plugins.
- Apply Fun Stuff modes with exact per-server rollback.
- Back up and restore server files and MySQL/MariaDB databases.
- Schedule RCON and maintenance operations using built-in or native schedulers.
- Receive authenticated game chat from the separate ChatRelay plugin over UDP.
- Maintain GeoLite country information and a 195-country flag set.

## Quick Start

1. Download and extract the package for your operating system.
2. Follow the matching [Windows](README-WINDOWS.md), [Linux](README-LINUX.md), or [macOS](README-MACOS.md) launch instructions.
3. Open **Servers > Manage servers**.
4. Add a server and enter its name, host/IP, game/RCON port, RCON password, and location.
5. Complete lifecycle and SteamCMD settings when the application will manage those operations.
6. Save the profile, select it in the top server list, and click **Refresh server list**.
7. Right-click the server and select **Install/Update server...**, or **Install/Update Server + Add-ons** when framework and tracked-plugin maintenance is also required. Wait for the operation to finish, review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**.

If you chose **Install/Update server...** without add-ons, right-click the running server and select **Install/Upgrade Metamod**, then **Install/Upgrade CSS**, then **Install/Upgrade CSS Plugins...**. Paste the requested package or plugin repository/release URLs, wait for each operation to finish, and restart the server after framework/plugin installation.

Paste each of these URLs into the **Install/Upgrade CSS Plugins...** window one at a time:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/nickj609/MenuManagerAPI
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```

RCON uses TCP. The configured RCON port must be reachable from the computer running the application; opening only the UDP game port is not sufficient.

## Lifecycle Modes

Every supported local or remote profile separates two ownership models:

- **Direct process**: the application builds the launch arguments and owns the CS2 process. Tickrate, VAC, LAN mode, startup map, CFG, credentials, and related launch settings come from the profile.
- **Service commands**: an existing systemd service or Windows service wrapper owns CS2. The application executes configured Start, Stop, and Restart commands but does not install the service. Launch arguments belong in the service configuration.

Stop the current owner before switching modes. Direct process and Service commands must never manage the same executable and port simultaneously.

Detailed setup:

- [Local and Remote Windows lifecycle](README-WINDOWS.md)
- [Local and Remote Linux lifecycle](README-LINUX.md)
- [Remote management from macOS](README-MACOS.md)

Next: open the guide for the application platform you are using and follow its matching Local or Remote configuration path.

## Server Installation And Updates

Right-click a configured server and select **Install/Update server...**.

- Local Windows can download and run Windows SteamCMD.
- Local Linux can check prerequisites and download Linux SteamCMD.
- Remote Linux and Remote Windows run configured SteamCMD operations over SSH.
- Use App ID `730`.

SteamCMD runs through the application, and its progress and output are captured in **Console Commands**, **Application Log**, and **Debug** so installation and validation can be monitored without a separate interactive terminal.

On Linux, the first local SteamCMD operation may show an administrator authorization dialog while the application installs required 32-bit runtime prerequisites. This is expected Linux prerequisite setup. See the [Linux SteamCMD prerequisites](README-LINUX.md#steamcmd-linux-prerequisites) section for the package details before cancelling the prompt.

**Validate server files** and **Install/Update Server** perform explicit SteamCMD validation. Ordinary Restart skips validation so routine restarts do not become lengthy updates. Direct-process local Start validates when SteamCMD and install-directory settings are configured; Service commands defer maintenance to explicit operations.

When a managed local CS2 process is running, the application pauses before SteamCMD and offers to stop the server and continue. Cancelling leaves the server running and aborts the update or validation. After SteamCMD restores core files, **Install/Update Server** reapplies and validates the Metamod/CounterStrikeSharp loader chain before reporting success.

**Install/Update Server + Add-ons** combines a server update with supported framework and tracked-plugin maintenance.

Next: [Main Areas](#main-areas) for server actions, integrations, scheduling, and plugin management.

## Main Areas

### Server List And Actions

The top list displays hostname, address, port, map, players, ping, location, and country. Its context menu provides lifecycle, installation, add-on, backup, restore, validation, copy, and edit operations.

Server Actions control hostname, bots, teams, passwords, friendly fire, cheats, pause, map changes, maximum rounds, timed restarts, and player punishments. These controls send runtime commands and do not edit server CFG files. Configure the Startup CFG and player connection password in the server profile.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete CS2 installations, plugins, backups, or files from the managed server.

### Joining A Server

Select a server and click **Join server**. Windows uses the registered Steam URI handler, Linux tries Steam and desktop URI handlers, and macOS uses `open`. If CS2 opens without connecting, paste the copied `connect IP:port` command into the CS2 developer console.

### Fun Stuff

Modes include AWP/Sniper Wars, Bhop, Deathmatch, Grenade Wars, Molotov/Incendiary Wars, Headshot Only, Knife Arena, Pistols Only, ScoutzKnives, Surf, and Zeus Wars. Every mode includes **Enable bots**, an optional quota from 1 to 64, and a quota mode of `normal`, `fill`, or `match`. Enabled with a blank quota preserves the existing quota; entering a value enforces it. `fill` replaces bots as humans join to maintain the configured total. Disabled removes current bots and holds the quota at zero until rollback or a mode switch restores the captured quota and mode.

Before applying a mode, the application captures that server's affected persistent cvars. Switching modes restores the active mode first, then captures and applies the replacement. **None (rollback)** restores the exact captured values without executing or editing a CFG. Managed-weapon modes clean up dropped and map-placed weapons; Headshot Only deliberately leaves buying, buy zones, loadouts, dropped weapons, and ground weapons unchanged.

ScoutzKnives provides controls for air acceleration, gravity, fall-damage scale, friction, bunnyhopping, and automatic bunnyhopping. Surf provides acceleration, air acceleration, gravity, jump and landing stamina costs, round time, freeze time, cheats, and both bunnyhop switches. These values are also available in standalone `RCT.json`. Grenade Wars, Molotov/Incendiary Wars, and Zeus Wars always enforce their required loadouts.

All modes require the current [RCON Tool Companion 1.0.1 build](https://github.com/ShAgGy2035/RconCompanionTool), targeting CounterStrikeSharp API 1.0.374. Install the complete MenuManagerAPI 1.0.3 release before Companion. Administrators with `@css/fun` can use `!fun` or `/fun` in game.

Each mode activates an isolated Companion profile, so stale cleanup from one mode cannot change another mode's loadout, buying state, ground cleanup, or Deathmatch preparation. Companion maintains player-aware inventories across spawns and bot takeovers while preserving the Terrorist bomb carrier's C4. It manages SSG 08 plus team knife for ScoutzKnives, AWP-only for AWP/Sniper Wars, and dedicated Pistols, Knife, Zeus, HE-grenade, and fire-grenade loadouts.

Deathmatch stages duration, respawn, and random-spawn rules before map and round initialization. Leaving Deathmatch reloads the current map because CS2 only fully exits its built-in Deathmatch mode on a map load; other rollbacks and switches restart the round. Companion stores one authoritative pre-mode snapshot under `configs/plugins/RCT`, and a server-side lock rejects overlapping apply or rollback attempts, including during Deathmatch map changes.

MenuManagerAPI provides the `MenuManagerAPI.Shared.dll` contract used during Companion registration and the `menu:api` capability for WASD navigation. Without the complete dependency, CounterStrikeSharp reports RCT as unregistered and its commands are unavailable.

### Scheduled Tasks

Task types include RCON commands, framework/plugin maintenance, server backups, database backups, and GeoLite/flag updates. Triggers include Day, Hour, and Startup. The built-in scheduler runs while the application is open and monitoring is active. Native external tasks invoke the headless task runner and do not require the GUI to remain open; they use Windows Task Scheduler, systemd user timers, or launchd user agents.

### Console, Logs, And Chat

Console Commands accepts free-form RCON commands and provides categorized suggestions. Each server has separate bounded history. Application Log reports normal operations and failures; Debug provides detailed diagnostics. Passwords, credentials, API keys, and tokens are redacted from console history, logs, and debug output. When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the app or **Exit program** to close anyway.

The Chat tab receives authenticated JSON messages from the separate [ChatRelay](https://github.com/ShAgGy2035/ChatRelay) CounterStrikeSharp plugin over UDP and can send admin chat through RCON. Configure the receiving adapter and port, set a unique token under **Edit server > Integrations**, then use **Copy ChatRelay target JSON**. The listener accepts authenticated JSON only, limits datagrams to 8 KiB, and drops rejected packets. Restrict firewall access to the game server or trusted LAN, or use a VPN.

## PlayerPunishments

The Slay and Slap controls send `css_slay #userid` and `css_slap #userid damage`. Install [PlayerPunishments 1.0.0 or newer](https://github.com/ShAgGy2035/PlayerPunishments) when the server's existing administration package does not provide those commands. The player table's Alive and Health values also depend on the required server-side player-information command and display **Unknown** when it is unavailable.

Configure Slap damage from `0` to `99` under **Settings > General**. PlayerPunishments is installed on the managed CS2 server as a CounterStrikeSharp plugin, not beside the desktop application.

## Settings

Open **Settings > Settings** for application-wide options. The **General** tab includes GeoLite and country-flag URLs, slap damage, automatic 15-second server-list refresh, noisy-output filtering, and GeoLite/flag updates. The **Backups** tab provides separate destinations for server-file and database backups.

Server-specific lifecycle, SteamCMD, map, RCON, player-password, database, ChatRelay, Steam authentication, and backup credentials remain under **Servers > Manage servers > Edit server**.

## Frameworks And Plugins

### Install Frameworks And Plugins

Install Metamod before CounterStrikeSharp. Interactive installation shows the latest 20 compatible releases and accepts a selected release or direct HTTP(S) package URL. Scheduled and headless maintenance use the newest compatible releases because they cannot display selection dialogs.

1. Select the configured server in the top server list.
2. Right-click the server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL when prompted, then wait for installation to finish.
3. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL when prompted, then wait for installation to finish.
4. Right-click the server again and select **Install/Upgrade CSS Plugins...** to install optional CounterStrikeSharp plugins. Paste the plugin's GitHub or supported GitLab repository/release URL, choose the matching release asset if prompted, and wait for deployment to finish.
5. Restart the server after installing or upgrading frameworks or plugins so the server loads them.

Under **Settings > Metamod Settings**, list plugins and run info, pause, unpause, retry, load, unload, or force-unload operations. Under **Settings > CStrikeSharp Settings**, reload admins and list, reload, or unload plugins. Use **Settings > Test Metamod/CSS command availability** to probe safe command forms with nonexistent plugin IDs without targeting real plugins.

Optional plugins can come from GitHub, a public GitLab.com project including nested groups, or a supported local archive or library. The installer filters assets by server OS. When a release has multiple packages for that OS, the application asks which asset to install and remembers a version-independent filename preference for later upgrades. Installed files are tracked per server in:

```text
game/csgo/.cs2-rcon-tool-plugins.json
```

Keep this ownership manifest with the server. It allows uninstall to remove owned files while preserving shared files, configuration, and framework files.

Remote framework and plugin updates replace native binaries instead of truncating loaded files in place. A running server continues using the previous binary until restart, avoiding memory-mapped library crashes during deployment.

## Backups And Restore

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking instead of stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` using the selected server's integration settings.
- **Restore server backup...** validates and restores ZIP, TAR.GZ, or TGZ content, stopping and restarting the server as needed.
- **Install/update + restore server backup...** updates core files before restoring a backup.
- **Restore database backup...** imports SQL using the configured MySQL client.

Remote operations require appropriate SSH permissions and archive tools on the managed host.

## Data Storage And Security

Writable data is stored per user, not beside the executable:

- Windows: `%APPDATA%\CS2RconTool`
- Linux: typically `~/.config/CS2RconTool`
- macOS: typically `~/Library/Application Support/CS2RconTool`

The directory contains:

- `servers.json`: profiles and protected credentials.
- `tasks.json`: scheduled tasks and protected credentials.
- `steam_settings.json`: application settings and protected tokens.
- `fun_stuff_state.json`: per-server Fun Stuff state without credentials.
- `credential.key`: the per-user AES key.
- `GeoLite2-Country.mmdb` and `flags/`: writable GeoIP assets.

Saved secrets use AES-256-GCM. Protect `credential.key` as carefully as the JSON files. Release archives intentionally exclude settings, keys, caches, PDB files, ChatRelay binaries, and Companion binaries.

## Updating Or Migrating Data

Close the application before updating. Replace application files or extract the new package to a new folder; on macOS, replace the entire app bundle. Back up the complete per-user `CS2RconTool` directory first.

For migration:

1. Open the new application once, then close it.
2. Back up its generated per-user data directory.
3. Copy `servers.json`, `tasks.json`, `steam_settings.json`, and `fun_stuff_state.json` from the old installation.
4. Copy the matching old `credential.key` when available.
5. Reopen the application.

Configuration can load when protected credentials cannot be decrypted, but affected RCON, SSH, database, ChatRelay, Steam API, and GSLT values must be re-entered and saved.

Remote profiles remain portable when valid credentials and target paths are supplied. New profiles default to the local type supported by the current OS. Existing imported profile locations are not changed automatically, and an incompatible local profile must be changed to a supported location before it can be saved.

- When moving to Windows, change `Local Linux` to `Local Windows`, then replace the Linux executable, SteamCMD, install-directory, working-directory, and launch paths with Windows paths.
- When moving to Linux, change `Local Windows` to `Local Linux`, then replace the Windows executable, SteamCMD, install-directory, working-directory, and launch paths with Linux paths.
- macOS cannot run local CS2 profiles. Convert an imported local profile to the correct Remote Linux or Remote Windows type and provide valid target paths and credentials.

## Troubleshooting

- Re-enter and save RCON or SSH credentials after migration.
- Verify the host/IP and RCON TCP port.
- Confirm firewalls permit the game UDP port and RCON TCP port.
- Check Application Log and Debug for lifecycle, SteamCMD, SSH, and RCON details.
- Do not run Direct process and Service commands against the same executable and port.
- Install or validate Metamod before CounterStrikeSharp when framework commands are unknown.
- If Steam opens CS2 without connecting, enable the developer console, press `~`, and paste the copied `connect IP:port` command.
- If Chat does not appear, verify ChatRelay is running, match the destination IP, adapter, UDP port, and token on both sides, check firewall rules, and use the Chat tab's **Test** action.
- Use **Settings > General > Update GeoLite + flags** when country data is missing. Confirm the configured URLs are direct-download sources and the per-user data directory is writable.
- Confirm built-in scheduler monitoring is active or inspect the platform's native scheduler when tasks do not run.

Platform-specific diagnostics:

- [Windows troubleshooting](README-WINDOWS.md#troubleshooting)
- [Linux troubleshooting](README-LINUX.md#troubleshooting)
- [macOS troubleshooting](README-MACOS.md#troubleshooting)

## About

Open **Help > About** to view the application version, developer information, and detected platform.

## Application Screenshots

| Server actions | Server context menu |
| --- | --- |
| ![Server actions](docs/images/server-actions.png) | ![Server context menu](docs/images/server-context-menu.png) |
| Manage servers | Edit a Remote Linux server |
| ![Manage servers](docs/images/manage-servers.png) | ![Edit a Remote Linux server](docs/images/edit-remote-linux-server.png) |
| Edit server integrations | Add scheduled task |
| ![Edit server integrations](docs/images/edit-server-integrations.png) | ![Add scheduled task](docs/images/add-scheduled-task.png) |
| Fun Stuff modes | Server overview |
| ![Fun Stuff mode selection](docs/images/fun-stuff-modes.png) | ![Server overview](docs/images/server-overview.png) |
| Scheduled tasks | Console commands |
| ![Scheduled tasks](docs/images/scheduled-tasks.png) | ![Console command suggestions](docs/images/console-commands.png) |
| Chat | Application log |
| ![Chat](docs/images/chat-tab.png) | ![Application log](docs/images/application-log-tab.png) |
| Debug output | General settings |
| ![Debug output](docs/images/debug-tab.png) | ![General settings](docs/images/settings-general.png) |
| Settings menu | Backup settings |
| ![Settings menu](docs/images/settings-menu.png) | ![Backup settings](docs/images/settings-backups.png) |
| Metamod controls | CounterStrikeSharp controls |
| ![Metamod controls](docs/images/settings-metamod-menu.png) | ![CounterStrikeSharp controls](docs/images/settings-counterstrikesharp-menu.png) |