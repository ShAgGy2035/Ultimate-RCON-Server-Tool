# Ultimate CS2 RCON and Server Management Tool - Linux Guide

Version: 1.0.6

This guide covers installing Universal, configuring Local Linux or Remote Linux/Windows servers, using optional plugins, and troubleshooting. [README.md](README.md) provides the project overview.

Download version 1.0.6 from [GitHub Releases](https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/releases/tag/v1.0.6) using the package names below.

## Table of Contents

- [Install And Launch](#install-and-launch)
- [Update The Application](#update-the-application)
- [Configuration And Data](#configuration-and-data)
- [Local Linux Server](#local-linux-server)
- [Remote Linux Server](#remote-linux-server)
- [Remote Windows Server](#remote-windows-server)
- [SteamCMD Behavior](#steamcmd-behavior)
- [Frameworks And Plugins](#frameworks-and-plugins)
- [RCON Tool Companion And Fun Stuff](#rcon-tool-companion-and-fun-stuff)
- [PlayerPunishments Setup](#playerpunishments-setup)
- [ChatRelay](#chatrelay)
- [Using The Application](#using-the-application)
- [Backups, Data, And Security](#backups-data-and-security)
- [Troubleshooting](#troubleshooting)
- [Issues](#issues)
- [About](#about)
- [Application Screenshots](#application-screenshots)

## Install And Launch

Use a Linux x64 desktop. Universal includes its .NET runtime; server plugins need their own compatible CounterStrikeSharp installation.

Extract the Linux package and open its directory:

```bash
tar -xzf cs2-rcon-tool-universal-v1.0.6-linux-x64.tar.gz
cd cs2-rcon-tool-universal-v1.0.6-linux-x64
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

If SteamCMD is not already installed, prepare an independently obtained publisher SHA-256 checksum before setup. Universal will request it before downloading SteamCMD; without one, cancel or use an existing verified installation.

The Edit server window requires an **RCON password** for a local profile. To then run **Install/Update server...** and download CS2 for the first time, the only other required field is an **install directory**. Leave **SteamCMD path**, **executable**, and **working directory** blank and the application derives and installs all three automatically: SteamCMD downloads into a sibling folder the first time it is needed, and the executable/working directory default to the standard `game/bin/linuxsteamrt64/cs2` paths under the install directory. App ID defaults to `730` with an anonymous SteamCMD login.

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

Choose the SSH or hosting-panel path below, then complete [connection security](#remote-rcon-transport-and-host-pins) before testing commands or file access.

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

Start, Stop, and Restart use the panel API and wait for the panel to report `offline` or `running`; console commands, desktop operations, and scheduled/headless RCON use the panel's short-lived-token WSS console. Panel permissions must include console access. HTTPS/WSS use normal certificate validation and redirects are rejected. The game RCON password is optional and is never sent to the panel. Keep SFTP credentials and independently verified host pins for file operations; SSH credentials are not required for panel console/lifecycle access. SteamCMD validation and remote shell commands are unavailable in this mode. Debug Mode excludes API keys, JWTs, and configured endpoints. Marker-delimited output has a short bounded post-marker window for asynchronous replies; unrelated server logs can interleave and later output may be missed because the protocol does not correlate output to individual commands.

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

Independently verify the host's SHA256 fingerprint through a trusted channel before trusting it in any client. Universal uses its own explicit endpoint-bound pins in **Edit server > Transport security**, not the command-line client's `known_hosts`; it never automatically accepts a new or changed host key.

### Remote RCON Transport And Host Pins

Remote RCON is blocked unless one of these is configured, listed from most to least secure:

1. **Verified SSH tunnel (recommended).** In **Edit server > Transport security**, tick **Enable verified SSH tunnel**, save the SSH username and password, and enter the pin host, port, and fingerprint below. RCON is then sent through the tunnel to the server's own `127.0.0.1`.
2. **Hosting panel console** (Remote Linux, Hosting panel API). The panel's shared console can drop lines and echo `rct_..._begin/_end` markers, so the player table may occasionally miss rows; Universal reuses the last complete roster to compensate.
3. **Unencrypted direct RCON (explicit exception).** Tick **Allow UNENCRYPTED direct RCON to this remote server** only when the provider offers neither SSH nor a reliable console. The RCON password and every command then cross the network in clear text. It is off by default, applies per server, requires the RCON TCP port to be reachable and the RCON password saved in the profile, and is never enabled automatically. Use a unique RCON password and restrict the port to your address where the provider allows it. Use this exception only if you accept that exposure.

**Getting the host fingerprint.** The pin is the SHA256 of the server's SSH host key as `SHA256:` followed by 43 unpadded base64 characters, the same format `ssh-keygen -lf` prints. Read it on the server itself or from the provider's trusted console, never from the connection you are about to trust:

- Linux: `ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`
- Windows (OpenSSH): `ssh-keygen -lf C:\ProgramData\ssh\ssh_host_ed25519_key.pub`

Enter the exact host and port the profile connects to, plus the fingerprint, in the SSH pin fields. A different key type, host, or port needs its own explicit pin. A mismatch is refused without fallback.

**Symptom: players appear as "Bot 1", "Bot 2".** The app could not run RCON `status` and fell back to the A2S query, which has no names. Check that the profile uses one of the transports above (for a lab or remote server on a private address, normally the verified SSH tunnel) and that the SSH password and pin are saved.

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

Before entering credentials, choose a [protected connection and verify the server fingerprint](#remote-rcon-transport-and-host-pins). SSH/SFTP operations reject unknown or changed fingerprints.

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

```powershell
$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'
New-Item -ItemType Directory -Force -Path C:\Services\cs2-server | Out-Null
Invoke-WebRequest `
    -Uri 'https://github.com/winsw/winsw/releases/download/v2.12.0/WinSW-x64.exe' `
    -OutFile 'C:\Services\cs2-server\cs2-server.exe'
```

Create `C:\Services\cs2-server\cs2-server.xml`, replacing the sample hostname, map, passwords, Steam API key, and GSLT. XML-escape `&`, `<`, and `>` in values.

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

Install the wrapper, create firewall rules, and start the service:

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

A missing SteamCMD installation requires an independently obtained publisher SHA-256 in the manual integrity dialog before extraction or startup. This applies to local Linux and remote Windows/Linux bootstrap. Unverified approval and unattended bootstrap are blocked; existing configured installations retain normal update/validate behavior. Automatic trusted Valve checksum discovery is not available; cancel if no independent publisher hash is available. See the shared README for provenance limitations.

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

## Frameworks And Plugins

Install Metamod before CounterStrikeSharp. Interactive installation shows the latest 20 compatible releases and accepts a selected release or direct HTTPS ZIP, TAR.GZ, or TGZ URL. Scheduled and headless maintenance use the newest compatible releases because they cannot display selection dialogs. Application downloads require certificate revocation checks and reject plaintext URLs and redirect downgrades. Native remote Linux downloads require curl with HTTPS-only redirect support; wget/Python fallbacks are removed, and missing curl fails explicitly. Add-on downloads on remote Linux are SHA-256 checked on the target before extraction whenever an expected hash is available. Framework and optional-plugin packages with a valid publisher SHA-256 are verified before extraction; mismatches block installation. Missing hashes show **Integrity unverified** and need an independently obtained publisher hash or one-time manual approval; unattended tasks cannot approve. See the shared README for remaining provenance and SteamCMD gaps.

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

## RCON Tool Companion And Fun Stuff

Select **Fun Stuff**, choose a mode, customize its settings, and start it. Use [RCON Tool Companion 1.0.3](https://github.com/ShAgGy2035/RconCompanionTool) with the full CounterStrikeSharp 1.0.376 runtime. Install it on the CS2 server, not beside Universal.

Modes include No Bots, AWP / Sniper Wars, Bhop, Deathmatch, Flying Dutchman, Grenade Wars, Molotov / Incendiary Wars, Headshot Only, Knife Arena, One in the Chamber, Pistols Only, ScoutzKnives, Super Hero, Surf, Weapon Rotation, and Zeus Wars. Super Hero offers fourteen configurable powers; each mode has its own gameplay, bot, and supported map settings.

New mode configurations have bots disabled. Enable them per mode if wanted; saved choices are preserved. Bot counting can use `normal` (fixed count), `fill` (maintain the total as humans join), or `match`. A blank enabled quota keeps the current value.

Modes temporarily change running-server settings, not CFG files. **None (rollback)** restores captured values. Some modes reload the map when starting or ending; expect connected players to see a map transition.

In game, trusted administrators with `@css/fun` can type `!fun` or `/fun`: **W/S** moves, **E** selects, **A** goes back, and **R** exits. No separate menu plugin is needed. All configurable modes use the quick-start wizard with Start, optional Customize gameplay, and Bots. Start applies choices only to that run, without saving to `RCT.json`; cancellation discards them. Temporary choices survive maps, but plugin reload/server restart uses configured defaults. The old editable menu is removed and `UseFunSetupWizard` is ignored. Separate Bot Control preferences still save when applied; Universal and scheduled-start behavior is unchanged.

Companion runs the active mode on the server and, by default, restores normal settings after five minutes without human players. Configure this under **Empty-server rollback**. Closing Universal does not stop a Companion-owned mode.

### Weapon Rotation

1. Open **Fun Stuff > Weapon Rotation**. Its tabs are Gameplay, Weapons, Rotation, Bots, and Maps.
2. Select categories and weapons; at least one selected weapon must belong to an enabled category.
3. Choose team rounds, team respawn, or free-for-all; give everyone the same weapon or use independent assignments.
4. Set the rotation interval (5-3600 seconds, default five minutes). **Duration minutes** limits the whole mode, not each weapon: `0` is unlimited; 1-1440 ends with Rollback ALL, including Bot Control.
5. Choose Popup, Chat, or both for announcements. Chat reminders default to 30 seconds; `0` disables periodic reminders.
6. Configure an optional Workshop map or collection, review, and start. Map settings are configured here or in `RCT.json`, not through `!fun`.

New settings use category rotation, shuffled weapons, infinite ammunition, no bots, and category voting. Players select the next category in a popup showing all six choices; `!wrvote` reopens it. Chat announces invitations/results but does not accept votes. Voting normally opens 30 seconds before a change and closes five seconds before it; very short intervals shorten or skip voting.

Weapons change while players are alive, and respawning keeps the current assignment. Team rounds preserve bomb objectives. Map voting and other menus take priority over weapon notices.

Use the editors rather than guessing JSON values. If editing by hand, follow the Companion README's **Settings with fixed values** reference. An active run keeps its starting settings; roll back and restart to apply changed defaults.

### Bot Control

The separate **Bot Control** tab adjusts count, difficulty, teams, and halftime over compatible modes. **Restore bots ONLY (keep mode)** restores bot changes; **Rollback ALL (mode + bots)** restores the pre-mode setup. Free-for-all modes reject incompatible team/halftime choices, and No Bots blocks conflicting bot edits.

## PlayerPunishments Setup

The app's Slay and Slap controls send `css_slay #userid` and `css_slap #userid damage`. Install [PlayerPunishments 1.0.1](https://github.com/ShAgGy2035/PlayerPunishments) on each managed CS2 server when its existing administration package does not provide those commands. Configure Slap damage from `0` to `99` under **Settings > General**.

PlayerPunishments is a CounterStrikeSharp server plugin. Install it in the managed Linux CS2 server's CounterStrikeSharp plugin directory, not in the desktop application's folder. Alive and Health display **Unknown** when the required server-side player-information command is unavailable.

## ChatRelay

ChatRelay is a separate [CounterStrikeSharp plugin](https://github.com/ShAgGy2035/ChatRelay); its binaries are not bundled with this application.

1. Install and configure ChatRelay on the CS2 server using its repository instructions.
2. In the Chat tab, select the receiving network adapter and UDP port; the default is `9090`.
3. Generate a unique secure key under **Edit server > Integrations**. It is 32 cryptographically random bytes encoded as base64.
4. Use **Copy ChatRelay target JSON** to create the matching `TargetIp`, `TargetPort`, and `KeyBase64` entry.
5. Use the local authenticated **Test** action and send admin chat through RCON as needed.

ChatRelay v1 encrypts chat with AES-256-GCM; its 12-byte nonce, version, algorithm, and timestamp are authenticated, and the shared key is not sent over UDP. The listener limits datagrams to 8 KiB, accepts timestamps within ±120 seconds, and persists a bounded replay journal across listener restarts. Legacy plaintext sender and receiver paths are removed. Existing saved tokens and legacy flags are retained only for migration diagnostics and are never used; a legacy-only profile cannot start the listener or send until a matching secure key is configured. Keep the existing firewall/router policy restrictive; UDP peer identity is not provided by encryption.

## Using The Application

### Connect And Refresh

1. Open **Servers > Manage servers**.
2. Add a server and enter its name, host/IP, game/RCON port, RCON password, and location.
3. Complete lifecycle and SteamCMD fields when the application will manage those operations.
4. Configure Workshop, database, and ChatRelay integration values when used.
5. Save the profile, select it in the top server list, and click **Refresh server list**.

RCON uses TCP. Opening only the UDP game port is not sufficient. The top list displays hostname, address, port, map, players, ping, location, and country. **Reload server list** reads saved profiles without contacting servers; **Refresh server list** queries live A2S information.

Deleting a server requires confirmation. Optional app-state cleanup removes that profile's scheduled tasks, external scheduler entries, console history, and cached state. It does not delete CS2 installations, plugins, backups, or managed-server files.

### Server Actions And Joining

Server Actions controls hostname, bots, teams, passwords, friendly fire, cheats, pause, maps, maximum rounds, timed restarts, and kick/ban/slay/slap operations. These are runtime commands and never edit server CFG files. Values in a Startup CFG can replace runtime changes when a map executes it.

The map selector lists playable maps and configured Workshop maps.

Select a server and click **Join server**. Linux tries Steam and available desktop URI handlers. If Steam opens CS2 without connecting, enable the developer console, press `~`, and paste the copied `connect IP:port` command.

### Settings

Open **Settings > Settings**. The **General** tab contains GeoLite and flag URLs, Slap damage from `0` to `99`, automatic 15-second server-list refresh, noisy-output filtering, and **Update GeoLite + flags**. The **Backups** tab provides separate server-file and database backup destinations.

Server-specific lifecycle, SteamCMD, map, Workshop, RCON, player-password, database, ChatRelay, Steam authentication, and backup credentials remain under **Servers > Manage servers > Edit server**. Protected values are encrypted before storage.

### Console, Logs, And Debug

Console Commands accepts free-form RCON commands and categorized suggestions. Each server has separate bounded history, and process, SteamCMD, SSH, and RCON output remains associated with its originating server. Application Log reports normal operations and failures; Debug provides detailed diagnostics. Passwords, credentials, API keys, and tokens are redacted even in Debug Mode and raw output.

When closing with Debug Mode enabled, choose **Stop debugging** to disable it and return to the application or **Exit program** to close anyway.

### Scheduled Tasks

Task types include RCON commands, framework/plugin maintenance, server backups, database backups, and GeoLite/flag updates. Triggers include Day, Hour, and Startup. The built-in scheduler runs while the application is open and monitoring is active. Native Linux tasks use systemd user timers and invoke the headless task runner, so the GUI does not need to remain open.

**Fun Stuff rotation** tasks start one or more Fun Stuff modes in order. Each step has a mode and a duration in minutes; after the last step the rotation can stay on the last mode, roll it back, or repeat from the first step. A single step with **Stay on last mode** simply switches the server into that mode at the trigger time. Modes are started by RCON Tool Companion 1.0.3 using its `RCT.json` settings (also editable in game through `!fun`). Rotations run only in the built-in scheduler while the app is open; editing or deleting the task, or stopping the scheduler, ends a running rotation and leaves the current mode active.

## Backups, Data, And Security

- **Backup server** archives configuration, add-ons, maps, `gameinfo.gi`, and plugin tracking rather than stock CS2 binaries or Workshop downloads.
- **Backup database** runs `mysqldump` with the selected server's integration settings.
- **Restore server backup...** accepts ZIP, TAR.GZ, or TGZ, validates it, stops CS2, restores supported content, and restarts it.
- **Install/update + restore server backup...** updates core files before restoring.
- **Restore database backup...** imports SQL with the configured MySQL/MariaDB client. Restore requires `--binary-mode` to disable local shell and file-source commands, including for loopback connections; unsupported clients fail before credentials are sent. SQL and `DELIMITER` routines remain supported. Only restore trusted SQL because database permissions and server-side effects are unchanged.
- **Validate server files** runs SteamCMD with `validate`.

Writable data is stored under `~/.config/CS2RconTool`, not beside the executable. It includes `servers.json`, `tasks.json`, `steam_settings.json`, `fun_stuff_state.json`, `app_state.json`, `credential.key`, GeoLite data, and flags. Startup validates the JSON files and moves invalid files into timestamped `migration-backups` folders. Saved secrets use AES-256-GCM. Protect and back up `credential.key` with the JSON files. Release archives exclude runtime settings, keys, caches, PDBs, ChatRelay binaries, and Companion binaries.

For migration, open the new application once and close it, back up the generated data directory, then copy the old JSON files and matching `credential.key`. If credentials cannot be decrypted, restore the matching original key and data files; the application stops loading protected settings rather than substituting empty values. Imported `Local Windows` profiles must be changed to `Local Linux` with Linux executable, SteamCMD, install, working-directory, and launch paths before they can be saved. Remote profiles remain usable with valid target paths and credentials.

## Troubleshooting

### RCON commands fail

- Verify the profile address and selected connection method. For an SSH tunnel, check SSH access and the saved server fingerprint; for a hosting panel, check its URL, API permissions, and console access.
- Check the ports needed for your chosen connection method. A verified SSH tunnel or hosting-panel console does not require exposing the game RCON port.
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
- Remote framework and SteamCMD downloads require `curl` with HTTPS-only initial/redirect support; missing curl fails explicitly, with no wget/Python fallback.
- Remote SteamCMD validation opens **Console Commands** and streams output while it runs.

### Metamod or CounterStrikeSharp commands are unknown

- Install or update Metamod first, then install or update CounterStrikeSharp.
- Restart CS2 after installing or updating either framework.
- Use **Settings > Test Metamod/CSS command availability** to check whether the framework commands are loaded.
- If the commands are still unavailable, stop CS2, run **Install/Update Server** to repair and validate the core files, reinstall Metamod and CounterStrikeSharp, and start CS2 again.

### Join Server opens CS2 but does not connect

- Open the CS2 developer console with `~` and paste the copied `connect IP:port` command.

### Chat does not appear

- Verify ChatRelay is running, match the destination IP, adapter, UDP port, and secure key, check firewall rules, and use the Chat tab's **Test** action.

### Scheduled tasks do not run

- Confirm the task is active and valid.
- Start monitoring for built-in tasks and inspect the systemd user timer for native tasks.

### Country or flag data is missing

- Run **Settings > General > Update GeoLite + flags**.
- Confirm the configured URLs are direct downloads and the per-user data directory is writable.

## Issues

Report application issues at https://github.com/ShAgGy2035/Ultimate-RCON-Server-Tool/issues/new.

## About

Open **Help > About** to view version `1.0.6`, developer information, and the detected application platform.

## Application Screenshots

### 1. Add a server

| Hosting and startup | Connection details |
| --- | --- |
| ![Add server wizard hosting and startup](docs/images/add-server-wizard-type.png) | ![Add server wizard connection details](docs/images/add-server-wizard-connection.png) |
| Review and installation | Saved servers |
| ![Add server wizard review and installation choices](docs/images/add-server-wizard-review.png) | ![Manage servers](docs/images/manage-servers.png) |

### 2. Configure the server

| Remote transport security | Integrations |
| --- | --- |
| ![SSH tunnel and trusted host fingerprint configuration](docs/images/transport-security.png) | ![Edit server integrations](docs/images/edit-server-integrations.png) |

**Plugin management**

![CounterStrikeSharp plugin management](docs/images/plugin-management.png)

### 3. Server commands and gameplay

| Console commands | Bot Control |
| --- | --- |
| ![Console command suggestions](docs/images/console-commands.png) | ![Bot Control](docs/images/bot-control.png) |
| Metamod controls | CounterStrikeSharp controls |
| ![Metamod controls](docs/images/settings-metamod-menu.png) | ![CounterStrikeSharp controls](docs/images/settings-counterstrikesharp-menu.png) |

**Fun Stuff modes**

![Fun Stuff modes](docs/images/fun-stuff-modes.png)

### 4. Schedule tasks

| Scheduled tasks | Add scheduled task |
| --- | --- |
| ![Scheduled tasks](docs/images/scheduled-tasks.png) | ![Add scheduled task](docs/images/add-scheduled-task.png) |

### 5. Chat and troubleshooting

| Chat | Debug output |
| --- | --- |
| ![Chat](docs/images/chat-tab.png) | ![Debug output](docs/images/debug-tab.png) |

### 6. Application preferences

| General settings | Backup settings |
| --- | --- |
| ![General settings](docs/images/settings-general.png) | ![Backup settings](docs/images/settings-backups.png) |
