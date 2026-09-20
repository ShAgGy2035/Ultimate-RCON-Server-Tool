# Ultimate CS2 RCON and Server Management Tool - Linux Guide

Version: 1.0.4

This guide covers the Linux x64 application and Linux CS2 server profiles. For Windows server setup, including WinSW, see [README-WINDOWS.md](README-WINDOWS.md). For shared features and project information, see [README.md](README.md).

- **Linux application**
  - [Install and launch](#install-and-launch)
  - [Update the application](#update-the-application)
  - [Configuration and data](#configuration-and-data)
   - [PlayerPunishments setup](#playerpunishments-setup)
- **Local Linux server**
  - [Choose a lifecycle mode](#choose-a-local-lifecycle-mode)
  - [Direct process](#local-linux-direct-process)
  - [Service commands](#local-linux-service-commands)
- **Remote Linux server**
  - [Requirements](#remote-linux-requirements)
  - [Direct process](#remote-linux-direct-process)
  - [SSH and file permissions](#remote-linux-ssh-and-file-permissions)
  - [Service commands with systemd](#linux-service-commands-with-systemd)
- [SteamCMD behavior](#steamcmd-behavior)
- [Troubleshooting](#troubleshooting)

## Install And Launch

Extract the Linux package and open its directory:

```bash
tar -xzf cs2-rcon-tool-universal-v1.0.4-linux-x64.tar.gz
cd cs2-rcon-tool-universal-v1.0.4-linux-x64
```

Launch through the included script:

```bash
./run.sh
```

Alternatively, mark `run.sh` as executable in your file manager and select **Run as a Program**. Use `./run.sh --foreground` to keep the process attached for troubleshooting.

The package is self-contained and does not require a separate .NET installation. The launcher uses an app-local `.net` directory for extracted native single-file libraries. It is a cache and can be deleted while the application is closed.

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

### PlayerPunishments Setup

The app's Slay and Slap controls send `css_slay #userid` and `css_slap #userid damage`. Install [PlayerPunishments 1.0.0 or newer](https://github.com/ShAgGy2035/PlayerPunishments) on each managed CS2 server when its existing administration package does not provide those commands. Configure Slap damage from `0` to `99` under **Settings > General**.

PlayerPunishments is a CounterStrikeSharp server plugin. Install it in the Linux CS2 server's CounterStrikeSharp plugin directory, not in the desktop application's folder. Alive and Health display **Unknown** when the required server-side player-information command is unavailable.

## Local Linux Server

Use a **Local Linux** profile only when the application and CS2 server run on the same Linux computer.

Configure the CS2 install directory, executable, working directory, SteamCMD path, App ID `730`, startup map, game type, game mode, maximum players, tickrate, VAC state, LAN mode, Startup CFG, and optional player password. Public internet servers can also use a Steam Web API key and game-server login token. These credentials are not required or prompted for when **LAN server** is enabled.

### Choose A Local Lifecycle Mode

- **Direct process**: the application builds the CS2 launch command and owns the process.
- **Service commands**: an existing systemd service owns CS2 and the application runs configured shell commands.

Stop the current owner before switching modes. systemd cannot adopt a CS2 process that was already launched directly.

### Local Linux Direct Process

Use this mode when the application should launch and stop the executable itself.

Typical values:

```text
Executable:        <install-dir>/game/bin/linuxsteamrt64/cs2
Working directory: <install-dir>/game
Arguments:         -dedicated +ip 0.0.0.0
```

The application adds authoritative map, game type, game mode, CFG, port, hostname, RCON, tickrate, VAC, LAN, Steam API, GSLT, and optional player-password arguments. An ordinary Start runs SteamCMD validation when SteamCMD and install-directory settings are configured. Restart does not run SteamCMD validation.

On supported Linux systems, the application can check SteamCMD prerequisites and bootstrap SteamCMD. Local profiles can also enable startup maintenance to warn players, stop CS2, update the server and supported add-ons, and restart it.

### Local Linux Service Commands

Use this mode when an existing systemd service should own CS2. The application does not create the unit or sudoers policy.

Default commands target `cs2-server`:

```bash
sudo -n systemctl start cs2-server
sudo -n systemctl stop cs2-server
sudo -n systemctl restart cs2-server
```

The account running the application needs noninteractive sudo permission for these commands. Put all launch arguments, including `-tickrate`, `+sv_lan`, and `-secure` or `-insecure`, in the systemd launcher. Service Start and Restart do not run SteamCMD validation.

Use the complete setup under [Linux Service Commands With systemd](#linux-service-commands-with-systemd). For a local profile, replace `<ssh-user>` in its sudoers example with the desktop account running the application.

## Remote Linux Server

Use a **Remote Linux** profile when CS2 runs on another Linux computer.

### Remote Linux Requirements

Configure:

- SSH host, port, username, and password.
- Optional SFTP address, username, password, and port. Blank values reuse SSH details; port `0` reuses the SSH port.
- Remote SteamCMD path and CS2 installation directory for managed updates.
- App ID `730` and Steam login, normally `anonymous`.
- The lifecycle mode and its corresponding fields.

The remote account must run lifecycle commands and write to the installation directory. Remote browsing and file operations require SFTP access.

### Remote Linux Direct Process

Direct process mode runs CS2 without a systemd service. It is appropriate when the SSH account owns the installation and the application should manage the executable.

1. Select **Remote Linux (SSH)** and set **Startup mode** to **Direct process**.
2. Enter the install directory, for example `/home/cs2/cs2server`. Blank fields derive:

   ```text
   Executable:  /home/cs2/cs2server/game/bin/linuxsteamrt64/cs2
   Working dir: /home/cs2/cs2server/game
   ```

3. Set the startup map, game type, and game mode. Put only optional switches in **Additional arguments**. The application supplies `-dedicated`, `-console`, port, hostname, `+map`, `+game_type`, `+game_mode`, `+exec`, `-usercon`, RCON password, tickrate, LAN mode, and VAC state. The default `+ip 0.0.0.0` permits remote connections.
4. Enter a path-specific Stop command. For the example above:

   ```bash
   executable='/home/cs2/cs2server/game/bin/linuxsteamrt64/cs2'; for process in /proc/[0-9]*; do target=$(readlink "$process/exe" 2>/dev/null || true); if [ "$target" = "$executable" ]; then kill -TERM "${process##*/}"; fi; done
   ```

5. Optionally enter the Steam API authentication key and GSLT. They are appended at launch and redacted from output.
6. Save the profile and use **Start**, **Stop**, or **Restart**.

Start refuses to run while SteamCMD is updating app `730`, verifies `game/csgo/gameinfo.gi`, supplies required Linux native-library paths, launches CS2 detached from SSH, and monitors it for 30 seconds. Startup output is captured in `/tmp/cs2-rcon-tool-startup-<port>.log`. Stop verifies that the exact executable exited.

Direct mode refuses to control an executable owned by the active `cs2-server` systemd cgroup. It does not start CS2 at boot or restart it after a crash.

### Remote Linux SSH And File Permissions

The simplest setup uses the Linux account that owns the CS2 installation for both SSH and SFTP. Leave separate SFTP fields blank to reuse the SSH credentials.

A different account must traverse every parent directory and read and write the installation. Replace placeholders before running these checks:

```bash
namei -l "<install-dir>/game/csgo"
sudo -u <ssh-user> test -r "<install-dir>/game/csgo/gameinfo.gi"
sudo -u <ssh-user> touch "<install-dir>/game/csgo/.rcon-tool-write-test"
sudo -u <ssh-user> rm "<install-dir>/game/csgo/.rcon-tool-write-test"
```

Prefer a dedicated shared group or filesystem ACL instead of world-writable permissions. One ACL example is:

```bash
sudo setfacl -m u:<ssh-user>:x "<parent-directory>"
sudo setfacl -R -m u:<ssh-user>:rwX "<install-dir>"
sudo setfacl -R -d -m u:<ssh-user>:rwX "<install-dir>"
```

The first command-line SSH connection may ask you to trust the host fingerprint. The application uses its own SSH library and does not depend on the command-line client's `known_hosts` entry.

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
   -dedicated +ip 0.0.0.0 -high -nobreakpad -nomemorystats -nojoy \
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

## SteamCMD Behavior

| Linux profile and mode | Start | Restart |
| --- | --- | --- |
| Local Linux, Direct process | Runs validation when SteamCMD and install-directory settings are configured | Does not run validation |
| Local Linux, Service commands | Runs the configured Start command without validation | Runs the configured Restart command without validation |
| Remote Linux, Direct process | Checks launch ownership and monitors startup without validation | Stops and starts without validation |
| Remote Linux, Service commands | Checks SteamCMD is idle, verifies the core file, and monitors the service process without validation | Performs the same preflight and monitoring without validation |

Use **Validate server files** or **Install/Update Server** for explicit SteamCMD validation.

## Troubleshooting

- Verify the executable, working directory, SteamCMD path, install directory, and profile OS.
- For a bridged virtual machine, use the guest's LAN address rather than the host address or `127.0.0.1`.
- Keep `+ip 0.0.0.0` in Direct-process arguments for remote access. Put it in the systemd launcher for Service commands mode.
- Verify listening sockets with `ss -lntup | grep 27015`, replacing the port as needed.
- `Connection refused` means no SSH service accepted the configured host and port.
- Authentication errors mean the SSH username or password was rejected.
- `Permission denied` after login usually means the account cannot traverse a parent directory or access the installation. Run the checks under [Remote Linux SSH And File Permissions](#remote-linux-ssh-and-file-permissions).
- Confirm lifecycle commands do not require an interactive privilege prompt.
- Remote archive operations require `tar` and either `unzip` or `python3` for ZIP files.
- Framework and SteamCMD downloads require `curl`, `wget`, or `python3`; the application uses the first available option.
- Do not run Direct process and Service commands modes against the same executable and port.
