# Ultimate CS2 RCON and Server Management Tool - Linux Guide

Version: 1.0.5

This self-contained guide covers the Linux x64 application, Local Linux servers, and Remote Linux or Remote Windows servers. [README.md](README.md) provides the project-wide GitHub overview.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## Table of Content
- **Linux application**
  - [Install and launch](#install-and-launch)
  - [Update the application](#update-the-application)
  - [Configuration and data](#configuration-and-data)
- **Local Linux server**
  - [Quick start](#quick-start)
  - [Direct process](#local-linux-direct-process)
  - [Service commands](#local-linux-service-commands)
   - [Finish Local Linux setup](#finish-local-linux-setup)
- **Remote Linux server**
  - [Requirements](#remote-linux-requirements)
   - [Account and permissions](#remote-linux-account-and-permissions)
   - [Remote Linux Hosting Panel](#remote-linux-hosting-panel)
  - [Direct process](#remote-linux-direct-process)
  - [Service commands with systemd](#linux-service-commands-with-systemd)
   - [Finish Remote Linux setup](#finish-remote-linux-setup)
- **Remote Windows server**
   - [Requirements](#remote-windows-requirements)
   - [Create the Windows SSH user](#create-the-windows-ssh-user)
   - [Direct process](#remote-windows-direct-process)
   - [Service commands with WinSW](#remote-windows-service-commands-with-winsw)
   - [Finish Remote Windows setup](#finish-remote-windows-setup)
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

Extract the Linux package and open its directory:

```bash
tar -xzf cs2-rcon-tool-universal-v1.0.5-linux-x64.tar.gz
cd cs2-rcon-tool-universal-v1.0.5-linux-x64
```

Launch through the included script:

```bash
./run.sh
```

Alternatively, open the extracted directory in your Linux file manager, right-click `run.sh`, and select **Run as a Program**. If that option is unavailable, open the file's **Properties > Permissions**, enable execution as a program, and try again.

Use `./run.sh --foreground` to keep the process attached for troubleshooting.

The package is self-contained and does not require a separate .NET installation. The launcher uses `~/.config/CS2RconTool/.net` for extracted native single-file libraries. It is a cache and can be deleted while the application is closed.

## Update The Application

Close the application, then either replace the existing application files with the contents of the new TAR.GZ or extract the new version to a new folder. The archive preserves executable permissions.

Writable settings are stored under `~/.config/CS2RconTool` in a typical Linux environment, not beside the executable. Back up that entire directory, including `credential.key`, before updating. Do not replace or delete it while updating application files.

## Configuration And Data

Open **Settings > Settings** for application-wide GeoLite, flag, slap-damage, refresh, output-filtering, and backup-directory options. Configure lifecycle, SteamCMD, RCON, database, ChatRelay, and Steam authentication values per server under **Servers > Manage servers > Edit server**.

The Linux data directory contains server profiles, scheduled tasks, application settings, protected credentials, Fun Stuff state, GeoLite data, and flags:

```text
~/.config/CS2RconTool
```

Saved secrets use AES-256-GCM. Keep `credential.key` with the JSON files when backing up or migrating data.

## Local Linux Server

Use a **Local Linux** profile only when the application and CS2 server run on the same Linux computer.

### Quick Start

The Edit server window requires an **RCON password** before it will save any profile. To then run **Install/Update server...** and download CS2 for the first time, the only other required field is an **install directory**. Leave **SteamCMD path**, **executable**, and **working directory** blank and the application derives and installs all three automatically: SteamCMD downloads into a sibling folder the first time it is needed, and the executable/working directory default to the standard `game/bin/linuxsteamrt64/cs2` paths under the install directory. App ID defaults to `730` with an anonymous SteamCMD login.

You also need to pick a **Startup mode** before Install/Update or Install/Update + Add-ons, since it controls how the application later starts and stops CS2:

- **Direct process**: the application builds the CS2 launch command and owns the process.
- **Service commands**: an existing systemd service owns CS2 and the application runs configured shell commands.

Before the first Local Linux **Install/Update server...** operation, review [SteamCMD Linux Prerequisites](#steamcmd-linux-prerequisites). SteamCMD needs Linux 32-bit runtime packages even on a 64-bit system (Debian/Ubuntu, Fedora/RHEL, and Arch families are detected automatically). If packages are missing, the application may open the desktop administrator authorization dialog to install them; this is expected prerequisite setup, not a server-plugin installation prompt.

Before you actually start the server for players, also configure the startup map, game type, game mode, maximum players, tickrate, VAC state, LAN mode, Startup CFG, and optional player password. Startup can use a local map, Workshop collection, or single Workshop map. The editable map field suggests Valve maps, including Premier maps, while accepting custom names. Public internet servers can also use a Steam Web API key and game-server login token; these are not required or prompted for when **LAN server** is enabled.

If you later change Startup mode on an already-running server, stop the current owner first; systemd cannot adopt a CS2 process that was already launched directly.

### Local Linux Direct Process

Use this mode when the application should launch and stop the executable itself.

Typical values:

```text
Executable:        <install-dir>/game/bin/linuxsteamrt64/cs2
Working directory: <install-dir>/game (But not required)
Arguments:         -dedicated -ip 0.0.0.0
```

The application adds authoritative map, game type, game mode, CFG, port, hostname, RCON, tickrate, VAC, LAN, Steam API, GSLT, and optional player-password arguments. The default `-ip 0.0.0.0` binds CS2 to all available network addresses; replace `0.0.0.0` with a specific address in the launch arguments when direct interface binding is required. An ordinary Start runs SteamCMD validation when SteamCMD and install-directory settings are configured. For troubleshooting, use **Start server without validation** from the server table's right-click menu to start without waiting for SteamCMD file validation. Restart does not run SteamCMD validation.

On supported Linux systems, the application can check SteamCMD prerequisites and bootstrap SteamCMD. Local profiles can also enable startup maintenance to warn players, stop CS2, update the server and supported add-ons, and restart it.

Local Linux Start and Restart show initial process output only through the startup check, then stop forwarding continuous CS2 runtime output. Explicit Local Linux console-fallback commands temporarily reopen output capture for their response.


### Local Linux Service Commands

Use this mode when an existing systemd service should own CS2. The application does not create the unit or sudoers policy.

Default commands target `cs2-server`:

```bash
sudo -n systemctl start cs2-server
sudo -n systemctl stop cs2-server
sudo -n systemctl restart cs2-server
```

These are application profile lifecycle commands, not systemd unit directives. The unit's `ExecStart` must invoke the CS2 launcher directly as its configured service user; do not put `sudo` in `ExecStart`.

The account running the application needs noninteractive sudo permission for these commands. Put all launch arguments, including `-tickrate`, `+sv_lan`, and `-secure` or `-insecure`, in the systemd launcher. Normal Service-command Start runs SteamCMD validation when SteamCMD and install-directory settings are configured; Restart skips validation.

Use the complete setup under [Linux Service Commands With systemd](#linux-service-commands-with-systemd). For a local profile, replace `<ssh-user>` in its sudoers example with the desktop account running the application.


### Finish Local Linux Setup

After the Local Linux profile is configured:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the Linux host firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**. For troubleshooting, **Start server without validation** starts a local Direct-process server without waiting for SteamCMD file validation.

Linux note: the first Local Linux SteamCMD operation may show an administrator authorization dialog while the application installs the required 32-bit runtime prerequisites. See [SteamCMD Linux Prerequisites](#steamcmd-linux-prerequisites) before cancelling that prompt.

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

Use a **Remote Linux** profile when CS2 runs on another Linux computer.

### Remote Linux Requirements

Configure:

- SSH host, port, username, and password.
- Optional SFTP address, username, password, and port. Blank values reuse SSH details; port `0` reuses the SSH port. Valid SFTP ports range through `65535`.
- Remote SteamCMD path and CS2 installation directory for managed updates.
- App ID `730` and Steam login, normally `anonymous`.
- The lifecycle mode and its corresponding fields.

The remote account must run lifecycle commands and write to the installation directory. Remote browsing and file operations require SFTP access. Before starting CS2, allow the configured game port through the server firewall over UDP and the configured RCON port over TCP. These are separate connections.

Remote Linux profiles reject Windows drive paths before SteamCMD starts. When a profile is changed from Remote Windows to Remote Linux, stale auto-generated executable and working-directory defaults are replaced with Linux defaults; explicitly configured valid custom paths remain editable.

### Remote Linux Hosting Panel

For a hosting provider that exposes a Pterodactyl-compatible panel but does not provide SSH, choose **Hosting panel API** in the Remote Linux **Startup mode** list. Configure:

- Panel URL, using the panel root such as `https://panel.example.com`.
- The panel server identifier or UUID.
- A client API key with power-control permission.
- SFTP address, username, password, and port for plugin deployment, browsing, and file operations.

Start, Stop, and Restart use the panel API and wait for the panel to report `offline` or `running`. SteamCMD validation and remote shell commands are unavailable in this mode. Debug Mode records panel requests, state polling, and failures without logging the API key.

### Remote Linux Account And Permissions

Prepare the remote accounts before configuring the application profile. The **SSH control account** is the username entered in the profile; it runs lifecycle commands and performs SFTP browsing, file transfers, plugin operations, backups, and SteamCMD operations. The **service account** owns and runs CS2 when using Service commands with systemd. These can be the same account, or they can be separate accounts.

The simplest setup uses the Linux account that owns the CS2 installation for both SSH and SFTP. If a dedicated SSH account does not already exist, create it as a normal user on the remote host and grant it SSH access:

```bash
sudo adduser <ssh-user>
sudo passwd <ssh-user>
```

Do not use `root` as the application, SteamCMD, or CS2 account. The SSH control account does not need unrestricted administrator access; it must be able to traverse every parent directory and read and write the CS2 installation. If it differs from the service account, grant access through a shared group or filesystem ACL rather than making the installation world-writable.

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

One ACL example is:

```bash
sudo setfacl -m u:<ssh-user>:x "<parent-directory>"
sudo setfacl -R -m u:<ssh-user>:rwX "<install-dir>"
sudo setfacl -R -d -m u:<ssh-user>:rwX "<install-dir>"
```

The first command-line SSH connection may ask you to trust the host fingerprint. The application uses its own SSH library and does not depend on the command-line client's `known_hosts` entry.


### Remote Linux Direct Process

Direct process mode runs CS2 without a systemd service. It is appropriate when the SSH account owns the installation and the application should manage the executable.

1. Select **Remote Linux (SSH)** and set **Startup mode** to **Direct process**.
2. Enter the install directory, for example `/home/cs2/cs2server`. Blank fields derive:

   ```text
   Executable:  /home/cs2/cs2server/game/bin/linuxsteamrt64/cs2
   Working dir: /home/cs2/cs2server/game (But not required)
   ```

3. Set the startup map, game type, and game mode. Put only optional switches in **Additional arguments**. The application supplies `-dedicated`, `-console`, port, hostname, `+map`, `+game_type`, `+game_mode`, `+exec`, `-usercon`, RCON password, tickrate, LAN mode, and VAC state. The default `-ip 0.0.0.0` binds to all available network addresses and permits remote connections; replace `0.0.0.0` with a specific address when direct interface binding is required.
4. Enter a path-specific Stop command. For the example above:

   ```bash
   executable='/home/cs2/cs2server/game/bin/linuxsteamrt64/cs2'; for process in /proc/[0-9]*; do target=$(readlink "$process/exe" 2>/dev/null || true); if [ "$target" = "$executable" ]; then kill -TERM "${process##*/}"; fi; done
   ```

5. Optionally enter the Steam API authentication key and GSLT. They are appended at launch and redacted from output.
6. Save the profile and use **Start**, **Stop**, or **Restart**.

Start refuses to run while SteamCMD is updating app `730`, verifies `game/csgo/gameinfo.gi`, supplies required Linux native-library paths, launches CS2 detached from SSH, and monitors it for 30 seconds. Startup output is captured in `/tmp/cs2-rcon-tool-startup-<port>.log`. Stop verifies that the exact executable exited.

Direct mode refuses to control an executable owned by the active `cs2-server` systemd cgroup. It does not start CS2 at boot or restart it after a crash.


### Linux Service Commands With systemd

Service commands mode requires an existing service. The application does not create privileged systemd or sudoers files. This pattern runs CS2 as the installation owner, supplies required native libraries, preserves crash exit codes for `Restart=on-failure`, and treats an explicit stop as clean.

Replace `<install-dir>`, `<service-home>`, `<service-user>`, `<service-group>`, `<ssh-user>`, hostname, map, and credentials.

Create a launcher named `cs2-rcon-tool-start`:

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

shutdown() {
   kill -TERM "$child" 2>/dev/null || true
   wait "$child" 2>/dev/null || true
   exit 0
}

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

Create `cs2-rcon-tool.sudoers` containing only the required lifecycle commands:

```sudoers
<ssh-user> ALL=(root) NOPASSWD: /usr/bin/systemctl start cs2-server, /usr/bin/systemctl stop cs2-server, /usr/bin/systemctl restart cs2-server, /usr/bin/systemctl status cs2-server, /usr/bin/systemctl is-active cs2-server
```

Install and validate from an administrator terminal. `<service-home>` must be the service account's real home, such as `/home/cs2`, not the CS2 installation directory. The launcher contains the RCON password, so restrict it to root and the service account's group.

```bash
sudo install -d -o root -g root -m 755 /usr/local/libexec
sudo install -o root -g <service-group> -m 750 ./cs2-rcon-tool-start /usr/local/libexec/cs2-rcon-tool-start
sudo install -o root -g root -m 644 ./cs2-server.service /etc/systemd/system/cs2-server.service
sudo visudo -cf ./cs2-rcon-tool.sudoers
sudo install -o root -g root -m 440 ./cs2-rcon-tool.sudoers /etc/sudoers.d/cs2-rcon-tool
sudo systemctl daemon-reload
sudo systemctl enable cs2-server.service
```

Configure a Remote Linux profile:

```text
Startup mode: Service commands
Start:   sudo -n systemctl start cs2-server
Stop:    sudo -n systemctl stop cs2-server
Restart: sudo -n systemctl restart cs2-server
Remote install directory: <install-dir>
```

For Local Linux, use the same lifecycle commands and configure the local install directory. Service launch arguments belong in the launcher, not in Direct-process fields.

Remote Service Start and Restart perform a SteamCMD/core-file preflight, resolve the expected executable, and require the service-owned process to remain alive for 30 seconds. Verification uses the exact executable when readable and otherwise requires a `cs2` process in the configured unit's exact cgroup. Stop requires the process to disappear.
Remote Linux Service Start and Restart also verify the installed Metamod loader and restore its `Game csgo/addons/metamod` entry in `game/csgo/gameinfo.gi` before launching the service. This repair only runs when a supported Metamod loader file is present; it does not install or update Metamod.


### Finish Remote Linux Setup

After the Remote Linux profile and its Direct process or Service commands setup are complete:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the remote Linux firewall allows the configured game port over UDP and RCON port over TCP.
6. Wait for the operation to finish and review **Console Commands**, **Application Log**, or **Debug**, then right-click the server again and select **Start server**. For Service commands, the configured systemd service remains the process owner when the app starts it.

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

Use a **Remote Windows** profile to manage a CS2 server on another Windows computer from the Linux application. Remote Windows management uses SSH and does not require WinRM.

### Remote Windows Requirements

- OpenSSH server access to the Windows computer.
- An SSH account in the remote computer's local Administrators group for WMI process creation and firewall configuration.
- Optional SFTP details for browsing and general transfers. Blank values reuse SSH details; port `0` reuses the SSH port. Valid ports range through `65535`.
- A Windows SteamCMD path and CS2 installation directory for managed updates.

Remote Windows plugin deployment and Direct-process startup do not require a separate SFTP service. Windows drive paths are required; Unix paths are rejected before SteamCMD starts. Switching a profile from Remote Linux replaces stale auto-generated executable and working-directory defaults with Windows defaults.


### Create The Windows SSH User

Perform this setup on the remote Windows computer from an elevated PowerShell prompt, before configuring the Linux application profile. Windows OpenSSH provides both SSH and SFTP; a separate SFTP server is not required. Use an existing suitable account instead if one is already available.

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

From the Linux computer running the application, test both SSH and SFTP before configuring the profile:

```bash
ssh cs2tool@<remote-windows-host>
sftp cs2tool@<remote-windows-host>
```

Use the account password, confirm SFTP can enter and write to `C:\CS2Server`, then exit both sessions. Use the same username and password in the application profile, and leave separate SFTP fields blank so the app reuses the SSH connection.

### Remote Windows Direct Process

Use this mode when the application should launch CS2 over SSH without a Windows service wrapper. For `C:\cs2server`, blank fields derive:

```text
Executable:        C:\cs2server\game\bin\win64\cs2.exe
Working directory: C:\cs2server\game (But not required)
SteamCMD path:     C:\cs2server-steamcmd\steamcmd.exe
```

Configure the startup map, game type, game mode, additional arguments, and Stop command. The application supplies `-dedicated`, `-console`, `-ip 0.0.0.0`, port, hostname, map, game type, game mode, Startup CFG, RCON password, tickrate, LAN mode, and VAC state. `0.0.0.0` binds to all available network addresses; replace it with a specific address in the launch arguments when you need to bind directly to one interface. Optional Steam API and GSLT values are appended and redacted from output.

Start transfers a temporary PowerShell launcher over SSH, starts CS2 through WMI independently of the SSH session, creates inbound UDP and TCP firewall rules, captures startup output, and monitors the process for 30 seconds. Restart runs Stop and then launches a new process. Direct mode refuses to control an executable owned by the active `cs2-server` service. It does not provide boot startup or automatic crash restart.


### Remote Windows Service Commands With WinSW

Windows service mode requires a wrapper because `cs2.exe` does not implement the Windows Service Control Manager protocol. This example uses [WinSW](https://github.com/winsw/winsw) and service name `cs2-server`. Run it once in an Administrator PowerShell terminal on the Windows server after stopping any Direct-process instance.

Keep the desktop app unelevated. During service setup, the Windows administrator must authorize the remote account to control the service and write to the installation. Universal does not elevate itself or change service permissions. See the [service-specific permission setup](README-WINDOWS.md#service-control-without-elevation); perform it on the Windows target, not the Linux desktop.

```powershell
$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'
New-Item -ItemType Directory -Force -Path C:\Services\cs2-server | Out-Null
Invoke-WebRequest `
    -Uri 'https://github.com/winsw/winsw/releases/download/v2.12.0/WinSW-x64.exe' `
    -OutFile 'C:\Services\cs2-server\cs2-server.exe'
```

Create `C:\Services\cs2-server\cs2-server.xml` beside the matching `cs2-server.exe`. `<id>` and `<executable>` are required; `<name>` is the display name in Services. Replace the password after `+rcon_password` in `<arguments>` with the RCON password in Universal. Steam API key (`-authkey`) and GSLT (`+sv_setsteamaccount`) are optional: remove those switches if unused, or supply valid values. Replace paths, hostname, and map; XML-escape `&`, `<`, and `>` in values.

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

Before first start, prepare Valve's Steamworks SDK Redist app `1007` on the Windows computer. A LocalSystem service cannot rely on your desktop Steam installation. Missing runtime files can cause **Could not determine Steam client install directory** and a repeated crash/restart loop. In Administrator PowerShell, with the service and any Direct-process instance stopped, adjust these sample paths:

```powershell
$ErrorActionPreference = 'Stop'
$installDir = 'C:\cs2server'
$steamCmdPath = 'C:\cs2server-steamcmd\steamcmd.exe'
$runtimeDir = Join-Path (Split-Path -Parent $steamCmdPath) 'steamworks-sdk-redist'
$destination = Join-Path $installDir 'game\bin\win64'
& $steamCmdPath +force_install_dir $runtimeDir +login anonymous +app_update 1007 validate +quit
if ($LASTEXITCODE -ne 0) { throw 'SteamCMD runtime installation failed.' }
foreach ($file in @('steamclient64.dll', 'tier0_s64.dll', 'vstdlib_s64.dll')) {
   $source = Get-ChildItem -LiteralPath $runtimeDir -Recurse -File -Filter $file |
      Select-Object -First 1
   if (-not $source) { throw "Missing Steamworks runtime file: $file" }
   Copy-Item -LiteralPath $source.FullName -Destination (Join-Path $destination $file) -Force
}
```

Install the wrapper only if the service ID is not already registered, create firewall rules, and start the service:

```powershell
New-Item -ItemType Directory -Force -Path C:\cs2server\logs | Out-Null
Set-Location C:\Services\cs2-server
.\cs2-server.exe install
New-NetFirewallRule -DisplayName 'CS2 RCON Tool UDP 27015' -Direction Inbound -Action Allow -Protocol UDP -LocalPort 27015 -Program 'C:\cs2server\game\bin\win64\cs2.exe' -Profile Any
New-NetFirewallRule -DisplayName 'CS2 RCON Tool TCP 27015' -Direction Inbound -Action Allow -Protocol TCP -LocalPort 27015 -Program 'C:\cs2server\game\bin\win64\cs2.exe' -Profile Any
Start-Service -Name 'cs2-server'
Get-Service -Name 'cs2-server'
Get-Process -Name 'cs2'
```

Configure the profile:

```text
Startup mode: Service commands
Start:   powershell -NoProfile -NonInteractive -Command "Start-Service -Name 'cs2-server'"
Stop:    powershell -NoProfile -NonInteractive -Command "Stop-Service -Name 'cs2-server'"
Restart: powershell -NoProfile -NonInteractive -Command "Restart-Service -Name 'cs2-server'"
```

All launch arguments belong in the WinSW XML. WinSW writes rolled output and error logs under `C:\cs2server\logs` and restarts CS2 after an unexpected exit.

`<startmode>Automatic</startmode>` starts the service at Windows boot without Universal; installation alone does not start it. Windows boot and Services commands do not run Universal's SteamCMD validation; normal Start in Universal validates when configured, while Restart skips validation. Confirm CS2 stays running, not just that the wrapper briefly reports Running.

If the service is not visible, press F5 in `services.msc` and look for its display name, or run `Get-Service -Name 'cs2-server'`. To locate the installed wrapper, run `Get-CimInstance Win32_Service -Filter "Name='cs2-server'" | Select-Object Name, DisplayName, State, PathName`. For Error 1053, check XML syntax, closed tags, and paths; inspect `cs2-server.wrapper.log` and CS2 output under `<logpath>`. Wrapper logs contain launch credentials: redact them before sharing.

To reuse a service with a different CS2 directory, stop it and update `<executable>`, `<workingdirectory>`, `<logpath>`, launch arguments, and firewall paths. Keep the registered wrapper path and `<id>` unchanged. Moving the wrapper itself requires uninstalling from the old location and installing from the new one; changing only the CS2 directory does not.

### Finish Remote Windows Setup

After the Remote Windows profile and its Direct process or Service commands setup are complete:

1. Save the server profile.
2. Select the server in the top server list.
3. Right-click the server and select **Install/Update server...** to install or validate CS2 with SteamCMD.
4. If you also need the supported frameworks and tracked plugins maintained, select **Install/Update Server + Add-ons** instead.
5. Confirm the remote Windows firewall allows the configured game port over UDP and RCON port over TCP.
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

| Linux profile and mode | Start | Restart |
| --- | --- | --- |
| Local Linux, Direct process | Runs validation when SteamCMD and install-directory settings are configured | Does not run validation |
| Local Linux, Service commands | Runs SteamCMD validation when configured, then the configured Start command | Runs the configured Restart command without validation |
| Remote Linux, Direct process | Runs SteamCMD validation, then starts and monitors the process | Stops and starts without validation |
| Remote Linux, Service commands | Runs SteamCMD validation, then runs the service command and monitors its process | Runs Restart and monitors its process without validation |

Normal **Start server** validates before startup when SteamCMD is configured. Use **Validate server files** to validate without starting the server, or **Install/Update Server** to update, repair, and validate the CS2 installation.

### SteamCMD Linux Prerequisites

SteamCMD is a 32-bit Linux binary, so 64-bit systems need 32-bit runtime packages before downloading or running SteamCMD. The application detects the distribution family from `ID` and `ID_LIKE` in `/etc/os-release`:

| Family | Detected when `ID`/`ID_LIKE` contains | Required packages |
| --- | --- | --- |
| Debian / Ubuntu (Mint, Pop!_OS, and others) | `debian` or `ubuntu` | `i386` architecture, `libc6-i386`, `lib32gcc-s1`, `lib32stdc++6` |
| Fedora / RHEL (Rocky, Alma, CentOS) | `fedora`, `rhel`, or `centos` | `glibc.i686`, `libstdc++.i686` |
| Arch (Manjaro, EndeavourOS) | `arch` | `lib32-gcc-libs` (requires the `[multilib]` repository) |

When any prerequisite is missing, the application asks for desktop administrator authorization before installing it. On other distributions, or when automatic installation is unavailable, install the packages manually. Debian / Ubuntu:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install -y libc6-i386 lib32gcc-s1 lib32stdc++6
```

Fedora / RHEL:

```bash
sudo dnf install -y glibc.i686 libstdc++.i686
```

Arch (enable `[multilib]` in `/etc/pacman.conf` first):

```bash
sudo pacman -S --needed lib32-gcc-libs
```

Do not run SteamCMD or the CS2 server as `root`. Run the application and SteamCMD under the normal account that owns the SteamCMD and CS2 installation directories, and grant that account only the filesystem and service permissions it needs.

On first launch, SteamCMD updates its own files before accepting commands. The application performs that bootstrap and uses anonymous Steam login by default for CS2 App ID `730`. SteamCMD runs through the application, and its progress and output are captured in **Console Commands**, **Application Log**, and **Debug** tabs so users can monitor installation and validation without a separate interactive terminal.

These are operating-system packages for SteamCMD, not CS2 server plugins. Metamod, CounterStrikeSharp, RCON Tool Companion, ChatRelay, and other add-ons are installed separately on the managed CS2 server as described in [Frameworks And Plugins](#frameworks-and-plugins) and [RCON Tool Companion And Fun Stuff](#rcon-tool-companion-and-fun-stuff).

When a managed local CS2 process is running, the application pauses before SteamCMD and offers to stop it and continue. Cancelling leaves the server running and aborts the update or validation. After SteamCMD restores core files, **Install/Update Server** reapplies and validates the Metamod/CounterStrikeSharp loader chain. **Install/Update Server + Add-ons** combines the core update with framework and tracked-plugin maintenance.

## Using The Application

Add server provisioning can reuse existing installations. Before changing files, it stops a local service through the configured Stop command or stops a detected local Direct-process target; remote profiles use their verified Stop path. If stopping fails or local `gameinfo.gi` remains locked, provisioning aborts before installation changes. Keep other applications from starting the target during provisioning. Plugin stages leave it stopped; start-after-install starts that profile only after all selected stages succeed. If unchecked, or if provisioning fails, the target stays stopped. The app account needs permission to execute the configured lifecycle commands.

### Connect And Refresh

1. Click **Add server** in the top toolbar and choose the location-specific startup method.
2. Enter a profile name, host/IP, RCON port, and RCON password. Enable **LAN server** when appropriate; LAN hides the Steam credential fields. With LAN off, Steam API key and GSLT fields are shown for app-managed launches but remain optional; Service commands use credentials from the service configuration.
3. For remote profiles, enter SSH credentials before browsing. For install actions, choose the CS2 install folder in the wizard; SteamCMD is installed or discovered automatically. Other advanced lifecycle fields remain under **Edit server**.
4. Configure Workshop, database, and ChatRelay integration values when used.
5. Save the profile, select it in the top server list, and click **Refresh server list**.

At the end of the Add server wizard, choose to finish setup or install/update the server, optionally with Metamod and CounterStrikeSharp, and optionally start it after successful installation. Hosting panel API profiles expose framework ownership; provider-managed CS2 core updates remain in the panel.

Use **Help > Sample server profile** for a complete read-only example of the available fields. It uses placeholder values and is not added to the server list.

RCON uses TCP. Opening only the UDP game port is not sufficient. The top list displays hostname, address, port, map, players, ping, location, and country. **Reload server list** reads saved profiles without contacting servers; **Refresh server list** queries live A2S information.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete CS2 installations, plugins, backups, or managed-server files.

### Server Actions And Joining

Server Actions controls hostname, bots, teams, passwords, friendly fire, cheats, pause, maps, maximum rounds, timed restarts, and kick/ban/slay/slap operations. These are runtime commands and never edit server CFG files. Values in a Startup CFG can replace runtime changes when a map executes it.

The map selector lists playable maps and configured Workshop maps.

Select a server and click **Join server**. Linux tries Steam and available desktop URI handlers. If Steam opens CS2 without connecting, enable the developer console, press `~`, and paste the copied `connect IP:port` command.

### Settings

Open **Settings > Settings**. The **General** tab contains GeoLite and flag URLs, Slap damage from `0` to `99`, automatic 15-second server-list refresh, noisy-output filtering, startup update checks, **Minimize to tray**, and **Update GeoLite + flags**. The **Backups** tab provides separate server-file and database backup destinations. **Help > Check for Updates** only notifies and links to GitHub; it never downloads or installs the app.

Server-specific lifecycle, SteamCMD, map, Workshop, RCON, player-password, database, ChatRelay, Steam authentication, and backup credentials remain under **Servers > Manage servers > Edit server**. Protected values are encrypted before storage.

### Console, Logs, And Debug

Console Commands accepts free-form RCON commands and categorized suggestions. Each server has separate bounded history, and process, SteamCMD, SSH, and RCON output remains associated with its originating server. Application Log reports normal operations and failures; Debug provides detailed diagnostics. Passwords, credentials, API keys, and tokens are redacted even in Debug Mode and raw output.

When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the application or **Exit program** to close anyway.

### Scheduled Tasks

Task types include RCON commands, framework/plugin maintenance, server backups, database backups, and GeoLite/flag updates. Triggers include Day, Hour, and Startup. The built-in scheduler runs while the application is open and monitoring is active. Native Linux tasks use systemd user timers and invoke the headless task runner, so the GUI does not need to remain open.

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

PlayerPunishments is a CounterStrikeSharp server plugin. Install it in the managed Linux CS2 server's CounterStrikeSharp plugin directory, not in the desktop application's folder. Alive and Health display **Unknown** when the required server-side player-information command is unavailable.

## ChatRelay

ChatRelay is a separate [CounterStrikeSharp plugin](https://github.com/ShAgGy2035/ChatRelay); its binaries are not bundled with this application.

1. Install and configure ChatRelay on the CS2 server using its repository instructions.
2. In the Chat tab, select the receiving network adapter and UDP port; the default is `9090`.
3. Set a unique high-entropy token under **Edit server > Integrations**.
4. Use **Copy ChatRelay target JSON** to create the matching `TargetIp`, `TargetPort`, and `SharedToken` entry.
5. Use the local authenticated **Test** action and send admin chat through RCON as needed.

The listener accepts authenticated JSON only, limits datagrams to 8 KiB, and drops rejected packets. UDP does not encrypt traffic or prove the sender's network address. Restrict firewall access to the game server or trusted LAN, or use a VPN rather than exposing it directly to the internet.

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

Optional plugins can come from GitHub, a public GitLab.com project including nested groups, or a supported local archive or library. Assets are filtered by server OS. When multiple packages match, the application asks which asset to install and remembers a version-independent filename preference. Installed files are tracked in `game/csgo/.cs2-rcon-tool-plugins.json`; keep this ownership manifest with the server so uninstall removes only owned files and preserves shared configuration and framework files.

Use the **Manage Metamod plugins** window only for packages containing an `addons/metamod` payload. Use the **Manage CounterStrikeSharp plugins** window for CounterStrikeSharp plugin packages. The application rejects packages submitted through the wrong manager before deployment.

Remote updates replace native binaries atomically rather than truncating loaded files. A running server keeps using the previous binary until restart, preventing memory-mapped library crashes. Local plugin batches stop CS2 once before replacing plugin/shared API assemblies and restart it without SteamCMD.

## Backups, Data, And Security

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking rather than stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` with the selected server's integration settings.
- **Restore server backup...** accepts ZIP, TAR.GZ, or TGZ, validates it, stops CS2, restores supported content, and restarts it.
- **Install/update + restore server backup...** updates core files before restoring.
- **Restore database backup...** imports SQL with the configured MySQL client.
- **Validate server files** runs SteamCMD with `validate`.

Writable data is stored under `~/.config/CS2RconTool`, not beside the executable. It includes `servers.json`, `tasks.json`, `steam_settings.json`, `fun_stuff_state.json`, `app_state.json`, `credential.key`, GeoLite data, and flags. Startup validates the JSON files and moves invalid files into timestamped `migration-backups` folders. Saved secrets use AES-256-GCM. Protect and back up `credential.key` with the JSON files. Release archives exclude runtime settings, keys, caches, PDBs, ChatRelay binaries, and Companion binaries.

For migration, open the new application once and close it, back up the generated data directory, then copy the old JSON files and matching `credential.key`. Re-enter and save any credential that cannot be decrypted. Imported `Local Windows` profiles must be changed to `Local Linux` with Linux executable, SteamCMD, install, working-directory, and launch paths before they can be saved. Remote profiles remain usable with valid target paths and credentials.

## Troubleshooting

### RCON commands fail

- Verify the host/IP and RCON TCP port.
- Confirm firewalls permit the game UDP port and RCON TCP port.
- Verify listening sockets with `ss -lntup | grep 27015`, replacing the port as needed.

### A Linux server in a bridged VirtualBox VM is unreachable

- Use the guest's LAN address rather than the host address or `127.0.0.1`.
- Ensure `/etc/hosts` maps `127.0.1.1` to the VM hostname, for example `127.0.1.1 <vm-hostname>`.
- If VirtualBox created a Host-Only adapter such as `Ethernet 2` and it interferes with routing, disable it on the host while keeping the physical Ethernet/Wi-Fi adapter enabled.
- Keep `-ip 0.0.0.0` in Direct-process arguments for remote access, or replace it with a specific address. Put the chosen `-ip` value in the systemd launcher for Service commands mode.

### Local server exits immediately

- Verify the executable, SteamCMD path, install directory, profile OS, and launch arguments. Working directory can be left blank because it is derived from the install directory.
- Check Application Log and Debug for the exit code and stderr.
- Confirm required Linux runtime libraries and Steam client files are available.
- Do not run Direct process and Service commands modes against the same executable and port.

### Remote lifecycle, browsing, or installation fails

- `Connection refused` means no SSH service accepted the configured host and port; authentication errors mean the SSH username or password was rejected.
- `Permission denied` usually means the account cannot traverse a parent directory or access the installation. Run the checks under [Remote Linux Account And Permissions](#remote-linux-account-and-permissions).
- Confirm lifecycle commands do not require an interactive privilege prompt.
- Remote archive operations require `tar` and either `unzip` or `python3` for ZIP files.
- Framework and SteamCMD downloads require `curl`, `wget`, or `python3`; the application uses the first available option.
- Remote SteamCMD validation opens **Console Commands** and streams output while it runs.

### Metamod or CounterStrikeSharp commands are unknown

- Local Service commands Start restores the installed Metamod search-path entry after validation and before starting the service. If validation removed the entry, Stop and then Start through Universal; Restart only invokes the saved lifecycle command.
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
- Start monitoring for built-in tasks and inspect the systemd user timer for native tasks.

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
