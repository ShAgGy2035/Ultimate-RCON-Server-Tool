# Ultimate CS2 RCON and Server Management Tool - Windows Guide

Version: 1.0.4

This self-contained guide covers the Windows x64 application, Local Windows servers, and Remote Windows or Remote Linux servers. [README.md](README.md) provides the project-wide GitHub overview.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## Table of Content
- **Windows application**
  - [Install and launch](#install-and-launch)
  - [Update the application](#update-the-application)
  - [Configuration and data](#configuration-and-data)
- **Local Windows server**
  - [Choose a lifecycle mode](#choose-a-local-lifecycle-mode)
  - [Direct process](#local-windows-direct-process)
  - [Service commands](#local-windows-service-commands)
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

1. Extract `cs2-rcon-tool-universal-v1.0.4-windows-x64.zip` to a folder you control.
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


### Finish Local Windows Setup

After the Local Windows profile is configured:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the Windows host firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**.

If you selected **Install/Update server...** without add-ons, complete the framework setup now:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
3. Right-click the server again and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the **Install/Upgrade CSS Plugins...** window:
```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
4. Restart the server so the frameworks and plugins load.

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


### Finish Remote Windows Setup

After the Remote Windows profile and its Direct process or Service commands setup are complete:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the remote Windows firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**.

If you selected **Install/Update server...** without add-ons, complete the framework setup now:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
3. Right-click the server again and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the **Install/Upgrade CSS Plugins...** window:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
4. Restart the server so the frameworks and plugins load.

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
Working directory: /home/cs2/cs2server/game
```

Set map, game type, game mode, and optional additional arguments. The application supplies `-dedicated`, `-console`, port, hostname, `+map`, `+game_type`, `+game_mode`, `+exec`, `-usercon`, RCON password, tickrate, LAN mode, and VAC state. The default `+ip 0.0.0.0` permits remote connections.

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
   -dedicated +ip 0.0.0.0 -high -nobreakpad -nomemorystats -nojoy \
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

If you selected **Install/Update server...** without add-ons, complete the framework setup now:

1. Right-click the running server and select **Install/Upgrade Metamod**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
2. Right-click the server again and select **Install/Upgrade CSS**. Choose a compatible release or paste the direct package URL, then wait for installation to finish.
3. Right-click the server again and select **Install/Upgrade CSS Plugins...**. Paste each of these URLs into the **Install/Upgrade CSS Plugins...** window:

```text
https://github.com/ShAgGy2035/RconCompanionTool
https://github.com/ShAgGy2035/ChatRelay
https://github.com/ShAgGy2035/PlayerPunishments
```
4. Restart the server so the frameworks and plugins load.

Once installation is complete, continue to the corresponding sections for configuration and usage details:

- [RCON Tool Companion and Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Install Frameworks And Plugins](#install-frameworks-and-plugins) for Metamod, CounterStrikeSharp, and optional CSS plugin details.

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

Remote Windows add-on downloads suppress PowerShell progress serialization so CLIXML telemetry cannot flood **Console Commands** or turn a successful command into a false failure. Installed payload checks use one constant-size PowerShell operation that reads the ownership manifest on the server instead of growing with its file count.

For local plugin batches, the application detects a running CS2 process, stops it once before replacing plugin and shared API assemblies, then restarts it without SteamCMD. This avoids Windows file-lock failures on loaded DLLs. Stop a Remote Windows service before upgrading plugins that replace loaded shared assemblies, then start it after the batch.

## Using The Application

### Connect And Refresh

1. Open **Servers > Manage servers**.
2. Add a server and enter its name, host/IP, game/RCON port, RCON password, and location.
3. Complete lifecycle and SteamCMD fields when the application will manage those operations.
4. Configure Workshop, database, and ChatRelay integration values when used.
5. Save the profile, select it in the top list, and click **Refresh server list**.

RCON uses TCP. Opening only the UDP game port is not sufficient. The top list displays hostname, address, port, map, players, ping, location, and country. **Reload server list** reads profiles without contacting servers; **Refresh server list** queries live A2S information.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete installations, plugins, backups, or managed-server files.

### Server Actions And Joining

Server Actions controls hostname, bots, teams, passwords, friendly fire, cheats, pause, maps, maximum rounds, timed restarts, and kick/ban/slay/slap operations. These are runtime commands and never edit server CFG files. Values in a Startup CFG can replace runtime changes when a map executes it.

Select a server and click **Join server**. Windows uses the registered Steam URI handler. If Steam opens CS2 without connecting, enable the developer console, press `~`, and paste the copied `connect IP:port` command.

### Settings

Open **Settings > Settings**. The **General** tab contains GeoLite and flag URLs, Slap damage from `0` to `99`, automatic 15-second refresh, noisy-output filtering, and **Update GeoLite + flags**. The **Backups** tab provides separate server-file and database backup destinations.

Server-specific lifecycle, SteamCMD, map, Workshop, RCON, player-password, database, ChatRelay, Steam authentication, and backup credentials remain under **Servers > Manage servers > Edit server**. Protected values are encrypted before storage.

### Console, Logs, And Debug

Console Commands accepts free-form RCON commands and categorized suggestions. Each server has separate bounded history, and process, SteamCMD, SSH, and RCON output remains associated with its originating server. Application Log reports normal operations and failures; Debug provides detailed diagnostics. Passwords, credentials, API keys, and tokens are redacted even in Debug Mode and raw output.

When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the application or **Exit program** to close anyway.

### Scheduled Tasks

Task types include RCON commands, framework/plugin maintenance, server backups, database backups, and GeoLite/flag updates. Triggers include Day, Hour, and Startup. The built-in scheduler runs while the application is open and monitoring is active. Native Windows tasks use Windows Task Scheduler and invoke the headless task runner, so the GUI does not need to remain open.

## RCON Tool Companion And Fun Stuff

All Fun Stuff modes require the current [RCON Tool Companion 1.0.1 build](https://github.com/ShAgGy2035/RconCompanionTool). The plugin targets CounterStrikeSharp API 1.0.374 and supports `v1.0.374-khs-khook`. Install the complete MenuManagerAPI 1.0.3 release first; RCT requires its `MenuManagerAPI.Shared.dll` contract during registration and its `menu:api` capability for WASD navigation. Without it, CounterStrikeSharp reports RCT as unregistered and its commands are unavailable.

Modes include AWP/Sniper Wars, Bhop, Deathmatch, Grenade Wars, Molotov/Incendiary Wars, Headshot Only, Knife Arena, Pistols Only, ScoutzKnives, Surf, and Zeus Wars. Administrators with `@css/fun` can use `!fun` or `/fun` in game without the RCON password.

Every mode includes **Enable bots**, an optional quota from 1 to 64, and `normal`, `fill`, or `match` quota modes. A blank enabled quota preserves the existing value; disabling bots removes them and holds the quota at zero until rollback or a mode switch restores the captured quota and mode.

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

Remote updates replace native binaries atomically rather than truncating loaded files. A running server keeps using the previous binary until restart, preventing memory-mapped library crashes. Windows-specific transfer and file-lock behavior is detailed under [Windows Plugin Deployment](#windows-plugin-deployment).

## Backups, Data, And Security

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking rather than stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` with the selected server's integration settings.
- **Restore server backup...** accepts ZIP, TAR.GZ, or TGZ, validates it, stops CS2, restores supported content, and restarts it.
- **Install/update + restore server backup...** updates core files before restoring.
- **Restore database backup...** imports SQL with the configured MySQL client.
- **Validate server files** runs SteamCMD with `validate`.

Writable data is stored under `%APPDATA%\CS2RconTool`, not beside the executable. It includes `servers.json`, `tasks.json`, `steam_settings.json`, `fun_stuff_state.json`, `credential.key`, GeoLite data, and flags. Saved secrets use AES-256-GCM. Protect and back up `credential.key` with the JSON files. Release archives exclude runtime settings, keys, caches, PDBs, ChatRelay binaries, and Companion binaries.

For migration, open the new application once and close it, back up the generated data directory, then copy the old JSON files and matching `credential.key`. Re-enter and save credentials that cannot be decrypted. Imported `Local Linux` profiles must be changed to `Local Windows` with Windows executable, SteamCMD, install, working-directory, and launch paths before they can be saved. Remote profiles remain usable with valid target paths and credentials.

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
- If Chat does not appear, verify ChatRelay is running, match the destination IP, adapter, UDP port, and token, check firewall rules, and use the Chat tab's **Test** action.
- If country or flag data is missing, run **Settings > General > Update GeoLite + flags**, confirm the configured URLs are direct downloads, and verify the per-user data directory is writable.
- If a scheduled task does not run, confirm it is active and valid. Start monitoring for built-in tasks; inspect Windows Task Scheduler for native tasks.
- For Remote Linux permission or Metamod problems, use the checks and repair steps in the Remote Linux sections above.
- Do not run Direct process and Service commands modes against the same executable and port.

## About

Open **Help > About** to view version `1.0.4`, developer information, and the detected application platform.

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
