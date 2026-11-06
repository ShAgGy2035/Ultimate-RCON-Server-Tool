# Ultimate CS2 RCON and Server Management Tool

Development version: 1.0.6

Developer: ShAgGy

Manage Counter-Strike 2 dedicated servers from one Windows, Linux, or macOS desktop application. Use RCON commands, monitor players, manage server processes, install updates and plugins, schedule tasks, and make backups without switching between separate administration tools.

**New here?** Start with [Getting Started](#getting-started), then follow your [platform guide](#choose-your-platform). Already using Universal? Read the [1.0.6 release notes](Release%20Notes-1.0.6.md) and [upgrade guidance](#updating-or-migrating-data). To see the interface first, browse the [screenshots](#application-screenshots).

## Table of Contents

- [Features](#features)
- [Getting Started](#getting-started)
  - [Choose Your Platform](#choose-your-platform)
  - [Requirements](#requirements)
  - [Download And Launch](#release-packages)
  - [Add Your First Server](#quick-start)
- [Configure Your Server](#configure-your-server)
  - [Lifecycle Modes](#lifecycle-modes)
  - [Remote Connection Security](#remote-connection-security)
  - [Hosting Panels](#remote-linux-hosting-panel)
  - [SSH Tunnels And Host Pins](#remote-rcon-transport-and-host-pins)
- [Server Installation And Updates](#server-installation-and-updates)
- [Frameworks And Plugins](#frameworks-and-plugins)
  - [Which Plugins Do You Need?](#what-belongs-where)
  - [Installation And Maintenance](#install-frameworks-and-plugins)
  - [Slay And Slap](#playerpunishments)
- [Everyday Server Management](#main-areas)
- [Gameplay Modes](#gameplay-modes)
  - [Fun Stuff](#fun-stuff)
  - [Weapon Rotation](#weapon-rotation)
  - [Bot Control](#bot-control)
- [Scheduled Tasks](#scheduled-tasks)
- [Chat](#chat)
- [Backups And Restore](#backups-and-restore)
- [Settings](#settings)
  - [Debug Tracking](#debug-tracking)
- [Data Storage And Security](#data-storage-and-security)
- [Updating Or Migrating Data](#updating-or-migrating-data)
- [Troubleshooting](#troubleshooting)
- [Building Release Packages](#building-release-packages)
- [Issues](#issues)
- [About](#about)
- [Application Screenshots](#application-screenshots)

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

## Getting Started

Universal is the desktop application. You can use its basic server administration without installing any of the optional companion plugins. Add those later for gameplay modes, extra player controls, or incoming game chat.

### Choose Your Platform

The root README is the project overview. The platform guides are the complete start-to-finish procedures for preparing the host, configuring a profile, installing or updating CS2, and starting the server.

| Application package | Guide | Local CS2 hosting | Remote targets |
| --- | --- | --- | --- |
| Windows x64 | [Windows guide](README-WINDOWS.md) | Local Windows | Linux and Windows |
| Linux x64 | [Linux guide](README-LINUX.md) | Local Linux | Linux and Windows |
| macOS Intel or Apple Silicon | [macOS guide](README-MACOS.md) | Not supported by Valve | Linux and Windows |

macOS 12.0 or later is required; native local CS2 hosting is not supported on macOS. Remote file operations use SFTP. Remote commands use a verified SSH tunnel or, for compatible Linux hosts, the hosting panel's encrypted console. Remote Windows does not require WinRM or a separate SFTP server.

### Requirements

- A supported desktop platform from the table above. Application packages include .NET; a separate installation is not needed.
- A CS2 dedicated server, or permission to install one on a supported Windows/Linux host.
- Connection details and permission to administer the server. Remote setup needs either SSH access or a compatible hosting panel; file management also needs SFTP access.
- Metamod and CounterStrikeSharp only if you want server-plugin features. Companion, PlayerPunishments, and ChatRelay are separate, optional installations described below.

<a id="release-packages"></a>

### Download And Launch

- `cs2-rcon-tool-universal-v1.0.6-windows-x64.zip`
- `cs2-rcon-tool-universal-v1.0.6-linux-x64.tar.gz`
- `cs2-rcon-tool-universal-v1.0.6-macos-x64.tar.gz`
- `cs2-rcon-tool-universal-v1.0.6-macos-arm64.tar.gz`
- `cs2-rcon-tool-universal-v1.0.6-all-platforms.zip`

These are the package names for the upcoming 1.0.6 release, not a claim that it has been published. Download published builds from [GitHub Releases](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/releases). macOS packages use `.tar.gz` only.

Each platform build contains a self-contained application. GeoLite data and flag images remain separate runtime assets.

Extract the package, then follow the launch instructions in your platform guide. If you already have profiles, [back up your data before replacing an older build](#updating-or-migrating-data).

<a id="quick-start"></a>

### Add Your First Server

1. Click **Add server** above the server list. You can also open **Servers > Manage servers** and choose **Add**.
2. **Choose a server type.** Select where CS2 runs and how it starts. With **Service commands**, configure the service first using the linked platform guide; Universal does not create it.
3. **Enter connection details.** Add a profile name, host/IP, port, and the credentials required for that location. For remote servers, configure a [verified SSH tunnel](#remote-rcon-transport-and-host-pins) or [hosting panel](#remote-linux-hosting-panel). Steam API keys and GSLTs are separate from the RCON password and are optional.
4. **Review.** Choose **Finish setup only** for an existing server, **Install CS2 server**, or install CS2 with Metamod and CounterStrikeSharp. Supply the install folder when requested. Optional startup waits for successful installation and plugin selection.
5. After setup, select the saved server and click **Refresh server list**. Review **Console Commands**, **Application Log**, or **Debug** if an operation fails. Do not reinstall a working server merely to connect to it.
6. To start a configured, stopped server, right-click it and choose **Start server**. Normal Start validates first when SteamCMD is configured.

Change advanced settings later through **Manage servers > Edit server**. The wizard's [three screenshots](#1-add-a-server) show this same sequence. Hosting-panel profiles have provider-specific installation choices rather than CS2 core installation.

## Configure Your Server

Before installing or starting CS2, establish who controls its process and, for a remote host, how Universal connects. Keep server-specific settings in the profile; application-wide preferences are under [Settings](#settings).

### Lifecycle Modes

Every supported local or remote profile separates two ownership models:

- **Direct process**: the application builds the launch arguments and owns the CS2 process. Tickrate, VAC, LAN mode, startup map, CFG, credentials, and related launch settings come from the profile. Direct startup is monitored for 30 seconds before it is reported healthy.
- **Service commands**: an existing systemd service or Windows service wrapper owns CS2. The application executes configured Start, Stop, and Restart commands but does not install the service. Launch arguments belong in the service configuration.

Stop the current owner before switching modes. Direct process and Service commands must never manage the same executable and port simultaneously.

Use the Local or Remote procedure in your [Windows](README-WINDOWS.md), [Linux](README-LINUX.md), or [macOS](README-MACOS.md) guide for executable paths, service permissions, and launch settings.

### Remote Connection Security

Choose your connection method before testing commands or starting scheduled tasks. The sections below cover hosting panels and verified SSH tunnels.

Remote commands are **blocked by default** until you configure a verified SSH tunnel or a compatible hosting-panel console. The separate unencrypted-RCON exception is off by default and exposes the password and commands if enabled. There is no automatic insecure fallback, and Universal does not change firewall or network settings.

SSH/SFTP also needs a verified server fingerprint: obtain it from the server owner or trusted console, not from an unverified connection. Save it against the exact host and port. Separate SFTP overrides need their own saved fingerprint; otherwise SFTP uses the SSH fingerprint. Unknown or changed fingerprints block the connection.

Existing profiles need these settings too. Scheduled tasks cannot approve a fingerprint interactively. Literal local loopback connections remain supported. With an SSH tunnel, RCON connects through the server's own loopback address; a hosting-panel console does not require exposing the game RCON TCP port.

### Remote Linux Hosting Panel

Users whose provider exposes a Pterodactyl-compatible panel but no SSH can manage a hosted Linux server with **Hosting panel API**. Select **Remote Linux**, choose that startup mode, and configure:

- Panel URL, using the panel root such as `https://panel.example.com`.
- The panel server identifier or UUID.
- A client API key with power-control permission.
- SFTP address, username, password, and port for plugin deployment, browsing, and file operations.

Start, Stop, and Restart use the panel API and wait for the panel to report `offline` or `running`. Console commands and scheduled/headless RCON use its encrypted WSS console, so the API key also needs console access. The game RCON password is optional and is never sent to the panel; SSH credentials are not required for console or lifecycle access.

Keep SFTP credentials and independently verified host pins for file operations. The provider owns CS2 core installation and updates: SteamCMD validation and remote shell commands are unavailable in this mode.

**Panel limitations:** console output is bounded and marker-delimited, but the panel does not correlate replies to individual commands. Unrelated logs can interleave and delayed output can be missed; the player table reuses the last complete roster when needed. HTTPS/WSS uses certificate validation and rejects redirects. Debug Mode excludes API keys, JWTs, and configured endpoints.

### Remote RCON Transport And Host Pins

Remote RCON is blocked unless one of these is configured, listed from most to least secure:

1. **Verified SSH tunnel (recommended).** In **Edit server > Transport security**, tick **Enable verified SSH tunnel**, save the SSH username and password, and enter the pin host, port, and fingerprint below. RCON is then sent through the tunnel to the server's own `127.0.0.1`.
2. **Hosting panel console** (Remote Linux, Hosting panel API). The panel's shared console can drop lines and echo `rct_..._begin/_end` markers, so the player table may occasionally miss rows; Universal reuses the last complete roster to compensate.
3. **Unencrypted direct RCON (explicit exception).** Tick **Allow UNENCRYPTED direct RCON to this remote server** only when the provider offers neither SSH nor a reliable console. The RCON password and every command then cross the network in clear text. It is off by default, applies per server, requires the RCON TCP port to be reachable and the RCON password saved in the profile, and is never enabled automatically. Use a unique RCON password and restrict the port to your address where the provider allows it. Use this exception only if you accept that exposure.

**Getting the host fingerprint.** The pin is the SHA256 of the server's SSH host key as `SHA256:` followed by 43 unpadded base64 characters, the same format `ssh-keygen -lf` prints. Read it on the server itself or from the provider's trusted console, never from the connection you are about to trust:

- Linux: `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`
- Windows (OpenSSH): `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`

Enter the exact host and port the profile connects to, plus the fingerprint, in the SSH pin fields. A different key type, host, or port needs its own explicit pin. A mismatch is refused without fallback.

If players appear as "Bot 1" or "Bot 2", follow [RCON troubleshooting](#rcon-commands-fail) and check the transport, SSH password, and pin. The app could not obtain the named RCON roster and fell back to its A2S query.

## Server Installation And Updates

Right-click a configured server and select **Install/Update server...**.

- Local Windows can download and run Windows SteamCMD.
- Local Linux can check prerequisites and download Linux SteamCMD.
- Remote Linux and Remote Windows run configured SteamCMD operations over SSH.
- Use App ID `730`.

SteamCMD runs through the application, and its progress and output are captured in **Console Commands**, **Application Log**, and **Debug** so installation and validation can be monitored without a separate interactive terminal.

On Linux, the first local SteamCMD operation may show an administrator authorization dialog while the application installs required 32-bit runtime prerequisites. This is expected Linux prerequisite setup. See the [Linux SteamCMD prerequisites](README-LINUX.md#steamcmd-linux-prerequisites) section for the package details before cancelling the prompt.

Normal **Start server** validates before startup when the profile has SteamCMD and install-directory settings, including Direct process and Service commands profiles. **Validate server files** validates without starting the server, while **Install/Update Server** updates, repairs, and validates the CS2 installation. Ordinary Restart skips validation. Use **Start server without validation** from the server table's right-click menu for troubleshooting when you need to start without waiting for SteamCMD validation.

When a managed local CS2 process is running, the application pauses before SteamCMD and offers to stop the server and continue. Cancelling leaves the server running and aborts the update or validation. After SteamCMD restores core files, **Install/Update Server** reapplies and validates the Metamod/CounterStrikeSharp loader chain before reporting success.

**Install/Update Server + Add-ons** combines a server update with supported framework and tracked-plugin maintenance.

If you also want gameplay modes, Slay/Slap, or incoming chat, choose the optional plugins below. Otherwise, continue to [Everyday Server Management](#main-areas).

## Frameworks And Plugins

<a id="what-belongs-where"></a>

### Which Plugins Do You Need?

Install only the server plugins needed for the features you use. Each has its own release and configuration; install them on the CS2 server, not beside the desktop application.

| Component | Purpose | Needed for |
| --- | --- | --- |
| Universal app | Desktop UI, RCON, lifecycle, updates, scheduling, and backups | Basic administration; no optional plugin required |
| [RCON Tool Companion 1.0.3](https://github.com/ShAgGy2035/RconCompanionTool) | Server control layer, status, roster output, and `css_alive` alive/dead and health information | Fun Stuff, Bot Control, and enhanced player information |
| [PlayerPunishments 1.0.1](https://github.com/ShAgGy2035/PlayerPunishments) | `css_slay` and `css_slap` | Slay/Slap when an existing admin plugin does not provide them |
| [ChatRelay 1.0.1](https://github.com/ShAgGy2035/ChatRelay) | Encrypted UDP chat delivery | Incoming game chat in the Chat tab |

Companion adds no network channel of its own. ChatRelay is opt-in: if incoming UDP is unavailable, you lose incoming game chat, not the other administration features.

**Upgrade these together:** `css_alive` moved from PlayerPunishments to Companion 1.0.3 without changing its command or output format. PlayerPunishments 1.0.1 no longer registers it. Keeping an older PlayerPunishments alongside the new Companion would duplicate the command. Companion requires the full CounterStrikeSharp 1.0.376 runtime or a compatible newer runtime, not just a replacement API DLL.

### Install Frameworks And Plugins

Install Metamod before CounterStrikeSharp. Interactive installation shows the latest 20 compatible releases and accepts a selected release or direct HTTPS package URL. Scheduled and headless maintenance use the newest compatible releases because they cannot display selection dialogs.

1. Select the configured server in the top server list.
2. Right-click it and choose **Install/Upgrade Metamod**. Choose a compatible release or direct HTTPS package URL, then wait for installation to finish.
3. Choose **Install/Upgrade CSS** and wait for CounterStrikeSharp installation to finish.
4. Choose **Install/Upgrade CSS Plugins...** for optional CounterStrikeSharp plugins. Enter their GitHub or supported GitLab repositories; the manager accepts one per line. Select the matching release asset if prompted.
5. Restart the server after framework or plugin changes so it loads the new binaries.

The setup wizard can install the frameworks and open both plugin managers for you. Close a manager to skip optional plugins. For an existing installation, use the same managers from the server context menu.

For the optional plugins listed above, enter only the repositories you need:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/PlayerPunishments
https://github.com/ShAgGy2035/ChatRelay
```

### Download Verification

Downloads reject HTTP, embedded URL credentials and HTTPS-to-HTTP redirects. Application update, plugin, add-on and SteamCMD downloads require normal certificate validation plus revocation checking; unavailable revocation status may block the operation, with no insecure retry. HTTPS CDN redirects remain supported (maximum ten). Native remote Windows add-on downloads apply the same policy; native remote Linux requires curl with HTTPS-only initial/redirect protocols and the host's certificate/status capabilities, which still require separate deployment verification. Wget/Python fallbacks are not used; missing curl fails explicitly without installing it.

For framework add-ons and optional plugins, a valid publisher SHA-256 digest is checked automatically against the downloaded package before extraction or deployment; a mismatch always blocks installation. Missing digests are labeled **Integrity unverified** and require either an independently obtained publisher SHA-256 or an explicit approval in the current manual install dialog. That approval is never saved or accepted by scheduled/headless tasks. Remote framework downloads are checked on the target before extraction; optional plugin packages are verified locally before their payload is deployed. Manual approval does not prove that the package came from its publisher.

**SteamCMD bootstrap is stricter:** a missing SteamCMD installation requires an independently obtained publisher SHA-256 before download/extraction, locally or over SSH. There is no unverified-approval option, saved exception or unattended bootstrap. Existing configured SteamCMD installations continue to update/validate normally. No automatic trusted Valve checksum-discovery source has been established: if you cannot obtain an independent publisher hash, cancel and use an existing independently verified installation; do not calculate the expected hash from the same untrusted download. Verification failure must not start SteamCMD. A path appearing after authorization cannot skip the required verified bootstrap.

### Maintain Installed Plugins

Under **Settings > Metamod Settings**, list plugins and run info, pause, unpause, retry, load, unload, or force-unload operations. Under **Settings > CStrikeSharp Settings**, reload admins and list, reload, or unload plugins. Use **Settings > Test Metamod/CSS command availability** to probe safe command forms with nonexistent plugin IDs without targeting real plugins.

Optional plugins can come from GitHub, a public GitLab.com project including nested groups, or a supported local archive or library. The installer filters assets by server OS. When a release has multiple packages for that OS, the application asks which asset to install and remembers a version-independent filename preference for later upgrades. Installed files are tracked per server in:

```text
game/csgo/.cs2-rcon-tool-plugins.json
```

Use the **Manage Metamod plugins** window only for packages containing an `addons/metamod` payload. Use the **Manage CounterStrikeSharp plugins** window for CounterStrikeSharp plugin packages. The application rejects packages submitted through the wrong manager before deployment.

Keep this ownership manifest with the server. It allows uninstall to remove owned files while preserving shared files, configuration, and framework files.

Remote framework and plugin updates replace native binaries instead of truncating loaded files in place. A running server continues using the previous binary until restart, avoiding memory-mapped library crashes during deployment.

<a id="playerpunishments"></a>

### Slay And Slap

The player controls send `css_slay #userid` and `css_slap #userid damage`. Install PlayerPunishments when your existing administration package does not provide those commands. Slap damage is configurable from `0` to `99` under **Settings > Settings > General**.

The player table's Alive and Health values use Companion's `css_alive` command, not PlayerPunishments. They display **Unknown** when that information is unavailable.

<a id="main-areas"></a>

## Everyday Server Management

### Server List And Actions

The top list displays hostname, address, port, map, players, ping, location, and country. Its context menu provides lifecycle, installation, add-on, backup, restore, validation, copy, and edit operations.

Use **Server actions** for hostname, teams, friendly fire, cheats, pause, map changes, maximum rounds, and timed restarts. Open the separate **Bot Control** tab for bot adjustments; select a player in the player table for kick, ban, Slay, or Slap. Runtime controls do not edit server CFG files. Configure the Startup CFG and player connection password in the server profile.

The map selector lists playable maps and configured Workshop maps.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete CS2 installations, plugins, backups, or files from the managed server.

### Joining A Server

Select a server and click **Join server**. Windows uses the registered Steam URI handler, Linux tries Steam and desktop URI handlers, and macOS uses `open`. If CS2 opens without connecting, paste the copied `connect IP:port` command into the CS2 developer console.

<a id="console-logs-and-chat"></a>

### Console And Logs

**Console Commands** accepts free-form RCON commands and provides categorized suggestions. Each server has its own bounded history. **Application Log** reports normal operations and failures; **Debug** provides detailed diagnostics when [enabled](#debug-tracking).

Known configured secrets are redacted from console history, logs, and debug output. Review output before sharing it, particularly when custom commands or third-party plugins can print arbitrary content.

## Gameplay Modes

Gameplay modes require the matching Companion on the managed CS2 server. They are optional and do not replace the ordinary server-management tools above.

### Fun Stuff

Select **Fun Stuff**, choose a mode, customize its settings, and start it. Use [RCON Tool Companion 1.0.3](https://github.com/ShAgGy2035/RconCompanionTool) with the full CounterStrikeSharp 1.0.376 runtime. Install it on the CS2 server, not beside Universal.

Modes include No Bots, AWP / Sniper Wars, Bhop, Deathmatch, Flying Dutchman, Grenade Wars, Molotov / Incendiary Wars, Headshot Only, Knife Arena, One in the Chamber, Pistols Only, ScoutzKnives, Super Hero, Surf, Weapon Rotation, and Zeus Wars. Super Hero offers fourteen configurable powers; each mode has its own gameplay, bot, and supported map settings.

New mode configurations have bots disabled. Enable them per mode if wanted; saved choices are preserved. Bot counting can use `normal` (fixed count), `fill` (maintain the total as humans join), or `match`. A blank enabled quota keeps the current value.

Modes temporarily change running-server settings, not CFG files. **None (rollback)** restores captured values. Some modes reload the map when starting or ending; expect connected players to see a map transition.

Companion runs the active mode on the server and, by default, restores normal settings after five minutes without human players. Configure this under **Empty-server rollback**. Closing Universal does not stop a Companion-owned mode.

#### In-Game Controls

Trusted administrators with `@css/fun` can type `!fun` or `/fun`: **W/S** moves, **E** selects, **A** goes back, and **R** exits. No separate menu plugin is needed.

All configurable modes use a quick-start wizard with Start, optional Customize gameplay, and Bots. Start applies choices only to that run, without saving to `RCT.json`; cancellation discards them. Temporary choices survive maps, but plugin reload/server restart uses configured defaults. The old editable menu is removed and `UseFunSetupWizard` is ignored. Separate Bot Control preferences still save when applied; Universal and scheduled-start behavior is unchanged.

Companion's `HiddenFunMenuModes` setting can hide gameplay modes and No Bots from `!fun`. It only changes menu visibility, not access through Universal, scheduled tasks, or direct commands. Bot Control and Rollback remain visible.

#### Team Balancing

With the matching Companion 1.0.3 build, round-based team modes correct uneven bot teams at round starts, including when the only human is spectating. Each mode has an independent handler; `mp_autoteambalance 1` enables its bot-only check and `0` disables it. The fallback never moves humans or human-controlled bots, and skips FFA, respawn gameplay, warmup, halftime/team switching, and fixed-team Bot Control. Deathmatch, One in the Chamber, and No Bots remain unchanged. Weapon Rotation uses its own option described below.

### Weapon Rotation

1. Open **Fun Stuff > Weapon Rotation**. Its tabs are Gameplay, Weapons, Rotation, Bots, and Maps.
2. Select categories and weapons; at least one selected weapon must belong to an enabled category.
3. Choose team rounds, team respawn, or free-for-all; give everyone the same weapon or use independent assignments.
4. Set the rotation interval (5-3600 seconds, default five minutes). **Duration minutes** limits the whole mode, not each weapon: `0` is unlimited; 1-1440 ends with Rollback ALL, including Bot Control.
5. Choose Popup, Chat, or both for announcements. Chat reminders default to 30 seconds; `0` disables periodic reminders.
6. Configure an optional Workshop map or collection, review, and start. Map settings are configured here or in `RCT.json`, not through `!fun`.

New settings use category rotation, shuffled weapons, infinite ammunition, no bots, and category voting. Players select the next category in a popup showing all six choices; `!wrvote` reopens it. Chat announces invitations/results but does not accept votes. Voting normally opens 30 seconds before a change and closes five seconds before it; very short intervals shorten or skip voting.

In **Rotation > Change weapons by**, select **Timed interval** (the existing default) or **Every round** with **Gameplay > Team rounds**. Every round follows your server's existing round length, including 1:55 rounds, without changing it. Category voting runs during the round, closes at round end, and selects the next round's category. The assignment changes once at the next round start, not on respawn; notices say "Changes next round". Team/FFA respawn requires timed rotation. The matching Companion 1.0.3 build is required, and dedicated scheduled tasks save this choice independently.

Weapons change while players are alive, and respawning keeps the current assignment. Team rounds preserve bomb objectives. Map voting and other menus take priority over weapon notices.

**Auto-balance teams** defaults on in Gameplay and can be disabled for team play. It enables CS2's native balancing and a one-player team-size limit. Companion also checks at round starts and moves bots from the larger team until the difference is at most one, where eligible bots are available. This bot-only fallback works while the only human spectates; it never moves humans or bots under human control. Checks skip warmup, halftime/team switching, and match-end screens. Balancing is not immediate when someone joins spectators; team respawn has no regular round boundaries, so it does not guarantee ongoing rebalancing. Free-for-all and fixed-team Bot Control setups are exempt. Install the matching Companion 1.0.3 build for this setting.

Use the editors rather than guessing JSON values. If editing by hand, follow the Companion README's **Settings with fixed values** reference. An active run keeps its starting settings; roll back and restart to apply changed defaults.

### Bot Control

The separate **Bot Control** tab adjusts count, difficulty, teams, and halftime over compatible modes. **Restore bots ONLY (keep mode)** restores bot changes; **Rollback ALL (mode + bots)** restores the pre-mode setup. Free-for-all modes reject incompatible team/halftime choices, and No Bots blocks conflicting bot edits.

## Scheduled Tasks

Use **Scheduled Tasks > Add task** to schedule RCON commands, framework/plugin maintenance, server or database backups, GeoLite/flag updates, or gameplay starts. Triggers include Day, Hour, and Startup.

1. Choose the task type and, where required, a **Saved server** near the top of the editor.
2. Select the scheduler and trigger, then set the time.
3. Enter the command or task-specific options and save the task with **Active** enabled.
4. For built-in tasks, start the scheduler and leave Universal open. For native tasks, verify the entry in your operating system's scheduler.

| Scheduler | When it runs |
| --- | --- |
| Built-in | While Universal is open and the task runner is started |
| Windows Task Scheduler | Through the headless runner; the GUI may be closed |
| Linux systemd user timer | Through the headless runner under the configured user |
| macOS launchd user agent | Through the headless runner under the configured user |

Tasks use the saved profile's current address, port, and credentials at each run rather than copying the RCON password. Changing the server's address or password does not require editing linked tasks. If the profile is deleted, the task fails explicitly until you select another saved server. GeoLite/flag updates do not need a server.

Existing legacy tasks continue to work with their original connection details until edited. Opening a legacy task selects its saved server only when the address matches exactly one profile; saving converts it to a live reference and removes its copied connection details.

### Gameplay Scheduling

**Fun Stuff rotation** tasks start one or more Fun Stuff modes in order. Each step has a mode and a duration in minutes; after the last step the rotation can stay on the last mode, roll it back, or repeat from the first step. A single step with **Stay on last mode** simply switches the server into that mode at the trigger time. Modes are started by RCON Tool Companion 1.0.3 using its saved `RCT.json` defaults; temporary `!fun` customization does not overwrite those defaults. Rotations run only in the built-in scheduler while the app is open; editing or deleting the task, or stopping the scheduler, ends a running rotation and leaves the current mode active.

The dedicated **Weapon Rotation** task instead stores its own full gameplay configuration and supports built-in or native scheduling. The similarly named entry inside a general Fun Stuff sequence uses Companion's saved defaults, not a separate per-step configuration.

For two custom weapon blocks, create two dedicated tasks for the same saved server: for example, **18:00**, Assault rifles only, then **20:00**, Sniper rifles only. Set the first task's mode duration to **0** (until replaced), so its rollback cannot compete with the next start. Set the final task's duration to **120 minutes** to roll back after its two-hour block. These are scheduled starts, not a chained sequence: if the second start fails, the first continues until stopped or replaced.

## Chat

The **Chat** tab receives game chat through the separate ChatRelay plugin and sends admin messages through RCON.

1. Install the matching [ChatRelay 1.0.1](https://github.com/ShAgGy2035/ChatRelay) on the CS2 server.
2. Select the server profile and generate a unique 32-byte secure key under **Edit server > Integrations**.
3. In **Chat**, choose the receiving adapter, listen IP, and UDP port. Use **Copy ChatRelay target JSON** to prepare the plugin destination with the same key.
4. Configure the destination in the server plugin, then enable the listener. Stop and re-enable it after switching profiles or changing keys.
5. Use **Test** for the local receiver, then send actual player chat to verify the complete server-to-app path.

Delivery uses AES-256-GCM encrypted UDP v1. The listener limits datagrams to 8 KiB, authenticates/decrypts messages, and rejects replayed or invalid packets. Legacy plaintext sender/receiver paths are removed. Saved tokens and legacy flags remain only for migration diagnostics; a legacy-only profile fails closed until a matching secure key is configured.

Review firewall/VPN requirements for your deployment. Universal does not automatically change host or network policy.

## Backups And Restore

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking instead of stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` using the selected server's integration settings.
- **Restore server backup...** validates and restores ZIP, TAR.GZ, or TGZ content, stopping and restarting the server as needed.
- **Install/update + restore server backup...** updates core files before restoring a backup.
- **Restore database backup...** imports SQL using the configured MySQL/MariaDB client. The client must support `--binary-mode`, which disables local shell and file-source commands during noninteractive restore; unsupported clients are rejected before credentials are sent. Ordinary SQL and `DELIMITER` routines remain supported. Only restore trusted SQL: database permissions and server-side effects are unchanged.

Remote operations require appropriate SSH permissions and archive tools on the managed host.

## Settings

Open **Settings > Settings** for application-wide preferences:

| Tab | Options |
| --- | --- |
| General | GeoLite and country-flag URLs, slap damage, automatic 15-second server-list refresh, noisy-output filtering, startup update checks, minimize to tray, and GeoLite/flag updates |
| Backups | Separate destinations for server-file and database backups |

Server-specific lifecycle, SteamCMD, maps, RCON, player passwords, database, ChatRelay, and Steam authentication settings remain under **Servers > Manage servers > Edit server**.

### Debug Tracking

Enable **Settings > Debug Mode > Debug On** to populate Universal's **Debug** output. The persisted `DebugMode` setting also enables timestamped `[Debug]` console output for OS/headless tasks. Dynamic controls, settings validation, scheduler outcomes, RCON, backups/restores, and local/remote operation output are traced. Disabling debug stops diagnostics without suppressing normal errors; text-field values are omitted and known credential values are redacted.

When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the app or **Exit program** to close anyway.

Companion has its own default-off `"DebugMode": false` setting in `RCT.json`. Run `css_rct_debug 1` through the server console or RCON to enable and persist it; `css_rct_debug 0` disables it. `css_rct_status` reports its state. `[RCT DEBUG]` entries use the CounterStrikeSharp information log and trace command dispatch without raw arguments, configuration saves, mode ownership, menus, map priority, spawns/deaths, and Weapon Rotation loadouts, changes, votes, and announcements. State changes are traced rather than every tick.

Enabling Universal debug does not enable Companion debug. Review all diagnostic output before sharing; redaction is a safeguard, not a guarantee against arbitrary secrets embedded in custom output.

## Data Storage And Security

Writable data is stored per user, not beside the executable:

- Windows: `%APPDATA%\CS2RconTool`
- Linux: typically `~/.config/CS2RconTool`
- macOS: typically `~/Library/Application Support/CS2RconTool`

The directory contains:

- `servers.json`: profiles and protected credentials.
- `tasks.json`: scheduled tasks and saved-server references (older tasks may retain protected credentials until edited).
- `steam_settings.json`: application settings and protected tokens.
- `fun_stuff_state.json`: per-server Fun Stuff state without credentials.
- `app_state.json`: settings schema, last validated app version, and validation time.
- `migration-backups/`: invalid settings files moved aside during startup recovery.
- `credential.key`: the per-user AES key.
- `GeoLite2-Country.mmdb` and `flags/`: writable GeoIP assets.

Saved secrets use AES-256-GCM. Protect `credential.key` as carefully as the JSON files. Release archives intentionally exclude settings, keys, caches, PDB files, ChatRelay binaries, and Companion binaries.

## Updating Or Migrating Data

Close the application before updating. Replace application files or extract the new package to a new folder; on macOS, replace the entire app bundle. Back up the complete per-user `CS2RconTool` directory first.

For migration:

1. Open the new application once, then close it.
2. Back up its generated per-user data directory.
3. Copy `servers.json`, `tasks.json`, `steam_settings.json`, and `fun_stuff_state.json` from the old installation.
4. Copy the matching original `credential.key` with the encrypted files. Do not mix files and keys from different installations.
5. Reopen the application.

If protected settings cannot be decrypted, loading stops with an error. Restore the matching original key and data files; the application does not replace the key or silently load empty credentials.

Remote profiles remain portable when valid credentials and target paths are supplied. New profiles default to the local type supported by the current OS. Existing imported profile locations are not changed automatically, and an incompatible local profile must be changed to a supported location before it can be saved.

Hosting providers with a Pterodactyl-compatible panel but no SSH can use **Hosting panel API** for Remote Linux lifecycle control. Configure the panel URL, server identifier/UUID, and encrypted client API key, then keep SFTP credentials for plugin deployment, browsing, and file operations. This mode uses the panel API for Start, Stop, and Restart and does not attempt SteamCMD validation or remote shell commands. Debug Mode records panel requests and state polling without logging the API key.

For hosted profiles, set **Framework management** to **Host-managed** when the provider preloader owns Metamod and CounterStrikeSharp. In Host-managed mode, the app blocks framework installation and upgrades but continues to support optional CounterStrikeSharp plugin deployment over SFTP. Use **Application-managed** only when the server provider does not manage those frameworks.

- When moving to Windows, change `Local Linux` to `Local Windows`, then replace the Linux executable, SteamCMD, install-directory, working-directory, and launch paths with Windows paths.
- When moving to Linux, change `Local Windows` to `Local Linux`, then replace the Windows executable, SteamCMD, install-directory, working-directory, and launch paths with Linux paths.
- macOS cannot run local CS2 profiles. Convert an imported local profile to the correct Remote Linux or Remote Windows type and provide valid target paths and credentials.

## Troubleshooting

### RCON commands fail

- Re-enter and save the RCON password, especially after migration.
- Verify host/IP and TCP port.
- Confirm the server's `rcon_password` matches the profile.
- Confirm firewalls permit the RCON TCP connection.
- Use Application Log and Debug for connection details.

### A Linux server in a bridged VirtualBox VM is unreachable

- Set the app's server host/IP to the Linux guest VM's bridged LAN address, not the Windows host's address or `127.0.0.1`.
- Ensure the VM's `/etc/hosts` maps `127.0.1.1` to the VM hostname.
- Add `-ip 0.0.0.0` to Direct-process launch arguments so CS2 listens on every interface; put the chosen `-ip` value in the service unit for Service commands mode.
- Confirm the VirtualBox adapter is in Bridged Adapter mode. If a Host-Only adapter such as `Ethernet 2` interferes with routing, disable it while keeping the physical Ethernet/Wi-Fi adapter enabled.
- Confirm guest and host firewalls permit the configured game and RCON ports.

### Local server exits immediately

- Check Application Log and Debug for the exit code and stderr.
- Verify the executable, working directory, and launch arguments.
- Confirm the local profile type matches the current OS.
- On Linux, verify required runtime libraries and Steam client files.

### Remote lifecycle, browsing, or installation fails

- Verify SSH host, port, username, and password.
- Re-enter the SSH password after migration if necessary.
- Confirm the account can run lifecycle commands and write to the install directory.
- Confirm the target has the archive tools required by the operation.

### Metamod or CounterStrikeSharp commands are unknown

- Install or validate Metamod before CounterStrikeSharp.
- Confirm `gameinfo.gi` contains the Metamod search path.
- Restart CS2 after framework installation or repair.
- Run `meta list` and `css_plugins list` from Console Commands.
- Use the built-in command-availability test.

### Join Server opens CS2 but does not connect

- Open the CS2 developer console with `~`.
- Paste the copied `connect IP:port` command and press Enter.
- Enable the developer console in CS2 settings if needed.

### Chat does not appear

- Verify ChatRelay is installed and running separately.
- Verify destination IP, adapter, UDP port and matching 32-byte secure keys on both sides.
- After upgrading, provision the encrypted sender key; legacy plaintext sender and receiver paths are removed. Stale token-only profiles fail closed with a migration message and require a matching secure key. Stop/re-enable the listener after switching profiles.
- Review existing firewall rules and use the Chat tab's **Test** action for local UDP only; actual remote player chat is needed to verify the plugin path.

### Scheduled tasks do not run

- Confirm the task is active and its trigger/time is valid.
- Start monitoring for built-in tasks and keep the app open.
- For native tasks, inspect Task Scheduler, the user systemd timer, or LaunchAgent.
- Confirm the selected profile and credentials are valid.

### Country or flag data is missing

- Select **Settings > General > Update GeoLite + flags**.
- Confirm the configured URLs are direct-download sources.
- Confirm the per-user data directory is writable.

Platform-specific diagnostics:

- [Windows troubleshooting](README-WINDOWS.md#troubleshooting)
- [Linux troubleshooting](README-LINUX.md#troubleshooting)
- [macOS troubleshooting](README-MACOS.md#troubleshooting)

## Building Release Packages

For all four Universal platforms, use `.\packaging\Prepare-LocalRelease.ps1 -OutputDirectory '<empty release directory>'`. It reads the project version, publishes from the current source, includes the documentation/screenshots, builds macOS tar.gz bundles with executable permissions, verifies every archived payload hash, and writes checksums. It refuses nonempty output directories and never publishes to GitHub or deploys servers.

After finalizing **all** archives in a release/test-bundle directory, generate the checksum files locally:

```powershell
.\packaging\Write-PackageChecksums.ps1 -Directory 'artifacts\<bundle-directory>'
```

This writes `<archive>.sha256` for every ZIP/tar.gz and a sorted `SHA256SUMS.txt` for that directory. Run it again after changing any archive, including an all-platform bundle. The Linux packaging script already writes its archive's sidecar; use this final step to refresh the complete bundle manifest. macOS packages remain tar.gz only. Add `-Verbose` for packaging diagnostics; no credentials or package contents are logged.

RCT, ChatRelay, and PlayerPunishments generate their sidecars with their own packaging scripts in their separate workspaces. When publishing is explicitly approved, include the matching checksum files and confirm GitHub's release-asset `digest` matches each final archive's SHA-256. Universal's plugin installer uses that API digest automatically; it does not discover `.sha256` files. A sidecar is useful for independent/manual verification but does not bypass missing-digest approval or prove publisher identity.

These commands only create local files; they do not upload, publish, or deploy. For a local regression check, run `.\testing\Tools\PackageChecksumCheck.ps1`.

Finalized notes are read-only locally and registered in `testing/historical-release-notes.json`; plugin baselines live in each plugin's `tests/historical-release-notes.json`. The TestLab release-note suites read current project versions and verify historical notes before accepting new notes. Do not modify finalized notes, remove baselines, or regenerate their hashes to bypass a failure.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## About

Open **Help > About** to view the application version, developer information, and detected platform.

## Application Screenshots

Screenshots show the shared application UI with fictional server details. Platform-specific window borders are omitted.

### 1. Add a server

| Hosting and startup | Connection details |
| --- | --- |
| ![Add server wizard hosting and startup](docs/images/add-server-wizard-type.png) | ![Add server wizard connection details](docs/images/add-server-wizard-connection.png) |
| Review and installation | Saved servers |
| ![Add server wizard review and installation choices](docs/images/add-server-wizard-review.png) | ![Manage servers](docs/images/manage-servers.png) |

### 2. Configure the server

| Remote Linux settings | Transport security |
| --- | --- |
| ![Edit a Remote Linux server](docs/images/edit-remote-linux-server.png) | ![SSH tunnel and trusted host fingerprint configuration](docs/images/transport-security.png) |
| Integrations | Plugin management |
| ![Edit server integrations](docs/images/edit-server-integrations.png) | ![CounterStrikeSharp plugin management](docs/images/plugin-management.png) |

### 3. Manage the running server

| Server overview | Server context menu |
| --- | --- |
| ![Server overview](docs/images/server-overview.png) | ![Server context menu](docs/images/server-context-menu.png) |
| Server actions | Console commands |
| ![Server actions](docs/images/server-actions.png) | ![Console command suggestions](docs/images/console-commands.png) |
| Metamod controls | CounterStrikeSharp controls |
| ![Metamod controls](docs/images/settings-metamod-menu.png) | ![CounterStrikeSharp controls](docs/images/settings-counterstrikesharp-menu.png) |

### 4. Customize gameplay and schedule tasks

| Bot Control | Fun Stuff modes |
| --- | --- |
| ![Bot Control](docs/images/bot-control.png) | ![Fun Stuff mode selection](docs/images/fun-stuff-modes.png) |
| Scheduled tasks | Add scheduled task |
| ![Scheduled tasks](docs/images/scheduled-tasks.png) | ![Add scheduled task](docs/images/add-scheduled-task.png) |

### 5. Chat and troubleshooting

| Chat | Application log |
| --- | --- |
| ![Chat](docs/images/chat-tab.png) | ![Application log](docs/images/application-log-tab.png) |

**Debug output**

![Debug output](docs/images/debug-tab.png)

### 6. Application preferences

**Settings menu**

![Settings menu](docs/images/settings-menu.png)

| General settings | Backup settings |
| --- | --- |
| ![General settings](docs/images/settings-general.png) | ![Backup settings](docs/images/settings-backups.png) |
