# Ultimate CS2 RCON and Server Management Tool - Windows Guide

Version: 1.0.4

This guide covers the Windows x64 application and Windows CS2 server profiles. For Linux server setup, including systemd, see [README-LINUX.md](README-LINUX.md). For shared features and project information, see [README.md](README.md).

- **Windows application**
  - [Install and launch](#install-and-launch)
  - [Update the application](#update-the-application)
  - [Configuration and data](#configuration-and-data)
   - [PlayerPunishments setup](#playerpunishments-setup)
- **Local Windows server**
  - [Choose a lifecycle mode](#choose-a-local-lifecycle-mode)
  - [Direct process](#local-windows-direct-process)
  - [Service commands](#local-windows-service-commands)
- **Remote Windows server**
  - [Requirements](#remote-windows-requirements)
  - [Direct process](#remote-windows-direct-process)
  - [Service commands with WinSW](#remote-windows-service-commands-with-winsw)
- [SteamCMD behavior](#steamcmd-behavior)
- [Troubleshooting](#troubleshooting)

## Install And Launch

1. Extract `cs2-rcon-tool-universal-v1.0.4-windows-x64.zip` to a folder you control.
2. Run `cs2-rcon-tool.exe`.

This is a self-contained GUI application. A separate .NET installation and persistent command prompt are not required.

## Update The Application

Close the application, then either replace the existing application files with the contents of the new Windows ZIP or extract the new version to a new folder.

Writable settings are stored under `%APPDATA%\CS2RconTool`, not beside the executable. Back up that entire directory, including `credential.key`, before updating. Do not replace or delete it while updating application files.

## Configuration And Data

Open **Settings > Settings** for application-wide GeoLite, flag, slap-damage, refresh, output-filtering, and backup-directory options. Configure lifecycle, SteamCMD, RCON, database, ChatRelay, and Steam authentication values per server under **Servers > Manage servers > Edit server**.

The Windows data directory contains server profiles, scheduled tasks, application settings, protected credentials, Fun Stuff state, GeoLite data, and flags:

```text
%APPDATA%\CS2RconTool
```

Saved secrets use AES-256-GCM. Keep `credential.key` with the JSON files when backing up or migrating data.

### PlayerPunishments Setup

The app's Slay and Slap controls send `css_slay #userid` and `css_slap #userid damage`. Install [PlayerPunishments 1.0.0 or newer](https://github.com/ShAgGy2035/PlayerPunishments) on each managed CS2 server when its existing administration package does not provide those commands. Configure Slap damage from `0` to `99` under **Settings > General**.

PlayerPunishments is a CounterStrikeSharp server plugin. Install it in the Windows CS2 server's CounterStrikeSharp plugin directory, not in the desktop application's folder. Alive and Health display **Unknown** when the required server-side player-information command is unavailable.

## Local Windows Server

Use a **Local Windows** profile only when the application and CS2 server run on the same Windows computer.

Configure the RCON password, CS2 install directory, SteamCMD path, App ID `730`, startup map, game type, game mode, maximum players, tickrate, VAC state, LAN mode, Startup CFG, and optional player password. Startup can use a local map, Workshop collection, or single Workshop map. The editable map field suggests Valve maps, including Premier maps, while accepting custom names. Public internet servers can also use a Steam Web API key and game-server login token. These credentials are not required or prompted for when **LAN server** is enabled.

### Choose A Local Lifecycle Mode

- **Direct process**: the application builds the CS2 launch command and owns the process.
- **Service commands**: an existing Windows service wrapper owns CS2 and the application runs configured PowerShell Start, Stop, and Restart commands.

Stop the current owner before switching modes. A Windows service cannot adopt a CS2 process that was already launched directly.

### Local Windows Direct Process

Use this mode when the application should launch and stop `cs2.exe` itself.

Typical values:

```text
Executable:        <install-dir>\game\bin\win64\cs2.exe
Working directory: <install-dir>\game
Arguments:         -dedicated +ip 0.0.0.0
```

The application adds authoritative map, game type, game mode, CFG, port, hostname, RCON, tickrate, VAC, LAN, Steam API, GSLT, and optional player-password arguments. Local Windows launches CS2 without opening a native console window and uses RCON for command responses.

Windows SteamCMD is stored in a sibling `<install-dir>-steamcmd` directory because SteamCMD rejects a game installation inside its own directory. Existing profiles that used `<install-dir>\steamcmd` are migrated without redownloading an already completed app `730` payload. An ordinary Direct-process Start runs SteamCMD validation when both SteamCMD and install-directory settings are configured. Restart does not run SteamCMD validation.

### Local Windows Service Commands

Use this mode when a wrapper such as WinSW already manages CS2. The application does not install or configure the service.

Default commands target `cs2-server`:

```powershell
Start-Service -Name 'cs2-server'
Stop-Service -Name 'cs2-server'
Restart-Service -Name 'cs2-server'
```

The account running the application must have permission to control the service. Put all CS2 launch arguments, including `-tickrate`, `+sv_lan`, and `-secure` or `-insecure`, in the service wrapper configuration. Service Start and Restart do not run SteamCMD validation.

Use the WinSW example below for either a local or remote Windows service. For a local profile, enter the commands above without an outer `powershell -Command` wrapper because the application already invokes local PowerShell.

## Remote Windows Server

Use a **Remote Windows** profile when CS2 runs on another Windows computer.

### Remote Windows Requirements

- OpenSSH server access to the remote Windows computer.
- The SSH account must belong to the remote computer's local Administrators group for WMI process creation and firewall configuration.
- Optional SFTP settings for browsing and general file transfers. Blank values reuse the SSH host and credentials; port `0` reuses the SSH port. Valid SFTP ports range through `65535`.
- A Windows SteamCMD path and CS2 installation directory for managed updates.

Remote Windows lifecycle and plugin deployment do not require WinRM. Direct process startup and plugin transfers do not require a separate SFTP service.

Remote Windows profiles reject Unix paths before SteamCMD starts. When a profile is changed from Remote Linux to Remote Windows, stale auto-generated executable and working-directory defaults are replaced with Windows defaults. Existing overlapping or former global SteamCMD defaults are migrated automatically; an explicitly configured non-overlapping custom path remains unchanged.

### Remote Windows Direct Process

Use this mode when the application should launch CS2 over SSH without installing a service wrapper.

For an install directory of `C:\cs2server`, the application derives:

```text
Executable:        C:\cs2server\game\bin\win64\cs2.exe
Working directory: C:\cs2server\game
SteamCMD path:     C:\cs2server-steamcmd\steamcmd.exe
```

Configure the startup map, game type, game mode, additional arguments, and Stop command. The application supplies `-dedicated`, `-console`, `+ip 0.0.0.0`, port, hostname, map, game type, game mode, Startup CFG, RCON password, tickrate, LAN mode, and VAC state. Optional Steam API and GSLT credentials are appended and redacted from output.

Start transfers a temporary PowerShell launcher over SSH, starts CS2 through WMI independently of the SSH session, creates inbound UDP and TCP firewall rules for the configured port, captures startup output, and monitors the process for 30 seconds. Restart performs the configured Stop command and then launches a new process.

Direct mode refuses to control the configured executable while the `cs2-server` Windows service is actively managing it. It does not automatically start CS2 at operating-system boot or restart it after a crash.

### Remote Windows Service Commands With WinSW

Windows service mode requires a wrapper because `cs2.exe` does not implement the Windows Service Control Manager protocol. This example uses WinSW and the service name `cs2-server`. Run the setup from an Administrator PowerShell terminal on the Windows server. Stop any Direct-process CS2 instance first.

Install WinSW:

```powershell
$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'

New-Item -ItemType Directory -Force -Path C:\Services\cs2-server | Out-Null
Invoke-WebRequest `
    -Uri 'https://github.com/winsw/winsw/releases/download/v2.12.0/WinSW-x64.exe' `
    -OutFile 'C:\Services\cs2-server\cs2-server.exe'
```

If an earlier wrapper registered the same service, remove only its Windows service registration:

```powershell
$existingService = Get-Service -Name 'cs2-server' -ErrorAction SilentlyContinue
if ($existingService) {
   Stop-Service -Name 'cs2-server' -Force -ErrorAction SilentlyContinue
   sc.exe delete 'cs2-server' | Out-Null
   while (Get-Service -Name 'cs2-server' -ErrorAction SilentlyContinue) {
      Start-Sleep -Seconds 1
   }
}
```

Create `C:\Services\cs2-server\cs2-server.xml`. Replace the sample hostname, map, passwords, Steam API key, and GSLT. XML-escape special characters in values, especially `&` as `&amp;`, `<` as `&lt;`, and `>` as `&gt;`.

```xml
<service>
   <id>cs2-server</id>
   <name>CS2 Dedicated Server</name>
   <description>Counter-Strike 2 Dedicated Server managed by CS2 RCON Tool</description>
   <executable>C:\cs2server\game\bin\win64\cs2.exe</executable>
   <arguments>-dedicated -console -usercon -port 27015 -tickrate 64 -secure +ip 0.0.0.0 +sv_lan 0 +map de_dust2 +game_type 0 +game_mode 1 +exec server.cfg +hostname "CS2 Server" +rcon_password "&lt;rcon-password&gt;" -authkey "&lt;steam-api-key&gt;" +sv_setsteamaccount "&lt;gslt&gt;"</arguments>
   <workingdirectory>C:\cs2server\game</workingdirectory>
   <env name="SteamAppId" value="730" />
   <env name="SteamGameId" value="730" />
   <startmode>Automatic</startmode>
   <stoptimeout>30 sec</stoptimeout>
   <onfailure action="restart" delay="10 sec" />
   <logpath>C:\cs2server\logs</logpath>
   <log mode="roll-by-size">
      <sizeThreshold>10240</sizeThreshold>
      <keepFiles>8</keepFiles>
   </log>
</service>
```

Install the service:

```powershell
New-Item -ItemType Directory -Force -Path C:\cs2server\logs | Out-Null
Set-Location C:\Services\cs2-server
.\cs2-server.exe install
```

Permit game queries over UDP and RCON over TCP:

```powershell
New-NetFirewallRule -DisplayName 'CS2 RCON Tool UDP 27015' `
   -Direction Inbound -Action Allow -Protocol UDP -LocalPort 27015 `
   -Program 'C:\cs2server\game\bin\win64\cs2.exe' -Profile Any
New-NetFirewallRule -DisplayName 'CS2 RCON Tool TCP 27015' `
   -Direction Inbound -Action Allow -Protocol TCP -LocalPort 27015 `
   -Program 'C:\cs2server\game\bin\win64\cs2.exe' -Profile Any
```

Start and validate:

```powershell
Start-Service -Name 'cs2-server'
Get-Service -Name 'cs2-server'
Get-Process -Name 'cs2'
```

For a **Remote Windows** profile, configure:

```text
Startup mode: Service commands
Start:   powershell -NoProfile -NonInteractive -Command "Start-Service -Name 'cs2-server'"
Stop:    powershell -NoProfile -NonInteractive -Command "Stop-Service -Name 'cs2-server'"
Restart: powershell -NoProfile -NonInteractive -Command "Restart-Service -Name 'cs2-server'"
```

For a **Local Windows** profile, use the shorter commands shown under [Local Windows Service Commands](#local-windows-service-commands).

Service mode expects all CS2 launch arguments in the WinSW XML. It monitors the service-owned process and prevents Direct mode from managing the same executable. WinSW writes rolled output and error logs under `C:\cs2server\logs`.

## SteamCMD Behavior

| Windows profile and mode | Start | Restart |
| --- | --- | --- |
| Local Windows, Direct process | Runs validation when SteamCMD and install-directory settings are configured | Does not run validation |
| Local Windows, Service commands | Runs the configured Start command without validation | Runs the configured Restart command without validation |
| Remote Windows, Direct process | Prepares required runtime files and monitors startup without validation | Stops and starts without validation |
| Remote Windows, Service commands | Runs the service command and monitors its process without validation | Runs Restart and monitors its process without validation |

Use **Validate server files** or **Install/Update Server** for an explicit SteamCMD validation.

## Windows Plugin Deployment

Remote Windows transfers normalized plugin packages as acknowledged 32 KB parts over SSH to stay below the OpenSSH channel window. A stalled part is retried over a fresh connection, the assembled byte count is verified, transfer progress is reported, and temporary parts and partial archives are removed after failure. PowerShell extracts the package, preserves existing configuration files, and atomically replaces plugin binaries without requiring SCP or SFTP.

Remote Windows add-on downloads suppress PowerShell progress serialization so CLIXML telemetry cannot flood **Commands sent** or turn a successful command into a false failure. Installed payload checks use one constant-size PowerShell operation that reads the ownership manifest on the server instead of growing with its file count.

For local plugin batches, the application detects a running CS2 process, stops it once before replacing plugin and shared API assemblies, then restarts it without SteamCMD. This avoids Windows file-lock failures on loaded DLLs. Stop a Remote Windows service before upgrading plugins that replace loaded shared assemblies, then start it after the batch.

## Troubleshooting

- Re-enter SSH or RCON credentials after migration when decryption fails.
- `Connection refused` means the configured SSH host or port did not accept a connection.
- Authentication errors mean the SSH username or password was rejected.
- Direct-process startup requires the remote SSH account to be a local Administrator.
- Remote Windows Install/Update provisions Steamworks SDK Redist app `1007` and repairs required 64-bit Steam runtime DLLs beside `cs2.exe`.
- Remote Windows SteamCMD retries one incomplete update pass for `0x602`, `0x202`, or exit code `8`.
- Direct-process startup is monitored for 30 seconds and reports captured output on failure.
- Stop operations verify that the configured executable exited.
- If a local server exits immediately, check Application Log and Debug for its exit code and stderr, then verify the executable, working directory, launch arguments, and profile OS.
- Do not run Direct process and Service commands modes against the same executable and port.
