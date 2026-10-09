# Ultimate CS2 RCON and Server Management Tool - Windows Guide

Version: 1.0.5

This self-contained guide covers the Windows x64 application, Local Windows servers, and Remote Windows or Remote Linux servers. [README.md](README.md) provides the project-wide GitHub overview.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## Table of Content
- **Windows application**
  - [Install and launch](#install-and-launch)
  - [Update the application](#update-the-application)
  - [Configuration and data](#configuration-and-data)
- **Local Windows server**
  - [Quick start](#quick-start)
  - [Direct process](#local-windows-direct-process)
  - [Service commands](#local-windows-service-commands)
   - [Create a service with WinSW](#create-a-local-windows-service-with-winsw)
   - [Steamworks runtime for services](#steamworks-runtime-for-windows-services)
   - [Service control without elevation](#service-control-without-elevation)
   - [Finish Local Windows setup](#finish-local-windows-setup)
- **Remote Windows server**
  - [Requirements](#remote-windows-requirements)
   - [Create the Windows SSH user](#create-the-windows-ssh-user)
   - [Direct process](#remote-windows-direct-process)
  - [Service commands with WinSW](#remote-windows-service-commands-with-winsw)
   - [Finish Remote Windows setup](#finish-remote-windows-setup)
- **Remote Linux server**
   - [Requirements](#remote-linux-requirements)
   - [Account and permissions](#remote-linux-account-and-permissions)
   - [Remote Linux Hosting Panel](#remote-linux-hosting-panel)
   - [Direct process](#remote-linux-direct-process)
   - [Service commands with systemd](#remote-linux-service-commands-with-systemd)
   - [Finish Remote Linux setup](#finish-remote-linux-setup)
- [SteamCMD behavior](#steamcmd-behavior)
- [Using the application](#using-the-application)
- [RCON Tool Companion and Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments setup](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Frameworks and plugins](#frameworks-and-plugins)
   - [Install Frameworks And Plugins](#install-frameworks-and-plugins)
- [Backups, data, and security](#backups-data-and-security)
- [Troubleshooting](#troubleshooting)
- [Application Screenshots](#application-screenshots)

## Install And Launch

1. Extract `cs2-rcon-tool-universal-v1.0.5-windows-x64.zip` to a folder you control.
2. Run `cs2-rcon-tool.exe`.

This is a self-contained GUI application. A separate .NET installation and persistent command prompt are not required.

## Update The Application

Close the application, then either replace the existing application files with the contents of the new Windows ZIP or extract the new version to a new folder.

Writable settings are stored under `%APPDATA%\CS2RconTool`, not beside the executable. Back up that entire directory, including `credential.key`, before updating. Do not replace or delete it while updating application files.

The Windows data directory contains server profiles, scheduled tasks, application settings, protected credentials, Fun Stuff state, GeoLite data, and flags:

```text
%APPDATA%\CS2RconTool
```
Saved secrets use AES-256-GCM. Keep `credential.key` with the JSON files when backing up or migrating data.

## Local Windows Server

Use a **Local Windows** profile only when the application and CS2 server run on the same Windows computer.

### Quick Start

The Edit server window requires an **RCON password** before it will save any profile. To then run **Install/Update server...** and download CS2 for the first time, the only other required field is an **install directory**. Leave **SteamCMD path** blank and the application downloads and installs SteamCMD into a sibling folder automatically the first time it is needed; App ID defaults to `730` with an anonymous SteamCMD login.

You also need to pick a **Startup mode** before Install/Update or Install/Update + Add-ons, since it controls how the application later starts and stops CS2:

- **Direct process**: the application builds the CS2 launch command and owns the process.
- **Service commands**: an existing Windows service wrapper owns CS2 and the application runs configured PowerShell Start, Stop, and Restart commands.

Before you actually start the server for players, also configure the startup map, game type, game mode, maximum players, tickrate, VAC state, LAN mode, Startup CFG, and optional player password. Startup can use a local map, Workshop collection, or single Workshop map. The editable map field suggests Valve maps, including Premier maps, while accepting custom names. Public internet servers can also use a Steam Web API key and game-server login token; these are not required or prompted for when **LAN server** is enabled.

If you later change Startup mode on an already-running server, stop the current owner first; a Windows service cannot adopt a CS2 process that was already launched directly.

### Local Windows Direct Process

Use this mode when the application should launch and stop `cs2.exe` itself.

Typical values:

```text
Executable:        <install-dir>\game\bin\win64\cs2.exe
Working directory: <install-dir>\game (But not required)
Arguments:         -dedicated -ip 0.0.0.0
```

The application adds authoritative map, game type, game mode, CFG, port, hostname, RCON, tickrate, VAC, LAN, Steam API, GSLT, and optional player-password arguments. The default `-ip 0.0.0.0` binds CS2 to all available network addresses so it can accept connections through any configured interface; replace `0.0.0.0` with a specific address in the launch arguments when direct interface binding is required. Local Windows launches CS2 without opening a native console window, monitors Direct-process startup for 30 seconds, and uses RCON for command responses.

Windows SteamCMD is stored in a sibling `<install-dir>-steamcmd` directory because SteamCMD rejects a game installation inside its own directory. Existing profiles that used `<install-dir>\steamcmd` are migrated without redownloading an already completed app `730` payload. An ordinary Direct-process Start runs SteamCMD validation when both SteamCMD and install-directory settings are configured. Restart does not run SteamCMD validation.


### Local Windows Service Commands

Use this mode when a wrapper such as WinSW already manages CS2. The application does not install or configure the service. The service ID in the wrapper XML must match the service name used by the saved Start/Stop/Restart commands.

Default commands target `cs2-server`:

```powershell
Start-Service -Name 'cs2-server'
Stop-Service -Name 'cs2-server'
Restart-Service -Name 'cs2-server'
```

The account running the application must have permission to control the service. Put all CS2 launch arguments, including `-tickrate`, `+sv_lan`, and `-secure` or `-insecure`, in the service wrapper configuration. Normal Service-command Start runs SteamCMD validation when SteamCMD and install-directory settings are configured; Restart skips validation.

WinSW's default service permissions may deny Stop/Start from an unelevated app. Use the one-time [service-control permission setup](#service-control-without-elevation) below instead of routinely running Universal as administrator.

Use the complete Local WinSW setup below. Enter the commands above without an outer `powershell -Command` wrapper because the application already invokes local PowerShell.


### Create a Local Windows Service With WinSW

The commands above only control an existing service. To create a local service, perform these steps in an elevated PowerShell window on the Local Windows computer (not through SSH). WinSW requires `<id>` and `<executable>` in its XML; `<name>` is optional but supplies the friendly name shown in Services. Keep the wrapper and XML side by side with matching basenames: `cs2-server.exe` and `cs2-server.xml`. The `<id>` is `cs2-server`, matching the default commands above.

1. Choose a stable service ID, here `cs2-server`, and the CS2 install folder, here `D:\CS2Server`.
2. Download and name the WinSW wrapper:

```powershell
$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'
$wrapperDir = 'C:\Services\cs2-server'
New-Item -ItemType Directory -Force -Path $wrapperDir | Out-Null
Invoke-WebRequest `
   -Uri 'https://github.com/winsw/winsw/releases/download/v2.12.0/WinSW-x64.exe' `
   -OutFile (Join-Path $wrapperDir 'cs2-server.exe')
```

3. Create `C:\Services\cs2-server\cs2-server.xml`. The required RCON password is the value after `+rcon_password` on the `<arguments>` line; replace its placeholder with the same password configured in Universal. The Steam API key (`-authkey`) and GSLT (`+sv_setsteamaccount`) are optional; remove those switches if unused. Also replace the sample map, hostname, and install path. XML-escape special characters, especially `&`, `<`, and `>`.

```xml
<service>
   <id>cs2-server</id>
   <name>CS2 Dedicated Server</name>
   <description>Counter-Strike 2 Dedicated Server</description>
   <executable>D:\CS2Server\game\bin\win64\cs2.exe</executable>
   <arguments>-dedicated -console -usercon -port 27015 -tickrate 64 -secure -ip 0.0.0.0 +sv_lan 0 +map de_dust2 +game_type 0 +game_mode 1 +exec server.cfg +hostname "CS2 Server" +rcon_password "&lt;rcon-password&gt;" -authkey "&lt;steam-api-key&gt;" +sv_setsteamaccount "&lt;gslt&gt;"</arguments>
   <workingdirectory>D:\CS2Server\game</workingdirectory>
   <env name="SteamAppId" value="730" />
   <env name="SteamGameId" value="730" />
   <startmode>Automatic</startmode>
   <stoptimeout>30 sec</stoptimeout>
   <onfailure action="restart" delay="10 sec" />
   <logpath>D:\CS2Server\logs</logpath>
   <log mode="roll-by-size">
     <sizeThreshold>10240</sizeThreshold>
     <keepFiles>8</keepFiles>
   </log>
</service>
```

Before the first start, prepare the [Steamworks runtime for Windows services](#steamworks-runtime-for-windows-services). Use the actual install and SteamCMD directories, not the sample paths.

4. Create the log folder, install the service, and check that Windows registered it:

```powershell
New-Item -ItemType Directory -Force -Path 'D:\CS2Server\logs' | Out-Null
Set-Location 'C:\Services\cs2-server'
.\cs2-server.exe install
Get-Service -Name 'cs2-server'
sc.exe query 'cs2-server'
Get-CimInstance Win32_Service -Filter "Name='cs2-server'" |
   Select-Object Name, DisplayName, State, PathName
```

After registration, authorize the normal account that will control this service using [Service Control Without Elevation](#service-control-without-elevation). This is part of the one-time administrator-run service setup, before onboarding. Universal itself should run normally, not as administrator. If you do not want administrator-managed service setup, choose Direct process and an install directory your account owns instead.

5. If `Get-Service -Name 'cs2-server'` returns nothing, check for another registered CS2 service ID and inspect the wrapper installation output:

```powershell
Get-Service | Where-Object { $_.Name -match 'cs2' -or $_.DisplayName -match 'CS2' }
Get-CimInstance Win32_Service |
   Where-Object { $_.Name -match 'cs2' -or $_.DisplayName -match 'CS2' } |
   Select-Object Name, DisplayName, State, PathName
```

If no service is listed, installation did not register it. Rerun `.\cs2-server.exe install` from an elevated PowerShell window and read its console output. If `services.msc` was already open, press F5. Find the service by its XML `<id>`; `<name>` is only its display name.

If the service is registered but Start reports **Error 1053**, registration succeeded but WinSW did not complete startup in time. Check `cs2-server.wrapper.log` under the XML `<logpath>`, confirm `<executable>` and `<workingdirectory>` point to existing paths, and make sure a Direct-process CS2 instance is not already running. The Windows Service Control Manager event alone does not identify the underlying child-process error.

6. To reuse the registered service with another CS2 install folder, leave `C:\Services\cs2-server\cs2-server.exe`, `cs2-server.xml`, and `<id>cs2-server</id>` in place. Stop the service, edit `<executable>`, `<workingdirectory>`, `<logpath>`, and install-specific launch arguments in the XML, prepare the runtime in the new directory, and update executable-specific firewall rules before restarting it. WinSW reads the XML when the wrapper starts; changing the CS2 path does not change the Windows service registration. If you move the WinSW wrapper itself, uninstall it from the old wrapper directory before moving it, then install it from its new directory.
7. Add inbound UDP and TCP firewall rules for the configured game/RCON port, using the local `cs2.exe` path. The remote example below also includes sample firewall rules.
8. In Add server, choose **Local Windows** and **Service commands**. Set the Local CS2 install folder, review the prefilled Start/Stop/Restart commands, and change them if the registered service ID differs. The wizard saves the commands to the profile; launch arguments remain in WinSW XML.

Do not start the same CS2 installation with Direct process while the service owns it.

### Service Control Without Elevation

Windows requires authorization to start and stop a service. Creating it requires Administrator PowerShell, but daily control does not need the whole application elevated if your normal Windows account has permission on that specific service. File installation also requires write access to the CS2 and SteamCMD directories; this permission grant does not change directory permissions.

Run this once in Administrator PowerShell. Replace `COMPUTER\username` with the account that normally runs Universal (not a different administrator account). It preserves the existing service ACL, saves a backup, and grants only query/read, dependent-service enumeration, Start, and Stop rights for `cs2-server`; it does not grant service configuration changes or deletion:

```powershell
$ErrorActionPreference = 'Stop'
$serviceName = 'cs2-server'
$account = 'COMPUTER\username'
$identity = [System.Security.Principal.NTAccount]::new($account)
$sid = $identity.Translate([System.Security.Principal.SecurityIdentifier])
$output = & sc.exe sdshow $serviceName
if ($LASTEXITCODE -ne 0) { throw 'Unable to read service permissions.' }
$sddl = ($output | Where-Object { $_.Trim() -match '^[DOGS]:' }) -join ''
if ([string]::IsNullOrWhiteSpace($sddl)) { throw 'No service security descriptor returned.' }
$backup = Join-Path $env:USERPROFILE ('cs2-server-permissions-' + (Get-Date -Format 'yyyyMMdd-HHmmss') + '.sddl')
$sddl | Set-Content -LiteralPath $backup
$descriptor = [System.Security.AccessControl.RawSecurityDescriptor]::new($sddl)
$ace = [System.Security.AccessControl.CommonAce]::new(
   [System.Security.AccessControl.AceFlags]::None,
   [System.Security.AccessControl.AceQualifier]::AccessAllowed,
   0x2003D, $sid, $false, $null)
$descriptor.DiscretionaryAcl.InsertAce($descriptor.DiscretionaryAcl.Count, $ace)
$updated = $descriptor.GetSddlForm([System.Security.AccessControl.AccessControlSections]::All)
& sc.exe sdset $serviceName $updated
if ($LASTEXITCODE -ne 0) { throw 'Unable to grant service control permissions.' }
Write-Output "Original service permissions backed up to $backup"
```

Run Universal normally after setup. If an elevated instance was used earlier, fully exit it (including its tray icon) first. Verify Start/Stop/Restart from the unelevated app. Universal checks access to services named by standard lifecycle commands before attempting control and never grants itself access or requests elevation. Custom command scripts remain supported, but their permissions cannot be prechecked without executing them. To undo the grant, use Administrator PowerShell to pass the backed-up descriptor to `sc.exe sdset cs2-server`. Do not replace a service ACL with an unrelated example or grant control to Everyone.

### Steamworks Runtime For Windows Services

WinSW normally runs as LocalSystem, which cannot rely on your signed-in user's Steam installation. If CS2 exits with **Could not determine Steam client install directory**, check that `steamclient64.dll`, `tier0_s64.dll`, and `vstdlib_s64.dll` are beside `cs2.exe`. Steam app-ID environment variables alone do not supply these DLLs. Stop the service and any Direct-process instance before replacing runtime files.

Run this on the Windows server in Administrator PowerShell. Replace the install and SteamCMD paths; SteamCMD must already be installed. Download Valve's Steamworks SDK Redist app `1007`, not DLLs from third-party download sites:

```powershell
$ErrorActionPreference = 'Stop'
$installDir = 'D:\CS2Server'
$steamCmdPath = 'D:\CS2Server-steamcmd\steamcmd.exe'
$runtimeDir = Join-Path (Split-Path -Parent $steamCmdPath) 'steamworks-sdk-redist'
$destination = Join-Path $installDir 'game\bin\win64'
if (Get-Service -Name 'cs2-server' -ErrorAction SilentlyContinue) {
   Stop-Service -Name 'cs2-server' -ErrorAction Stop
}
& $steamCmdPath +force_install_dir $runtimeDir +login anonymous +app_update 1007 validate +quit
if ($LASTEXITCODE -ne 0) { throw 'SteamCMD runtime installation failed.' }
foreach ($file in @('steamclient64.dll', 'tier0_s64.dll', 'vstdlib_s64.dll')) {
   $source = Get-ChildItem -LiteralPath $runtimeDir -Recurse -File -Filter $file |
      Select-Object -First 1
   if (-not $source) { throw "Missing Steamworks runtime file: $file" }
   Copy-Item -LiteralPath $source.FullName -Destination (Join-Path $destination $file) -Force
}
```

Start the service after configuring its XML, then check both the service and CS2 process. **Running** briefly is not proof that CS2 stayed alive: check the latest output for the Steamworks error and confirm the server responds. WinSW's failure policy can otherwise repeatedly restart a crashing server. Its wrapper log includes launch arguments, so redact passwords, API keys, and tokens before sharing logs.

`<startmode>Automatic</startmode>` starts the registered service at Windows boot without Universal running. Installing the service does not immediately start it. Windows boot and Services commands do not run Universal's SteamCMD validation; normal Start in Universal validates when configured, while Restart intentionally skips validation.


### Finish Local Windows Setup

After the Local Windows profile is configured:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the Windows host firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**. For troubleshooting, **Start server without validation** starts a local Direct-process server without waiting for SteamCMD file validation.

If you selected **Install/Update server...** without add-ons, install the frameworks first:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.

Whether you selected **Install/Update server...** or **Install/Update Server + Add-ons**, the next step is to install the optional CSS plugins:

1. Right-click the running server and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the plugin window:
```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
2. Right-click the server and select **Restart server** so the frameworks and plugins load.

Once installation is complete, continue to the corresponding sections for configuration and usage details:

- [RCON Tool Companion and Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Install Frameworks And Plugins](#install-frameworks-and-plugins) for Metamod, CounterStrikeSharp, and optional CSS plugin details.

## Remote Windows Server

Use a **Remote Windows** profile when CS2 runs on another Windows computer.

### Remote Windows Requirements

- OpenSSH server access to the remote Windows computer.
- The SSH account must belong to the remote computer's local Administrators group for WMI process creation and firewall configuration.
- Optional SFTP settings for browsing and general file transfers. Blank values reuse the SSH host and credentials; port `0` reuses the SSH port. Valid SFTP ports range through `65535`.
- A Windows SteamCMD path and CS2 installation directory for managed updates.

Remote Windows lifecycle and plugin deployment do not require WinRM. Direct process startup and plugin transfers do not require a separate SFTP service.

Remote Windows profiles reject Unix paths before SteamCMD starts. When a profile is changed from Remote Linux to Remote Windows, stale auto-generated executable and working-directory defaults are replaced with Windows defaults. Existing overlapping or former global SteamCMD defaults are migrated automatically; an explicitly configured non-overlapping custom path remains unchanged.


### Create The Windows SSH User

Perform this setup on the remote Windows computer from an elevated PowerShell prompt, before configuring the application profile. Windows OpenSSH provides both SSH and SFTP; a separate SFTP server is not required. Use an existing suitable account instead if one is already available.

Create a dedicated local account such as `cs2tool` and initially add it to the standard Users group:

```powershell
$sshUser = 'cs2tool'
$sshPassword = Read-Host 'Password for cs2tool' -AsSecureString
New-LocalUser -Name $sshUser -Password $sshPassword -Description 'CS2 RCON Tool remote management'
Add-LocalGroupMember -Group 'Users' -Member $sshUser
```

For this application's Remote Windows Direct process workflow, also add the account to local Administrators. The app uses WMI to launch CS2 and configures Windows Firewall rules; this was required by the tested setup:

```powershell
Add-LocalGroupMember -Group 'Administrators' -Member $sshUser
```

Install and enable OpenSSH Server. The capability check avoids reinstalling it when it is already present:

```powershell
$openssh = Get-WindowsCapability -Online | Where-Object Name -Like 'OpenSSH.Server*'
if ($openssh.State -ne 'Installed') {
   Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
}
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd
```

Allow inbound SSH if Windows did not create its firewall rule:

```powershell
if (-not (Get-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -ErrorAction SilentlyContinue)) {
   New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
}
```

Open `C:\ProgramData\ssh\sshd_config` as an administrator and make sure password authentication is enabled:

```text
PasswordAuthentication yes
```

Restart SSH after changing the file and verify it is listening:

```powershell
Restart-Service sshd
Get-Service sshd
Get-NetTCPConnection -LocalPort 22 -State Listen
```

Use a simple writable CS2 path such as `C:\CS2Server`, then grant the SSH account Modify access:

```powershell
New-Item -ItemType Directory -Path 'C:\CS2Server' -Force
icacls 'C:\CS2Server' /grant "${sshUser}:(OI)(CI)M" /T
```

From the computer running the application, test both SSH and SFTP before configuring the profile:

```powershell
ssh cs2tool@<remote-windows-host>
sftp cs2tool@<remote-windows-host>
```

Use the account password, confirm SFTP can enter and write to `C:\CS2Server`, then exit both sessions. Use the same username and password in the application profile, and leave separate SFTP fields blank so the app reuses the SSH connection.

### Remote Windows Direct Process

Use this mode when the application should launch CS2 over SSH without installing a service wrapper.

For an install directory of `C:\cs2server`, the application derives:

```text
Executable:        C:\cs2server\game\bin\win64\cs2.exe
Working directory: C:\cs2server\game (But not required)
SteamCMD path:     C:\cs2server-steamcmd\steamcmd.exe
```

Configure the startup map, game type, game mode, additional arguments, and Stop command. The application supplies `-dedicated`, `-console`, `-ip 0.0.0.0`, port, hostname, map, game type, game mode, Startup CFG, RCON password, tickrate, LAN mode, and VAC state. `0.0.0.0` binds to all available network addresses; replace it with a specific address in the launch arguments when you need to bind directly to one interface. Optional Steam API and GSLT credentials are appended and redacted from output.

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

If the service already exists, do not delete its registration just to change the CS2 install folder. Keep its wrapper path and `<id>` unchanged, stop it, and update the XML target paths and arguments. If you must move the wrapper itself, uninstall it using the old wrapper before installing from the new location.

Create `C:\Services\cs2-server\cs2-server.xml` beside `cs2-server.exe`; the basenames must match. The required `<id>` is the Windows service ID used by the app's lifecycle commands, and `<executable>` is the CS2 binary. `<name>` is the display name in Services. Replace the RCON password after `+rcon_password` in `<arguments>` with the password configured in Universal. Remove `-authkey` and `+sv_setsteamaccount` if unused, or replace their Steam API key/GSLT placeholders with valid values. Replace the sample paths, hostname, and map; XML-escape `&`, `<`, and `>` in values.

```xml
<service>
   <id>cs2-server</id>
   <name>CS2 Dedicated Server</name>
   <description>Counter-Strike 2 Dedicated Server managed by CS2 RCON Tool</description>
   <executable>C:\cs2server\game\bin\win64\cs2.exe</executable>
   <arguments>-dedicated -console -usercon -port 27015 -tickrate 64 -secure -ip 0.0.0.0 +sv_lan 0 +map de_dust2 +game_type 0 +game_mode 1 +exec server.cfg +hostname "CS2 Server" +rcon_password "&lt;rcon-password&gt;" -authkey "&lt;steam-api-key&gt;" +sv_setsteamaccount "&lt;gslt&gt;"</arguments>
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

Prepare the [Steamworks runtime for Windows services](#steamworks-runtime-for-windows-services) on the target Windows computer, using its install and SteamCMD paths. Install the service only if its ID is not already registered:

```powershell
New-Item -ItemType Directory -Force -Path C:\cs2server\logs | Out-Null
Set-Location C:\Services\cs2-server
.\cs2-server.exe install
Get-Service -Name 'cs2-server'
Get-CimInstance Win32_Service -Filter "Name='cs2-server'" |
   Select-Object Name, DisplayName, State, PathName
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

If the service is missing from `services.msc`, refresh with F5 and look for **CS2 Dedicated Server** (display name), or query its ID `cs2-server` with `Get-Service`. An install error is not corrected by refreshing the list. For Error 1053, check that the XML parses, every element is closed, and the paths exist; inspect wrapper logs when available. For a crash loop, inspect the latest CS2 output rather than just the service status. Automatic boot startup runs independently of Universal and does not invoke SteamCMD validation.


### Finish Remote Windows Setup

After the Remote Windows profile and its Direct process or Service commands setup are complete:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. For Remote Windows install/update and add-on operations, Universal runs the configured stop command and verifies the target CS2 process is gone before changing core files, frameworks, or plugins. If shutdown cannot be confirmed, the operation cancels before file writes. The server remains stopped after the manual repair/update; start it explicitly when ready.
6. Confirm the remote Windows firewall allows the configured game port over UDP and RCON port over TCP.
7. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**.

If you selected **Install/Update server...** without add-ons, install the frameworks first:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.

Whether you selected **Install/Update server...** or **Install/Update Server + Add-ons**, the next step is to install the optional CSS plugins:

1. Right-click the running server and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the plugin window:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
2. Right-click the server and select **Restart server** so the frameworks and plugins load.

Once installation is complete, continue to the corresponding sections for configuration and usage details:

- [RCON Tool Companion and Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Install Frameworks And Plugins](#install-frameworks-and-plugins) for Metamod, CounterStrikeSharp, and optional CSS plugin details.


## Remote Linux Server

Use a **Remote Linux** profile to manage a CS2 server on another Linux computer from the Windows application.

### Remote Linux Requirements

- SSH host, port, username, and password.
- Optional SFTP details. Blank values reuse SSH details; port `0` reuses the SSH port. Valid ports range through `65535`.
- Remote SteamCMD path, CS2 install directory, App ID `730`, and Steam login, normally `anonymous`.
- An account that can run lifecycle commands and read/write the installation.

Remote Linux profiles reject Windows drive paths before SteamCMD starts. Switching from Remote Windows replaces stale auto-generated executable and working-directory defaults with Linux defaults.

### Remote Linux Hosting Panel

For a hosting provider that exposes a Pterodactyl-compatible panel but does not provide SSH, select **Hosting panel API** as the Remote Linux startup mode. Configure the panel URL, server identifier/UUID, and a client API key with power-control permission. Keep the SFTP address, username, password, and port configured for plugin deployment, browsing, and file operations. Start, Stop, and Restart use the panel API and wait for `offline` or `running`; SteamCMD validation and remote shell operations are unavailable. Debug Mode records panel requests, state polling, and failures without logging the API key.

Set **Framework management** to **Host-managed** when the provider preloader owns Metamod and CounterStrikeSharp. This disables framework installation and upgrade actions while keeping optional CSS plugin deployment over SFTP available. Use **Application-managed** only for a provider that does not manage the frameworks.


### Remote Linux Account And Permissions

These instructions apply to the Linux computer being managed, not to the Windows computer running the application.

Prepare the remote accounts before configuring the application profile. The **SSH control account** is the username entered in the profile; it runs lifecycle commands and performs SFTP browsing, file transfers, plugin operations, backups, and SteamCMD operations. The **service account** owns and runs CS2 when using Service commands with systemd. These can be the same account, or they can be separate accounts.

The simplest setup uses the Linux account that owns the CS2 installation for both SSH and SFTP. If a dedicated SSH account does not already exist, create it as an administrator on the remote host and grant it SSH access:

```bash
sudo adduser <ssh-user>
sudo passwd <ssh-user>
```

Do not use `root` as the application, SteamCMD, or CS2 account. The SSH control account must be able to traverse every parent directory and read and write the CS2 installation. If it differs from the service account, grant access through a shared group or filesystem ACL rather than making the installation world-writable.

If the systemd service account does not already exist, create it before installing or assigning ownership of the CS2 files. It may be the same account as `<ssh-user>`:

```bash
sudo adduser <service-user>
```

Verify access before saving the application profile. Replace the placeholders with the configured account and install directory:

```bash
namei -l "<install-dir>/game/csgo"
sudo -u <ssh-user> test -r "<install-dir>/game/csgo/gameinfo.gi"
sudo -u <ssh-user> touch "<install-dir>/game/csgo/.rcon-tool-write-test"
sudo -u <ssh-user> rm "<install-dir>/game/csgo/.rcon-tool-write-test"
```

Use a dedicated group or ACL rather than world-writable permissions:

```bash
sudo setfacl -m u:<ssh-user>:x "<parent-directory>"
sudo setfacl -R -m u:<ssh-user>:rwX "<install-dir>"
sudo setfacl -R -d -m u:<ssh-user>:rwX "<install-dir>"
```

The first command-line SSH connection may ask you to trust the host fingerprint. The application uses its own SSH library and does not depend on the command-line client's `known_hosts` entry.

### Remote Linux Direct Process

Use Direct process when the SSH account owns the installation and the application should control the executable without systemd. For `/home/cs2/cs2server`, blank fields derive:

```text
Executable:        /home/cs2/cs2server/game/bin/linuxsteamrt64/cs2
Working directory: /home/cs2/cs2server/game (But not required)
```

Set map, game type, game mode, and optional additional arguments. The application supplies `-dedicated`, `-console`, port, hostname, `+map`, `+game_type`, `+game_mode`, `+exec`, `-usercon`, RCON password, tickrate, LAN mode, and VAC state. The default `-ip 0.0.0.0` binds to all available network addresses and permits remote connections; replace `0.0.0.0` with a specific address when direct interface binding is required.

Use a path-specific Stop command, for example:

```bash
executable='/home/cs2/cs2server/game/bin/linuxsteamrt64/cs2'; for process in /proc/[0-9]*; do target=$(readlink "$process/exe" 2>/dev/null || true); if [ "$target" = "$executable" ]; then kill -TERM "${process##*/}"; fi; done
```

Start refuses to run during an app `730` SteamCMD update, verifies `game/csgo/gameinfo.gi`, supplies required native-library paths, launches detached from SSH, and monitors for 30 seconds. Output is captured in `/tmp/cs2-rcon-tool-startup-<port>.log`. Direct mode refuses to control an executable in the active `cs2-server` systemd cgroup and does not provide boot startup or crash restart.


### Remote Linux Service Commands With systemd

The application controls an existing service; it does not create privileged unit or sudoers files. Create `/usr/local/libexec/cs2-rcon-tool-start` after replacing all placeholders:

```bash
#!/usr/bin/env bash
export HOME="<service-home>"
export LD_LIBRARY_PATH="<install-dir>/game/bin/linuxsteamrt64:<install-dir>/game/csgo/bin/linuxsteamrt64:<install-dir>/.steam/sdk64:<install-dir>/steamcmd/linux64${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
cd "<install-dir>/game"
"<install-dir>/game/bin/linuxsteamrt64/cs2" \
   -dedicated -ip 0.0.0.0 -high -nobreakpad -nomemorystats -nojoy \
   +sv_hibernate_when_empty 0 +map de_dust2 +game_type 0 +game_mode 1 \
   +exec server.cfg -console -port 27015 -tickrate 64 -secure +sv_lan 0 \
   +hostname "CS2 Server" -usercon +rcon_password "<rcon-password>" &
child=$!
shutdown() { kill -TERM "$child" 2>/dev/null || true; wait "$child" 2>/dev/null || true; exit 0; }
trap shutdown TERM INT
wait "$child"
exit $?
```

Create `cs2-server.service`:

```ini
[Unit]
Description=Counter-Strike 2 Dedicated Server
After=network-online.target
Wants=network-online.target
StartLimitIntervalSec=300
StartLimitBurst=5

[Service]
Type=simple
User=<service-user>
Group=<service-group>
ExecStart=/usr/local/libexec/cs2-rcon-tool-start
Restart=on-failure
RestartSec=5
TimeoutStopSec=30
KillSignal=SIGTERM
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

Grant only the required lifecycle commands:

```sudoers
<ssh-user> ALL=(root) NOPASSWD: /usr/bin/systemctl start cs2-server, /usr/bin/systemctl stop cs2-server, /usr/bin/systemctl restart cs2-server, /usr/bin/systemctl status cs2-server, /usr/bin/systemctl is-active cs2-server
```

Install the files and enable the unit:

```bash
sudo install -d -o root -g root -m 755 /usr/local/libexec
sudo install -o root -g <service-group> -m 750 ./cs2-rcon-tool-start /usr/local/libexec/cs2-rcon-tool-start
sudo install -o root -g root -m 644 ./cs2-server.service /etc/systemd/system/cs2-server.service
sudo visudo -cf ./cs2-rcon-tool.sudoers
sudo install -o root -g root -m 440 ./cs2-rcon-tool.sudoers /etc/sudoers.d/cs2-rcon-tool
sudo systemctl daemon-reload
sudo systemctl enable cs2-server.service
```

Configure the profile:

```text
Startup mode: Service commands
Start:   sudo -n systemctl start cs2-server
Stop:    sudo -n systemctl stop cs2-server
Restart: sudo -n systemctl restart cs2-server
Remote install directory: <install-dir>
```

All CS2 arguments belong in the launcher. Start and Restart verify SteamCMD is idle, check the core file, and require the service-owned process to remain alive for 30 seconds. Stop verifies that it exits.


### Finish Remote Linux Setup

After the Remote Linux profile and its Direct process or Service commands setup are complete:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the remote Linux firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**.

If you selected **Install/Update server...** without add-ons, install the frameworks first:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.

Whether you selected **Install/Update server...** or **Install/Update Server + Add-ons**, the next step is to install the optional CSS plugins:

1. Right-click the running server and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the plugin window:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
2. Right-click the server and select **Restart server** so the frameworks and plugins load.

Once installation is complete, continue to the corresponding sections for configuration and usage details:

- [RCON Tool Companion and Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Install Frameworks And Plugins](#install-frameworks-and-plugins) for Metamod, CounterStrikeSharp, and optional CSS plugin details.

## SteamCMD Behavior

| Windows profile and mode | Start | Restart |
| --- | --- | --- |
| Local Windows, Direct process | Runs validation when SteamCMD and install-directory settings are configured | Does not run validation |
| Local Windows, Service commands | Runs SteamCMD validation when configured, then the configured Start command | Runs the configured Restart command without validation |
| Remote Windows, Direct process | Runs SteamCMD validation, then starts and monitors the process | Stops and starts without validation |
| Remote Windows, Service commands | Runs SteamCMD validation, then runs the service command and monitors its process | Runs Restart and monitors its process without validation |

Normal **Start server** validates before startup when SteamCMD is configured. Use **Validate server files** to validate without starting the server, or **Install/Update Server** to update, repair, and validate the CS2 installation.

## Windows Plugin Deployment

Remote Windows transfers normalized plugin packages as acknowledged 32 KB parts over SSH to stay below the OpenSSH channel window. A stalled part is retried over a fresh connection, the assembled byte count is verified, transfer progress is reported, and temporary parts and partial archives are removed after failure. PowerShell extracts the package, preserves existing configuration files, and atomically replaces plugin binaries without requiring SCP or SFTP.

Remote Windows add-on downloads suppress PowerShell progress serialization so CLIXML telemetry cannot flood **Console Commands** or turn a successful command into a false failure. Installed payload checks use one constant-size PowerShell operation that reads the ownership manifest on the server instead of growing with its file count.

For local plugin batches, the application detects a running CS2 process, stops it once before replacing plugin and shared API assemblies, then restarts it without SteamCMD. This avoids Windows file-lock failures on loaded DLLs. Stop a Remote Windows service before upgrading plugins that replace loaded shared assemblies, then start it after the batch.

## Using The Application

Add server provisioning can reuse existing installations. Before changing files, it stops a local service through the configured Stop command or stops a detected local Direct-process target; remote profiles use their verified Stop path. If stopping fails or local `gameinfo.gi` remains locked, provisioning aborts before installation changes. Keep other applications from starting the target during provisioning. Plugin stages leave it stopped; start-after-install starts that profile only after all selected stages succeed. If unchecked, or if provisioning fails, the target stays stopped. The app account needs permission to execute the configured lifecycle commands.

### Connect And Refresh

1. Click **Add server** in the top toolbar and choose the location-specific startup method.
2. Enter a profile name, host/IP, RCON port, and RCON password. Enable **LAN server** when appropriate; LAN hides the Steam credential fields. With LAN off, Steam API key and GSLT fields are shown for app-managed launches but remain optional; Service commands use credentials from the service configuration.
3. For remote profiles, enter SSH credentials before browsing. For install actions, choose the CS2 install folder in the wizard; SteamCMD is installed or discovered automatically. Other advanced lifecycle fields remain under **Edit server**.
4. Configure Workshop, database, and ChatRelay integration values when used.
5. Save the profile, select it in the top list, and click **Refresh server list**.

At the end of the Add server wizard, choose to finish setup or install/update the server, optionally with Metamod and CounterStrikeSharp, and optionally start it after successful installation. Hosting panel API profiles expose framework ownership; provider-managed CS2 core updates remain in the panel.

Use **Help > Sample server profile** for a complete read-only example of the available fields. It uses placeholder values and is not added to the server list.

RCON uses TCP. Opening only the UDP game port is not sufficient. The top list displays hostname, address, port, map, players, ping, location, and country. **Reload server list** reads profiles without contacting servers; **Refresh server list** queries live A2S information.

Server-table column sorting is retained during refresh and restored after application restart. Saved profile names stay independent of the running CS2 hostname. Windows RCON actions use the selected profile's configured address and port; two profiles pointing to the same running instance still share its runtime settings.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete installations, plugins, backups, or managed-server files.

### Server Actions And Joining

Server Actions controls hostname, bots, teams, passwords, friendly fire, cheats, pause, maps, maximum rounds, timed restarts, and kick/ban/slay/slap operations. These are runtime commands and never edit server CFG files. Values in a Startup CFG can replace runtime changes when a map executes it.

The map selector lists playable maps and configured Workshop maps.

Select a server and click **Join server**. Windows uses the registered Steam URI handler. If Steam opens CS2 without connecting, enable the developer console, press `~`, and paste the copied `connect IP:port` command.

### Settings

Open **Settings > Settings**. The **General** tab contains GeoLite and flag URLs, Slap damage from `0` to `99`, automatic 15-second refresh, noisy-output filtering, startup update checks, **Minimize to tray**, and **Update GeoLite + flags**. The **Backups** tab provides separate server-file and database backup destinations. **Help > Check for Updates** only notifies and links to GitHub; it never downloads or installs the app.

Server-specific lifecycle, SteamCMD, map, Workshop, RCON, player-password, database, ChatRelay, Steam authentication, and backup credentials remain under **Servers > Manage servers > Edit server**. Protected values are encrypted before storage.

### Console, Logs, And Debug

Console Commands accepts free-form RCON commands and categorized suggestions. Each server has separate bounded history, and process, SteamCMD, SSH, and RCON output remains associated with its originating server. Application Log reports normal operations and failures; Debug provides detailed diagnostics. Passwords, credentials, API keys, and tokens are redacted even in Debug Mode and raw output.

When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the application or **Exit program** to close anyway.

### Scheduled Tasks

Task types include RCON commands, framework/plugin maintenance, server backups, database backups, and GeoLite/flag updates. Triggers include Day, Hour, and Startup. The built-in scheduler runs while the application is open and monitoring is active. Native Windows tasks use Windows Task Scheduler and invoke the headless task runner, so the GUI does not need to remain open.

**Fun Stuff rotation** tasks start one or more Fun Stuff modes in order. Each step has a mode and a duration in minutes; after the last step the rotation can stay on the last mode, roll it back, or repeat from the first step. A single step with **Stay on last mode** simply switches the server into that mode at the trigger time. Modes are started by RCON Tool Companion 1.0.2 using its `RCT.json` settings (also editable in game through `!fun`). Rotations run only in the built-in scheduler while the app is open; editing or deleting the task, or stopping the scheduler, ends a running rotation and leaves the current mode active.

## RCON Tool Companion And Fun Stuff

The GUI allows one instance per user/settings directory. Relaunching the shortcut restores the existing window, including from the tray; it does not create another GUI. Headless task/asset commands bypass this guard. Before switching from an older build, use the tray's **Exit** to stop it once, then start the updated app. IPC/startup/relaunch tests passed; actual desktop focus/restoration still needs testing.

With **Minimize to tray** enabled, hiding the window shows an eight-second **Still running in the tray** reminder with Show/Exit actions and a clearer tooltip. It does not take keyboard focus or add a taskbar item; unsupported-tray fallback remains unchanged. Console feedback now identifies expected-silent operations such as `css_plugins reload 9`, distinguishes failed sends, leaves unclassified empty replies unconfirmed, and preserves actual reply/error text. Headless reminder/position and formatter checks passed; desktop tray interactions still need testing.

Long-running monitoring updates player health/status on existing rows rather than rebuilding the player table. Read-only output panes keep bounded text without undo history. These reduce allocation churn; an overnight memory retest of the updated build is still pending.

All Fun Stuff modes require the current [RCON Tool Companion 1.0.2 build](https://github.com/ShAgGy2035/RconCompanionTool), built against CounterStrikeSharp API 1.0.376. Validate older or KHS fork runtimes against this Companion build before deployment. Companion includes its own WASD `!fun` menu (**W/S** move, **E** select, **A** back, **R** exit), so no separate menu plugin is required.

Modes include No Bots, AWP/Sniper Wars, Bhop, Deathmatch, Grenade Wars, Molotov/Incendiary Wars, Headshot Only, Knife Arena, Pistols Only, ScoutzKnives, Super Hero, Surf, and Zeus Wars. Super Hero keeps normal weapons and buying and gives every player one enabled power each spawn: Speedster, Moon Jumper, Tank, Ghost, Vampire, Regenerator, Juggernaut, Second Wind, Ammo Recycler, Berserker, Medic, Blink, Double Jump, or Gunslinger. Universal exposes all fourteen toggles and strength controls in its Options panel. The in-game `!fun` mode uses Companion defaults from `RCT.json`. Universal also has a standalone **Bot Control** option, mirrored under `!fun` > Bot control, for bot enablement, quota, quota mode, and team. The `!fun` Bot Control panel stages edits until **Apply now**, which saves preferences to `RCT.json` and applies them to the server. **Bot team** can put every bot on CT or T for lopsided training such as 20 bots against one player. Returning to **Any** restores the original team cvars without kicking bots. No Bots is the first `!fun` option; it only removes bots, and rollback restores the previous bot quota. Administrators with `@css/fun` can use `!fun` or `/fun` in game without the RCON password. Selecting a mode other than No Bots in `!fun` opens a submenu to start it and set its bot enablement, quota, and quota mode. By default, Companion rolls back the active mode when the last human player leaves. Upgrading Companion does not add new options to an existing `RCT.json`; missing options use their defaults. See the Companion README's **Upgrading an existing `RCT.json`** section to add them manually or regenerate the file.

**One in the Chamber** uses the native Deathmatch ruleset with individual FFA scoring, random spawns, a stock Deagle, knife, and exactly one starting bullet. Every non-self kill earns another bullet; missing requires a knife kill to recover ammo. Default Deagle and knife hits are lethal; optional **Deagle headshots only** suppresses Deagle body damage but keeps knife kills lethal. Companion controls respawning with a default one-second delay and the usual bot options. It is available in Universal, `!fun`, and scheduled rotations. Chamber owns its rules and reload handling independently of Deathmatch: entry and rollback each reload the current map, and its guarded five-second map reapply does not restart the round. Buying and C4 objectives are disabled; rollback restores captured settings after map configs load. CT/T assignments may remain internally, but team Deathmatch scoring is disabled. Requires refreshed Companion 1.0.2 (`one-in-the-chamber`, `chamber-native-ffa`). Roll back an active older Chamber session before upgrading. Focused rule, reload-order, restoration, and isolation checks passed; the native HUD/scoring and gameplay still require a live retest.

Active modes are intended to survive map changes. Universal's cvar reapply requires the app to stay open; `!fun` and scheduled modes use Companion's saved-command reapply. Updated Companion 1.0.2 (`mode-map-reapply`) explicitly resets Super Hero's map-specific power state and reassigns powers about five seconds after map start. Existing Deathmatch, No Bots, and Chamber handlers retain their dedicated behavior. The default empty-server rollback still intentionally ends modes when the last human leaves; recovery does not revive a rolled-back mode. Live map-change validation is pending.

Super Hero shows the assigned power name and description in a center-screen popup for five seconds by default, with chat retained as a fallback. **Power popup (seconds)** sets the duration from 0 to 30 (`0` disables); `!fun` uses `SuperHero.PowerNoticeSeconds` in `RCT.json`. Spectators see the power of the living player or bot they watch, and switching targets starts a fresh popup. The notice is informational: it does not freeze movement or replace the `!fun` menu, which pauses its timer. Requires updated Companion 1.0.2 (`superhero-notice`); live HUD/spectator testing is pending.

Updated Companion's `empty-server-grace` keeps the active mode for five minutes after the last human leaves. `EmptyServerRollbackDelaySeconds` in `RCT.json` defaults to `300` (`0`-`3600` allowed). A human returning cancels the pending rollback; the deadline survives map changes and repeated empty checks do not extend it. The roster and transition state are checked again at expiry. Explicit **Rollback** still takes effect immediately. Status reports **Empty-server grace**, **Map transition**, and **Last mode end**. Live transition behavior still needs verification.

Super Hero notices now maintain cached HTML at the same per-tick cadence as `!fun`, with the configured server-side clock controlling when to clear it. This replaces the single-event revision, which was reported to disappear after about one second. Spectator switching and activation instructions such as Blink's Use key are retained; menu pause/resume preserves remaining time. Five-second upkeep checks passed, but visible duration and pulsing still need an in-game retest.

**Fun Stuff > Empty-server rollback (server-wide)** now exposes the automatic rollback toggle and grace period in minutes (default five, range 0-60). The panel reads the selected server's current RCT policy; **Apply settings** saves only that policy in the server's `RCT.json`, without starting or replacing a mode. It applies to all modes, including `!fun` and scheduled rotations. Changes are not sent merely by opening the tab. Requires updated Companion 1.0.2 (`empty-server-settings`); live UI/RCON saving still needs testing.

The refreshed Companion corrects settings saves to `configs/plugins/RCT/RCT.json`, rather than the incorrectly assembly-derived `RconToolCompanion/RconToolCompanion.json` path. Its ZIP now begins at `addons/`; manual extraction belongs in the server's `game/csgo/` directory. Universal's plugin installer recognizes that layout. Do not loosen file permissions to work around the old path mismatch; hosted save access still requires a live retest.

Super Hero spreads enabled powers among living teammates before repeating one. With at least five enabled, a five-player team gets five different powers; larger teams or fewer enabled powers use balanced repeats. CT and T are assigned independently, including bots. Respawns prefer a different power when an equally suitable choice exists.

The eight additional powers provide damage reduction, one-hit survival, magazine refill on enemy kills, low-health bonuses, teammate healing, safe Blink with the Use key, one extra airborne jump, and faster reload completion. They default to enabled, with configurable strengths. Expanded powers require the updated Companion 1.0.2 (`superhero-expanded`) and the full CounterStrikeSharp 1.0.376 runtime, not only its API DLL. Live gameplay still needs testing; see the Companion guide for defaults and input details.

Moon Jumper combines a high launch velocity with reduced gravity. Defaults are **Jump velocity** `1200` units/s (range `300`-`1500`) and **Gravity multiplier** `0.275` (`1` is normal gravity; `0.275` approximates ScoutzKnives' default `sv_gravity 220` on a standard 800-gravity server). Hold Jump as you take off; the boost applies once the pawn is airborne. Enable only Moon Jumper when testing because each spawn otherwise receives a random enabled power. Existing `RCT.json` values are preserved; update `SuperHero.MoonJumperJumpVelocity` manually if `!fun` should use the new default.

Every mode's bot quota mode defaults to `fill`; `normal` and `match` remain selectable. A blank enabled quota preserves the existing value; disabling bots removes them and holds the quota at zero until rollback or a mode switch restores the captured quota and mode. Explicitly saved quota modes are preserved.

The application captures affected cvars before applying a mode. Switching restores the previous mode first; **None (rollback)** restores exact captured values without editing or executing a CFG. Each mode activates an isolated Companion profile, and a shared server-side lock rejects overlapping apply and rollback attempts. Companion maintains player-aware inventories across spawns and bot takeovers while preserving the Terrorist bomb carrier's C4. Managed-weapon modes clean dropped and map-placed weapons; Headshot Only deliberately leaves weapons and buying unchanged.

ScoutzKnives manages SSG 08 plus team knife and exposes air acceleration, gravity, fall damage, friction, and bunnyhop controls. AWP/Sniper Wars manages AWP-only inventories. Pistols, Knife, Zeus, HE-grenade, and fire-grenade modes retain dedicated loadouts. Surf exposes acceleration, air acceleration, gravity, stamina, round-time, freeze-time, cheats, and bunnyhop controls. Values are also available in standalone `RCT.json`.

Deathmatch stages duration, respawn, and random-spawn behavior before map initialization. Leaving Deathmatch reloads the current map because CS2 only fully exits its built-in Deathmatch mode on a map load; other rollbacks and switches restart the round. Companion stores one authoritative pre-mode snapshot under `configs/plugins/RCT`, allowing Universal or `!fun` to roll back a mode applied by either interface.

## PlayerPunishments Setup

The app's Slay and Slap controls send `css_slay #userid` and `css_slap #userid damage`. Install [PlayerPunishments 1.0.0 or newer](https://github.com/ShAgGy2035/PlayerPunishments) on each managed CS2 server when its existing administration package does not provide those commands. Configure Slap damage from `0` to `99` under **Settings > General**.

PlayerPunishments is a CounterStrikeSharp server plugin. Install it in the managed Windows CS2 server's CounterStrikeSharp plugin directory, not in the desktop application's folder. Alive and Health display **Unknown** when the required server-side player-information command is unavailable.

## ChatRelay

ChatRelay is a separate [CounterStrikeSharp plugin](https://github.com/ShAgGy2035/ChatRelay); its binaries are not bundled with this application.

1. Install and configure ChatRelay on the CS2 server using its repository instructions.
2. In the Chat tab, select the receiving network adapter and UDP port; the default is `9090`.
3. Set a unique high-entropy token under **Edit server > Integrations**.
4. Use **Copy ChatRelay target JSON** to create the matching `TargetIp`, `TargetPort`, and `SharedToken` entry.
5. Use the local authenticated **Test** action and send admin chat through RCON as needed.

The listener accepts authenticated JSON only, limits datagrams to 8 KiB, and drops rejected packets. UDP does not encrypt traffic or prove the sender's network address. Restrict firewall access to the game server or trusted LAN, or use a VPN.

## Frameworks And Plugins

Install Metamod before CounterStrikeSharp. Interactive installation shows the latest 20 compatible releases and accepts a selected release or direct HTTP(S) ZIP, TAR.GZ, or TGZ URL. Scheduled and headless maintenance use the newest compatible releases because they cannot display selection dialogs.

### Install Frameworks And Plugins

Install Metamod before CounterStrikeSharp. Use the application menus in this order:

1. Select the configured server in the top server list.
2. Right-click the server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL when prompted, then wait for installation to finish.
3. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL when prompted, then wait for installation to finish.
4. Right-click the server again and select **Install/Upgrade CSS Plugins...** to install optional CounterStrikeSharp plugins. Paste the plugin's GitHub or supported GitLab repository/release URL, choose the matching release asset if prompted, and wait for deployment to finish.
5. Restart the server after installing or upgrading frameworks or plugins so the server loads them.

Under **Settings > Metamod Settings**, list plugins and run info, pause, unpause, retry, load, unload, or force-unload operations. Under **Settings > CStrikeSharp Settings**, reload admins and list, reload, or unload plugins. **Settings > Test Metamod/CSS command availability** probes safe command forms with nonexistent plugin IDs.

Optional plugins can come from GitHub, a public GitLab.com project including nested groups, or a supported local archive or library. Assets are filtered by server OS. When multiple packages match, the application asks which asset to install and remembers a version-independent filename preference. Installed files are tracked in `game/csgo/.cs2-rcon-tool-plugins.json`; keep this manifest with the server so uninstall removes only owned files and preserves shared configuration and framework files.

Use the **Manage Metamod plugins** window only for packages containing an `addons/metamod` payload. Use the **Manage CounterStrikeSharp plugins** window for CounterStrikeSharp plugin packages. The application rejects packages submitted through the wrong manager before deployment.

Remote updates replace native binaries atomically rather than truncating loaded files. A running server keeps using the previous binary until restart, preventing memory-mapped library crashes. Windows-specific transfer and file-lock behavior is detailed under [Windows Plugin Deployment](#windows-plugin-deployment).

## Backups, Data, And Security

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking rather than stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` with the selected server's integration settings.
- **Restore server backup...** accepts ZIP, TAR.GZ, or TGZ, validates it, stops CS2, restores supported content, and restarts it.
- **Install/update + restore server backup...** updates core files before restoring.
- **Restore database backup...** imports SQL with the configured MySQL client.
- **Validate server files** runs SteamCMD with `validate`.

Writable data is stored under `%APPDATA%\CS2RconTool`, not beside the executable. It includes `servers.json`, `tasks.json`, `steam_settings.json`, `fun_stuff_state.json`, `app_state.json`, `credential.key`, GeoLite data, and flags. Startup validates the JSON files and moves invalid files into timestamped `migration-backups` folders. Saved secrets use AES-256-GCM. Protect and back up `credential.key` with the JSON files. Release archives exclude runtime settings, keys, caches, PDBs, ChatRelay binaries, and Companion binaries.

For migration, open the new application once and close it, back up the generated data directory, then copy the old JSON files and matching `credential.key`. Re-enter and save credentials that cannot be decrypted. Imported `Local Linux` profiles must be changed to `Local Windows` with Windows executable, SteamCMD, install, working-directory, and launch paths before they can be saved. Remote profiles remain usable with valid target paths and credentials.

## Troubleshooting

### RCON commands fail

- Verify the host/IP and RCON TCP port.
- Confirm firewalls permit the game UDP port and RCON TCP port.
- Re-enter SSH or RCON credentials after migration when decryption fails.

### A Windows server in a bridged VirtualBox VM is unreachable

- Use the guest's LAN address rather than the host address or `127.0.0.1`.
- If the guest hostname is used, add its LAN address and hostname to `C:\Windows\System32\drivers\etc\hosts` in the Windows guest. Edit the file as Administrator.
- If VirtualBox created a Host-Only adapter such as `Ethernet 2` and it interferes with routing, disable it in Windows Network Connections while keeping the physical Ethernet/Wi-Fi adapter enabled.
- For Direct process mode, keep `-ip 0.0.0.0` in the launch arguments for remote access, or replace it with the guest's specific LAN address. For Service commands mode, put the chosen `-ip` value in the WinSW XML.
- Confirm the Windows guest firewall permits the configured game port over UDP and RCON port over TCP.

### Local server exits immediately

- Check Application Log and Debug for the exit code and stderr.
- Verify the executable, launch arguments, profile OS, and working directory. Working directory can be left blank.
- Direct-process startup is monitored for 30 seconds and reports captured output on failure.
- Stop operations verify that the configured executable exited.
- Do not run Direct process and Service commands modes against the same executable and port.

### Remote lifecycle, browsing, or installation fails

- `Connection refused` means the configured SSH host or port did not accept a connection; authentication errors mean the SSH username or password was rejected.
- Direct-process startup requires the remote SSH account to be a local Administrator.
- Remote Windows Install/Update provisions Steamworks SDK Redist app `1007` and repairs required 64-bit Steam runtime DLLs beside `cs2.exe`.
- Remote Windows SteamCMD retries one incomplete update pass for `0x602`, `0x202`, or exit code `8`.
- For Remote Linux permission problems, use the checks and repair steps in the Remote Linux sections above.

### Metamod or CounterStrikeSharp commands are unknown

- If `meta` is unknown, Metamod is not loaded; CSS commands depend on that loader. For local Service commands profiles, Stop and then Start in Universal restores an installed Metamod search-path entry after validation before launching WinSW. Merely using Restart invokes the saved service command and does not perform this repair.
- For Remote Windows profiles, Direct and Service **Start** check for an installed Metamod loader and restore a missing `gameinfo.gi` search-path entry before launch. The SSH account must be able to write `gameinfo.gi`; startup fails with an error if repair cannot be completed. A Service **Restart** runs the saved restart command directly, so use Stop then Start to run the app's repair check.
- Install or update Metamod first, then install or update CounterStrikeSharp.
- Restart CS2 after installing or updating either framework.
- Use **Settings > Test Metamod/CSS command availability** to check whether the framework commands are loaded.
- If the commands are still unavailable, stop CS2, run **Install/Update Server** to repair and validate the core files, reinstall Metamod and CounterStrikeSharp, and start CS2 again.

### Join Server opens CS2 but does not connect

- Open the CS2 developer console with `~` and paste the copied `connect IP:port` command.

### Chat does not appear

- Verify ChatRelay is running, match the destination IP, adapter, UDP port, and token, check firewall rules, and use the Chat tab's **Test** action.

### Scheduled tasks do not run

- Confirm the task is active and valid.
- Start monitoring for built-in tasks and inspect Windows Task Scheduler for native tasks.

### Country or flag data is missing

- Run **Settings > General > Update GeoLite + flags**.
- Confirm the configured URLs are direct downloads and the per-user data directory is writable.

## About

Open **Help > About** to view version `1.0.5`, developer information, and the detected application platform.

## Application Screenshots

| Server management | Integrations |
| --- | --- |
| ![Manage servers](docs/images/manage-servers.png) | ![Edit server integrations](docs/images/edit-server-integrations.png) |
| Fun Stuff | Console commands |
| ![Fun Stuff modes](docs/images/fun-stuff-modes.png) | ![Console command suggestions](docs/images/console-commands.png) |
| Scheduled tasks | Add scheduled task |
| ![Scheduled tasks](docs/images/scheduled-tasks.png) | ![Add scheduled task](docs/images/add-scheduled-task.png) |
| Chat | Debug output |
| ![Chat](docs/images/chat-tab.png) | ![Debug output](docs/images/debug-tab.png) |
| General settings | Backup settings |
| ![General settings](docs/images/settings-general.png) | ![Backup settings](docs/images/settings-backups.png) |
| Metamod controls | CounterStrikeSharp controls |
| ![Metamod controls](docs/images/settings-metamod-menu.png) | ![CounterStrikeSharp controls](docs/images/settings-counterstrikesharp-menu.png) |
