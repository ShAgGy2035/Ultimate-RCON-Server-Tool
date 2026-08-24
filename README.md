# ShAgGy's Ultimate CS2 RCON/Server Tool

Version: 1.0.0  
Developer: ShAgGy

A shared Avalonia desktop application for managing Counter-Strike 2 dedicated servers from Windows, Linux, and macOS. All releases use the same C# codebase and shared user interface.

## What This App Does

The application provides one place to:

- Query CS2 server information and connected players.
- Run RCON commands with categorized suggestions and autocomplete.
- Start, stop, restart, install, update, and validate supported servers.
- Manage maps, bots, teams, passwords, friendly fire, cheats, and gameplay settings.
- Apply Fun Stuff presets such as AWP, Bhop, Surf, and Zeus Wars.
- Join a selected server through a Steam connection URI.
- Install or upgrade Metamod and CounterStrikeSharp.
- Install, upgrade, track, and safely uninstall supported optional plugins.
- Back up and restore server files and MySQL/MariaDB databases.
- Schedule RCON and maintenance operations.
- Receive authenticated game chat from the separate ChatRelay plugin over UDP.
- Maintain GeoLite country information and a 195-country flag set.

## Supported Platforms

| Application package | Local CS2 hosting | Remote Linux | Remote Windows | Native scheduler |
| --- | --- | --- | --- | --- |
| Windows x64 | Yes, with `Local Windows` | Yes | Yes | Windows Task Scheduler |
| Linux x64 | Yes, with `Local Linux` | Yes | Yes | systemd user timers |
| macOS Intel | No | Yes | Yes | launchd user agents |
| macOS Apple Silicon | No | Yes | Yes | launchd user agents |

macOS supports RCON and remote administration, but Valve does not publish a native macOS CS2 dedicated-server runtime. Imported local profiles for another OS can be displayed, but local lifecycle and installation actions only work when the profile matches the current OS.

Remote Linux and Remote Windows management use SSH/SFTP. Remote Windows does not require WinRM.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete CS2 installations, plugins, backups, or files from the managed server.

## Release Packages

- `cs2-rcon-tool-universal-v1.0.0-windows-x64.zip`
- `cs2-rcon-tool-universal-v1.0.0-linux-x64.zip`
- `cs2-rcon-tool-universal-v1.0.0-macos-x64.zip`
- `cs2-rcon-tool-universal-v1.0.0-macos-arm64.zip`
- `cs2-rcon-tool-universal-v1.0.0-all-platforms.zip`

Each platform build has one self-contained, single-file native application. A separate .NET installation is not required. The GeoLite database and flag images remain external runtime assets by design.

## Download And Launch

### Windows x64

1. Extract the Windows ZIP to a folder you control.
2. Run `cs2-rcon-tool.exe`.

This is a GUI application. A command prompt does not need to remain open.

### Linux x64

1. Extract the Linux ZIP.
2. Open a terminal in the extracted directory.
3. Restore executable permissions:

   ```bash
   chmod +x cs2-rcon-tool run.sh
   ```

4. Launch through the included script:

   ```bash
   ./run.sh
   ```

Use `./run.sh --foreground` to keep the process attached for troubleshooting. The launcher uses an app-local `.net` directory for extracted native single-file libraries. It is a cache, not a separately installed .NET runtime, and can be deleted while the app is closed.

### macOS Intel And Apple Silicon

Choose the x64 package for Intel Macs or arm64 for Apple Silicon, extract it, and open `CS2 RCON Tool.app`.

The macOS packages are unsigned and not notarized. If Gatekeeper quarantines a test build, run:

```bash
xattr -dr com.apple.quarantine "CS2 RCON Tool.app"
chmod +x "CS2 RCON Tool.app/Contents/MacOS/cs2-rcon-tool"
open "CS2 RCON Tool.app"
```

## First-Time Setup

1. Open **Servers > Manage servers**.
2. Add a server and enter its name, host/IP, game/RCON port, RCON password, and location.
3. Complete the location-specific lifecycle or SteamCMD fields when the app will manage those operations.
4. Add the per-server Workshop collection ID when Workshop map selection is needed.
5. Configure database and ChatRelay integration values when used.
6. Save the profile and select it in the top server list.
7. Click **Refresh server list** to query map, player, ping, and country information.

RCON uses TCP. The configured RCON port must be reachable from the computer running this app; opening only the UDP game port is not sufficient.

## Server Profiles

### Local Windows

Use this profile only in the Windows application. Configure:

- CS2 executable, normally `<install-dir>\game\bin\win64\cs2.exe`.
- Working directory and launch arguments.
- Game type, game mode, and maximum players.
- SteamCMD path, install directory, App ID `730`, and Steam login.
- Optional Steam API authentication key and game-server login token.
- Local map, Workshop collection, or single Workshop map startup mode.

The app can download Windows SteamCMD as a ZIP when bootstrapping a local installation.

### Local Linux

Use this profile only in the Linux application. Configure:

- CS2 executable, normally `<install-dir>/game/bin/linuxsteamrt64/cs2`.
- Working directory and launch arguments.
- Game type, game mode, and maximum players.
- SteamCMD path, install directory, App ID `730`, and Steam login.
- Optional Steam API authentication key and game-server login token.
- Local map, Workshop collection, or single Workshop map startup mode.

On supported Linux systems, the app can check SteamCMD prerequisites and bootstrap SteamCMD. Local profiles can enable startup maintenance, which warns players, stops the server, updates CS2 and supported add-ons, and restarts it.

### Remote Linux

Configure:

- SSH host, port, username, and password.
- Start and stop commands, normally non-interactive service commands.
- Remote SteamCMD path and CS2 install directory for managed updates.
- App ID `730` and Steam login, normally `anonymous`.

Remote path fields include SFTP browsing. The remote account must run the configured commands and write to the installation directory. Default Linux service commands use `sudo -n`, which requires suitable non-interactive sudo permissions.

### Remote Windows

Configure:

- SSH host, port, username, and password.
- Windows start and stop commands.
- Windows SteamCMD path, installation directory, App ID `730`, and login.

The default SteamCMD path is `C:\steamcmd\steamcmd.exe`. Remote browsing and transfers use SFTP, and saved paths are normalized to drive-letter form where applicable.

## Fresh Server Installation And Updates

Right-click a configured server and select **Install/Update server...**.

- Local Windows can download and run Windows SteamCMD.
- Local Linux can download and run Linux SteamCMD and check supported prerequisites.
- Remote Linux and Remote Windows run configured SteamCMD operations over SSH.

Use App ID `730`. Verify SteamCMD path, installation directory, and remote credentials before starting. Wait for completion before launching the server or installing add-ons.

**Install/Update Server + Add-ons** combines the server update with supported framework and tracked-plugin maintenance.

## Joining A Server

Select a server and click **Join server**.

- Windows uses the registered Steam URI handler.
- Linux tries Steam and available desktop URI handlers.
- macOS uses the native `open` command.
- If CS2 is not detected, the app copies `connect IP:port` and shows a confirmation window before launching Steam.

If Steam opens CS2 without connecting, enable the CS2 developer console, press `~`, paste the copied `connect` command, and press Enter.

## Main Areas Of The App

### Top Server List

- Displays hostname, address, port, map, players, ping, location, and country.
- Provides Join, Reload, and Refresh actions.
- The context menu includes copy, lifecycle, installation, add-on, backup, restore, validation, and edit operations.
- **Reload server list** reads saved profiles without contacting servers.
- **Refresh server list** queries live A2S information.

### Server Actions

Controls include hostname, bots, server password, friendly fire, cheats, pause, cfg execution, teams, map changes, timed restart, and player kick/ban/slay/slap actions.

Player Alive and Health values depend on the required server-side command. Slay and Slap use CounterStrikeSharp-compatible commands. PlayerPunishments 1.0.0 (https://github.com/ShAgGy2035/PlayerPunishments) or newer can provide them when the installed administration package does not.

### Fun Stuff

Preset modes include AWP/Sniper Wars, Bhop, Deathmatch, Grenade Wars, Headshot Only, Knife Arena, Pistols Only, ScoutzKnives, Surf, and Zeus Wars. Choose **None (rollback)** to restore the cvars changed by a preset.

### Server Overview

Shows a read-only snapshot of the selected server, endpoint, map, players, and ping.

### Scheduled Tasks

Task types include RCON command, framework/plugin maintenance, server backup, database backup, and GeoLite/flag updates. Triggers include Day, Hour, and Startup.

Each OS displays its supported external scheduler:

- Windows Task Scheduler on Windows.
- systemd user timers on Linux.
- launchd user agents on macOS.

The built-in scheduler is available everywhere and runs while the app is open and scheduler monitoring is started. Native external tasks invoke the headless task runner and do not require the GUI to remain open.

### Console Commands

- Enter free-form RCON commands or use categorized commands and autocomplete.
- Press Enter or click **Execute** to send a command.
- Each server keeps a separate bounded console history.
- Process, SteamCMD, SSH, and RCON output remains associated with its originating server.
- Local Start and Restart show initial process output only through the startup check, then stop forwarding continuous CS2 runtime output. Explicit local-console fallback commands temporarily reopen output capture for their response.
- Sensitive commands and debug output redact passwords, tokens, and authentication keys.

### Chat

- Receives authenticated JSON chat messages from ChatRelay over UDP.
- Selects a local adapter, listen address, and port; the default is `9090`.
- Uses the selected server's token from **Edit server > Integrations**.
- Includes a local authenticated Test action and can send admin chat through RCON.

### Application Log And Debug

- **Application Log** shows normal operations and user-facing failures.
- **Debug** shows detailed diagnostics when Debug Mode is enabled.
- Tracing covers windows, controls, lifecycle, RCON, scheduling, chat, SteamCMD, and add-on work.
- Centralized debug messages redact passwords, tokens, API keys, and similar secrets.

## Metamod And CounterStrikeSharp

Right-click a server and use:

1. **Install/Upgrade Metamod**.
2. **Install/Upgrade CSS**.

CounterStrikeSharp requires Metamod. The app resolves platform-appropriate assets, deploys them, validates loader files, and repairs the Metamod `gameinfo.gi` search path where needed. Linux installation also handles the known executable-stack requirement on affected CounterStrikeSharp ELF libraries.

Under **Settings > Metamod Settings**, the app can list plugins and run info, pause, unpause, retry, load, unload, or force-unload operations. Under **Settings > CStrikeSharp Settings**, it can reload admins and list, reload, or unload plugins. These operations query plugin lists and use the numeric IDs expected by each framework.

Use **Settings > Test Metamod/CSS command availability** to probe safe command forms with nonexistent plugin IDs. It does not target real plugins.

## Optional Plugins

Use **Install/Upgrade Metamod Plugins...** or **Install/Upgrade CSS Plugins...**. Packages can come from a GitHub repository/release or a supported local archive or library.

Installed files are tracked per server in:

```text
game/csgo/.cs2-rcon-tool-plugins.json
```

The ownership manifest lets uninstall remove only owned files, preserve shared files and common configuration, and protect framework files. Keep it with the server.

Optional plugin deployment is supported for compatible local profiles and Remote Linux. Framework installation supports Local Windows, Local Linux, Remote Windows, and Remote Linux. Do not assume every optional plugin publishes binaries for every server OS.

## Backups, Restore, And Validation

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking rather than stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` with the selected server's integration settings.
- **Restore server backup...** accepts ZIP, TAR.GZ, or TGZ, validates it, stops the server, restores supported content, and restarts it.
- **Install/update + restore server backup...** updates core files before restoring the backup.
- **Restore database backup...** imports SQL with the configured MySQL client.
- **Validate server files** runs SteamCMD with `validate`.

Server and database backup destinations are configured separately under **Settings > Backups**. Remote operations require suitable SSH permissions and archive tools on the managed host.

## ChatRelay

ChatRelay is a separate CounterStrikeSharp server plugin and is not included in these packages.

Repository: <https://github.com/ShAgGy2035/ChatRelay>

1. Install ChatRelay on the CS2 server.
2. Set its destination IP to the machine running this app.
3. Configure the same UDP port on both sides; the default is `9090`.
4. Set the same token in ChatRelay and **Edit server > Integrations**.
5. Select the correct local adapter and enable the listener.
6. Use **Test** to verify the local authenticated listener.

Allow the UDP port through the receiving computer's firewall. Remote networks may require routing or port forwarding.

## GeoLite And Flags

Release packages include `GeoLite2-Country.mmdb` and 195 country flags. The app copies missing assets into its writable data directory and can update them from **Settings > General > Update GeoLite + flags**.

Settings includes the database URL and flag URL template so trusted direct-download sources can be selected. Missing assets are self-healing when network access is available.

## Data Storage And Security

Writable data is stored per user, not beside the executable:

- Windows: `%APPDATA%\CS2RconTool`
- Linux: typically `~/.config/CS2RconTool`
- macOS: typically `~/Library/Application Support/CS2RconTool`

The directory contains:

- `servers.json`: server profiles and protected credentials.
- `tasks.json`: scheduled tasks and protected credentials.
- `steam_settings.json`: application settings and protected tokens.
- `credential.key`: the per-user AES key.
- `GeoLite2-Country.mmdb` and `flags/`: writable GeoIP assets.

Saved secrets use AES-256-GCM. Protect `credential.key` as carefully as the JSON files. Files beside the executable are packaged assets, not active writable configuration. Release ZIPs intentionally exclude settings JSON, credential keys, caches, PDB files, and ChatRelay binaries.

## Reinstalling Or Migrating Existing Data

For a clean installation or migration from another device:

1. Extract and open the new application once.
2. Close it completely.
3. Locate the new per-user `CS2RconTool` directory listed above.
4. Back up the generated directory before replacing anything.
5. Copy `servers.json`, `tasks.json`, and `steam_settings.json` from the old installation.
6. Copy the matching old `credential.key` when available.
7. Open the application again.

Configuration can load even when encrypted credentials cannot be decrypted. After a clean install, device change, missing key, or key mismatch, edit each affected profile and re-enter its RCON password. You may also need to re-enter SSH passwords, database passwords, ChatRelay tokens, Steam authentication keys, and game-server login tokens. Save each profile so those values are encrypted for the current installation.

When importing across operating systems:

- Remote Linux and Remote Windows profiles remain usable after valid credentials and paths are supplied.
- Change `Local Linux` to `Local Windows` for local Windows management, then replace Linux executable, SteamCMD, install-directory, and launch paths.
- Change `Local Windows` to `Local Linux` for local Linux management and replace Windows paths.
- macOS cannot run either local CS2 profile type.

## Troubleshooting

### RCON commands fail

- Re-enter and save the RCON password, especially after migration.
- Verify host/IP and TCP port.
- Confirm the server's `rcon_password` matches the profile.
- Confirm firewalls permit the RCON TCP connection.
- Use Application Log and Debug for connection details.

### A Linux server in a bridged VirtualBox VM is unreachable

- Set the app's server host/IP to the Linux guest VM's bridged LAN address, such as `10.0.1.3`, not the Windows host's address or `127.0.0.1`.
- Add `+ip 0.0.0.0` to the local Linux server launch arguments so CS2 listens on every network interface in the guest, then restart the server.
- Confirm the VirtualBox adapter is in Bridged Adapter mode and that guest and host firewalls permit the configured game and RCON ports.
- On the Linux guest, verify the listening sockets with `ss -lntup | grep 27015`, replacing `27015` when the profile uses another port.

### Local server exits immediately

- Check Application Log and Debug for exit code and stderr.
- Verify executable, working directory, and launch arguments.
- Confirm the local profile type matches the current OS.
- On Linux, verify required runtime libraries and Steam client files.

### Remote lifecycle, browsing, or installation fails

- Verify SSH host, port, username, and password.
- Re-enter the SSH password after migration if necessary.
- Confirm the account can run lifecycle commands and write to the install directory.
- Confirm the target has archive tools required by the operation.

### Metamod or CounterStrikeSharp commands are unknown

- Install or validate Metamod first, then CounterStrikeSharp.
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
- Verify destination IP, adapter, UDP port, and token on both sides.
- Re-enter the token after migration if necessary.
- Check firewall rules and use the Chat tab's Test action.

### Scheduled tasks do not run

- Confirm the task is active and its trigger/time is valid.
- Start monitoring for built-in tasks and keep the app open.
- For native tasks, inspect Task Scheduler, the user systemd timer, or LaunchAgent.
- Confirm the selected profile and credentials are valid.

### Country or flag data is missing

- Select **Settings > General > Update GeoLite + flags**.
- Confirm the configured URLs are direct-download sources.
- Confirm the per-user data directory is writable.

## About

Open **Help > About** to view version `1.0.0`, developer information, and the detected application platform.
