<p align="center">
  <img src="docs/images/orke-mark.png" width="88" alt="orke logo">
</p>

<h1 align="center">orke for Linux</h1>

<p align="center">
  Your coding workspace on a Linux server.<br>
  Run terminals, AI coding agents and Git on the server, then work from a browser.
</p>

<p align="center">
  <a href="https://orke.bomtobe.com">Homepage</a> ·
  <a href="https://github.com/bomtobe-hgkim/orke-linux/releases">Linux downloads</a> ·
  <a href="https://github.com/bomtobe-hgkim/orke-mac">Mac downloads</a> ·
  <a href="https://orke.bomtobe.com/support/">Support</a>
</p>

This README takes you from a fresh Ubuntu server to a running workspace, then covers everyday commands, updates, removal and troubleshooting. Linux 3.4.8 supports interface language selection and uses English for default server and CLI output. Korean interface labels below include their English meanings.

> [!IMPORTANT]
> **Check which Linux build you have.** The current public download is **3.4.8, released on October 15, 2026**. It includes `orke update`, `orke uninstall`, `--purge`, service readiness checks, session/package control commands and the server browser. Existing 3.4.1 users should rerun the one-line installer below to upgrade; that older package has no `orke update` command. Updating a running service ends its active terminal and agent sessions, so run the upgrade from SSH when those sessions can be stopped.

## Contents

- [Requirements and version differences](#requirements-and-version-differences)
- [1. Install orke](#1-install-orke)
- [2. Connect your ngrok address](#2-connect-your-ngrok-address)
- [3. Start the background service](#3-start-the-background-service)
- [4. Open your workspace](#4-open-your-workspace)
- [Agents and accounts](#agents-and-accounts)
- [Everyday command reference](#everyday-command-reference)
- [Update or choose a version](#update-or-choose-a-version)
- [Uninstall and remove data](#uninstall-and-remove-data)
- [Service readiness and recovery](#service-readiness-and-recovery)
- [Session commands and server web windows](#session-commands-and-server-web-windows)
- [Use orke support from your own service](#use-orke-support-from-your-own-service)
- [Connect through SSH without ngrok](#connect-through-ssh-without-ngrok)
- [Files, logs and backups](#files-logs-and-backups)
- [Troubleshooting](#troubleshooting)
- [Download manually](#download-manually)
- [Validation and help](#validation-and-help)

## Requirements and version differences

| You need | Details |
| --- | --- |
| Ubuntu 24.04 LTS, **x86_64** | The distributed installer rejects ARM. Other distributions have not been validated. Check with `uname -m`; it must print `x86_64`. |
| systemd and SSH access | Use a normal VM or physical server with a systemd user service manager. Log in directly as the user who will run orke. |
| A regular user | Run orke and its installer without `sudo`. Use `sudo` only for the system-package commands shown below. |
| Internet access | Needed for downloads, ngrok and any AI providers you use. |
| An ngrok account for public browser access | Have its free static domain, authtoken and your allowed Google email ready. An [SSH tunnel](#connect-through-ssh-without-ngrok) is an alternative. |
| Agent accounts, if wanted | Ordinary terminals and Git work without an AI subscription. Agents need their own installation and provider access. |

The package includes its .NET runtime. **You do not need to build orke or install the .NET SDK.** Node.js/npm below are for one Codex installation method, not for running orke itself.

| Capability | Older Linux 3.4.1 | Current Linux 3.4.8 |
| --- | --- | --- |
| Terminal sessions, agents, Git, browser access to the workspace, orke support | Available | Available |
| `serve`, `remote setup`, service install/status/restart/removal | Available | Available |
| `orke update`, `orke uninstall`, `orke uninstall --purge` | Use the older-version alternatives in this README | Available |
| `service wait-ready` and retaining an older payload if an update starts but is not ready | Not included | Available |
| Session/package control CLI and server-hosted web windows | Not included in this archive | Available; web windows also depend on the host's Chromium sandbox configuration |

All commands in the numbered installation steps run **in your SSH session on the server**, unless a step explicitly says to use your own computer's browser.

## 1. Install orke

### Install the system packages

Run as your ordinary SSH user:

```bash
sudo apt-get update
sudo apt-get install -y libicu74 ca-certificates procps curl git tar gzip coreutils \
  xvfb libnss3 libgtk-3-0t64 libatk1.0-0t64 libatk-bridge2.0-0t64 \
  libcups2t64 libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 \
  libxrandr2 libgbm1 libasound2t64
```

These cover Linux 3.4.8, including its [browser dependencies](#session-commands-and-server-web-windows), even when you plan to use terminals only. If an installer reports missing packages, run the exact `apt-get` command it prints and retry.

### Run the installer

```bash
curl -fsSL https://orke.bomtobe.com/install.sh | bash
```

The script selects the newest published Linux package, downloads it from this repository, verifies its SHA-256 and installs it for your user:

| Item | Location |
| --- | --- |
| Program | `~/.local/share/orke` |
| Command | `~/.local/bin/orke` |
| Settings, account information and records | `~/.orke` |

Look for `orke … 을 설치했습니다` — **orke has been installed**. Installation alone does not set up public access or the background service; continue with the next steps.

### Make the command available in this SSH session

```bash
export PATH="$HOME/.local/bin:$PATH"
command -v orke
```

The second command should print the path to `~/.local/bin/orke`. On Ubuntu, logging out and back in normally picks up that directory once it exists. You can always use the full command, for example:

```bash
~/.local/bin/orke service --help
```

**Use the same user throughout this guide.** Settings created as a different user are a different orke installation's settings.

## 2. Connect your ngrok address

If you will use an SSH tunnel instead, skip this step, continue with **3. Start the background service**, then follow [Connect through SSH without ngrok](#connect-through-ssh-without-ngrok).

### Install ngrok through its apt repository

These are ngrok's [official Linux apt instructions](https://ngrok.com/download/linux):

```bash
curl -fsSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo 'deb https://ngrok-agent.s3.amazonaws.com bookworm main' \
  | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt-get update
sudo apt-get install -y ngrok
ngrok version
```

orke requires **ngrok 3.39.9 or later** and rejects the snap package. Install ngrok as above rather than using `snap install ngrok`.

### Save the connection settings

In the [ngrok dashboard](https://dashboard.ngrok.com), find your static domain under **Domains** and your token under **Your Authtoken**. Then run:

```bash
orke remote setup
```

| Prompt | Enter |
| --- | --- |
| `ngrok 고정 주소…` — ngrok static domain | Your assigned domain, such as `my-workspace.ngrok-free.app`. Use the hostname without a path. The supported endings are `.ngrok-free.app` and `.ngrok-free.dev`. |
| `이 주소를 다른 컴퓨터의 orke(예: PC)도 쓰나요? [y/N]` — does another computer use this address? | Press **Enter** for **No** when the server has its own domain. If another computer uses it, read the additional confirmation before continuing. |
| `접속을 허락할 본인 Google 이메일` — allowed Google email | The Google account you will use to enter the workspace. |
| `ngrok Authtoken…` | Paste your ngrok authtoken. Input is hidden; press Enter after pasting. |

One ngrok address can serve only one computer at a time. If another orke uses the same address, disconnect it there and turn off its startup auto-connect before starting the server's connection. The shared-address question is a warning/confirmation; it does not create a multi-computer sharing mode.

orke stores this connection itself. You do not need to separately run `ngrok http` or `ngrok config add-authtoken` for this setup.

<details>
<summary>Scripted setup using a token from a private file</summary>

Use this form when you already have an owner-readable file containing the token. Replace the example hostname, email and file path:

```bash
orke remote setup \
  --hostname my-workspace.ngrok-free.app \
  --email owner@example.com \
  --authtoken-stdin < "$HOME/private-ngrok-token.txt"
```

The token comes through standard input. There is no `--authtoken` value option. Setup does not restart an already running daemon; apply a saved change with `orke service restart` from SSH when its running sessions can be ended.

</details>

## 3. Start the background service

```bash
orke service install
orke service status
```

`install` creates **`orke.service`**, enables it for startup and starts it. It also tries to enable **linger** for your user, so the service can start at boot without an interactive login and continue after you log out.

If it prints `linger를 켜지 못했습니다` — **could not enable linger** — run this from the **same installing user's SSH session**, if that user has sudo rights:

```bash
sudo loginctl enable-linger "$(id -un)"
```

Otherwise ask an administrator to enable linger for **your installing Linux username**. An administrator using their own shell must supply your username, not their own `id -un` result.

Check the actual service and linger setting:

```bash
systemctl --user is-active orke.service
loginctl show-user "$(id -un)" -p Linger
```

Expected values are `active` and `Linger=yes`. `orke service status` provides diagnostics even if a service is stopped or missing; its exit code alone is not a readiness check.

**Linux 3.4.8:** `service install` and `service restart` also wait for the daemon to answer. The server web browser and ngrok have their own status, so a ready daemon does not guarantee either connection is ready. See [readiness and recovery](#service-readiness-and-recovery).

### Optional foreground test

If you want to test before installing a service:

```bash
orke serve
```

It stays attached to this SSH terminal. `UI 는 ngrok 주소로 여세요` means **open the UI at your ngrok address**. Press **Ctrl+C** to stop the foreground process before running `orke service install`. If the service is already running, use its status/restart commands instead of starting a second daemon.

## 4. Open your workspace

On your own computer, open the ngrok address you configured, for example `https://my-workspace.ngrok-free.app`.

1. On ngrok's free-domain notice page, choose **Visit Site** if it appears.
2. Sign in with the Google account you allowed during setup.
3. In orke, use the terminal button in the bottom **Dock** to open **새 세션** — **New session**.
4. Choose **Terminal** for an ordinary shell, or an installed agent; choose the server's **작업 폴더** — **Working folder**.
5. Select **시작** — **Start**.

The folder and every command belong to the **server**, even when you are viewing it from another computer or phone. The allowed Google account gets access to that user's orke workspace and files.

| In the interface | Meaning |
| --- | --- |
| `세션` | Sessions; find work that is still running |
| `확인 필요` | Needs your check, often waiting for input |
| `작업 중` | Working |
| `탭만 닫기 (세션 유지)` | Close only the tab; keep its terminal session |
| `세션 종료…` | End the session and its running process |
| `설정` / `계정` / `정보` | Settings / Accounts / About |

Closing a terminal tab or losing your browser connection keeps the terminal running while orke runs. Stopping/restarting orke or rebooting the server ends those processes. A phone browser can open the same address; use a computer browser for full settings and orke support administration.

### Git and packages in the workspace

Use **git 저장소 열기** — **Open Git repository** — to choose a repository on the server. Check `git --version` in a terminal; Git was included in the system-package step. You can use an existing checkout or clone your own repository into a server folder, then select it. Remote Git operations use that Linux user's Git/SSH authentication. The Git window provides changed files, diffs, history, staging, commit, fetch, pull and push.

Open **패키지** — **Packages** — for supported skills, commands, MCP configuration and package work. Hermes scheduled jobs are managed by Hermes; install/sign in to Hermes before using that section.

## Agents and accounts

Install agents as the **same Linux user** who runs orke. Ordinary Terminal sessions work before any agent is installed.

Host agents use the server user's OS permissions, and orke starts supported agents with permission prompts skipped. Choose their working folders and instructions with that access in mind.

### Claude Code

Install using [Anthropic's native installer](https://code.claude.com/docs/en/setup), then sign in:

```bash
curl -fsSL https://claude.ai/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"
claude --version
claude auth login
```

Open the authorization link on your own computer, complete sign-in and paste the returned code into the server terminal. Pasted text may be hidden; press Enter. The earlier server checks used Claude Code 2.1.292/2.1.295; the native installer follows the provider's current release and can update itself.

### ChatGPT through Codex

To reproduce the **Codex 0.160.1** setup tested with the public server package:

```bash
sudo apt-get install -y nodejs npm
sudo npm install -g @openai/codex@0.160.1
codex --version
codex login --device-auth
```

Enable device-code login in your [ChatGPT security settings](https://chatgpt.com/#settings/Security), then open the link shown by the command on your own computer and enter its one-time code. Follow the [official device-login instructions](https://developers.openai.com/codex/auth/) if the option is controlled by a workspace administrator.

<details>
<summary>Installing the current Codex release instead</summary>

OpenAI also provides a [standalone installer](https://github.com/openai/codex#installing-and-running-codex-cli):

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
```

This installs the provider's current release, not the pinned version above. Compatibility with an older orke package is separate. If orke support says it cannot disable required features in your Codex version, use the tested 0.160.1 setup or a newer compatible orke package.

</details>

### Other agent choices

These commands install the provider's current CLI. Run the command only for an agent you want, then start that CLI to complete its own sign-in:

| Agent | Install command | Start / sign in | Official instructions |
| --- | --- | --- | --- |
| OpenCode | `curl -fsSL https://opencode.ai/install \| bash` | `opencode` | [OpenCode](https://opencode.ai/docs/) |
| Hermes | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh \| bash` | `hermes` | [Hermes](https://hermes-agent.nousresearch.com/docs/getting-started/installation/) |
| Antigravity | `curl -fsSL https://antigravity.google/cli/install.sh \| bash` | `agy` | [Antigravity](https://www.antigravity.google/docs/cli/install/) |

Those agents are choices in orke, but their operation on the published Linux server package has not been validated. OpenCode's linked installer currently supplies its OpenCode 1 CLI; it can replace a different OpenCode installation. Each provider manages its own authentication, terms, usage limits and charges.

The package includes orke's NVIDIA agent. Its API key is set under **설정 → 계정 → NVIDIA API 키** — **Settings → Accounts → NVIDIA API key**. NVIDIA use on a Linux server has not been validated.

Claude/ChatGPT account management is under **설정 → 계정 → 계정 관리…** — **Settings → Accounts → Manage accounts…**. Switching the current account changes this environment's CLI login, including work that shares it; it is not a separate account selected independently for each terminal tab.

## Everyday command reference

Run service maintenance, updates and removal in **SSH**, outside orke's own terminal tabs.

| Task | Command | What to expect |
| --- | --- | --- |
| Show service diagnostics | `orke service status` | Service state, startup/linger settings, ngrok path and recent connection error |
| Install and start the service | `orke service install` | Registers the user service; an already running service should be managed with restart |
| Restart it | `orke service restart` | Ends sessions, then starts the service again |
| Stop it temporarily | `systemctl --user stop orke.service` | Keeps the installation and startup registration |
| Start it again | `systemctl --user start orke.service` | Starts the registered service |
| Stop automatic startup | `systemctl --user disable --now orke.service` | Stops it and disables startup; keeps its unit file |
| Restore automatic startup | `systemctl --user enable --now orke.service` | Enables and starts the existing unit |
| Remove only the service | `orke service uninstall` | Stops/removes the service; preserves the program and settings |
| Change ngrok settings | `orke remote setup` | Saves settings; restart a running daemon to apply them |
| Read service help | `orke service --help` | Supported service subcommands for the installed build |
| Read connection help | `orke remote --help` | Supported remote setup options |
| Latest service events | `journalctl --user -u orke.service -n 50 --no-pager` | Startup/failure and systemd events |
| Follow detailed logs | `tail -f ~/.orke/logs/daemon.log` | Live daemon/connection details; Ctrl+C stops only the log viewer |

**Linux 3.4.8** includes the following commands:

| Task | Command |
| --- | --- |
| Update to the latest published Linux version | `orke update` |
| Select a particular version | `orke update --version X.Y.Z` — replace `X.Y.Z` with a published version |
| Remove service, program and command; keep data | `orke uninstall` |
| Remove those items and orke's data | `orke uninstall --purge` |
| Check daemon readiness | `orke service wait-ready` |
| Allow up to 120 seconds for daemon readiness | `orke service wait-ready --timeout 120` |

### Optional port reservation

If `service install` suggests reserving ports, first inspect the existing list:

```bash
sysctl net.ipv4.ip_local_reserved_ports
```

An administrator can add orke's range, **47620–48643 plus 48700**, to that list. Preserve any existing reservations: a new value replaces the complete list. This reserves local ports; it is not a firewall rule or an instruction to expose them publicly.

## Update or choose a version

### Linux 3.4.8: update in one command

From SSH, when the running sessions can be stopped:

```bash
orke update
```

It selects the latest published Linux version, checks the downloaded package and uses orke's installer. Settings and records are kept.

**Upgrading from 3.4.4 with its browser active:** that version's CLI can mistake the service browser's application scope for an independent process and refuse before installation. Rerun the one-line installer from SSH when sessions can end to obtain 3.4.5 or later. Once 3.4.5 or later is installed, `orke update` and `orke uninstall` recognize the service browser; independent programs and processes surviving service stop still block replacement/removal.

- If the installed version already matches, it does nothing and does not restart sessions.
- An unversioned update does not downgrade an installed version newer than the latest public release.
- `--version X.Y.Z` explicitly selects a published version, including an older one. An exact version match is still a no-op.
- When a running service is replaced, its terminals/agents end and browser access briefly disconnects. New installers wait for daemon readiness before removing the previous payload.

As of October 15, 2026, all earlier Linux releases have been withdrawn, so `--version` can select only `3.4.8`. The one-line reinstall below can replace package bytes even when the version number matches.

### Older 3.4.1: rerun the installer

Use this route until you install a release containing `orke update`:

```bash
curl -fsSL https://orke.bomtobe.com/install.sh | bash
```

For the current version explicitly:

```bash
curl -fsSL https://orke.bomtobe.com/install.sh | bash -s -- --version 3.4.8
```

Rerunning the one-line installer can reinstall the same version and restart its running service. If a foreground `orke serve` is running, stop it with Ctrl+C in its own SSH terminal first.

## Uninstall and remove data

Choose whether you want to keep settings and records. **The new uninstall command does not ask for confirmation.** Run it as the installing user from SSH.

### Linux 3.4.8: remove the program and keep data

```bash
orke uninstall
```

This stops/removes the user service, deletes the installed program and its command link, and cleans up installer-owned leftover folders. It preserves the settings and records under the actual `ORKE_HOME`, normally `~/.orke`, so a later installation can reuse them.

### Linux 3.4.8: also erase orke's data

If you used **orke support**, first delete its rooms/AIs in orke while it is running and let their associated history cleanup finish. Purge refuses while support rooms, test rooms or rooms still being deleted remain.

```bash
orke uninstall --purge
```

This also deletes the settings directory used by the service. The installed service's home takes precedence over a different `ORKE_HOME` in your SSH shell; without a service unit, the shell override is used, otherwise `~/.orke`. A custom data directory must pass orke's path and contents checks; a user home, its parent, a symlink or an unrelated directory is not accepted as a purge target. If data is inside the program directory, ordinary uninstall refuses to silently delete it.

Both removal modes leave separately installed agents, ngrok, the user's linger setting and the shared ASP.NET data-protection key directory in place. Provider files outside orke's home, such as `~/.claude` and `~/.codex`, are separate from purging orke's data. orke support cleans its associated provider histories through its own room-deletion flow.

Agent help skills installed by orke outside its data home can also remain. Remove only files you identify as orke-owned and still want removed; preserve your other skills and any user-edited guides.

If removal reports that an installation is in progress or a standalone process is still running, finish that operation or stop that exact foreground process, then retry. Do not manually delete a folder being used by a running update.

<details>
<summary>Older 3.4.1: remove the service and files in order</summary>

The old package has no top-level `uninstall` command. Prepare **before removing the service or program**:

1. If you want associated support histories cleaned, delete support rooms/AIs through the running orke UI and let cleanup finish.
2. Run `orke service status` and record the actual settings location, especially if you used `ORKE_HOME`; removing the unit also removes its recorded override.
3. Stop any foreground `orke serve` with Ctrl+C in its own terminal. Removing a service alone does not stop a daemon that you started separately.

Then, in SSH, remove the service before deleting the program:

```bash
~/.local/bin/orke service uninstall && \
  rm -rf -- "$HOME/.local/share/orke" "$HOME/.local/bin/orke"
```

This keeps `~/.orke`. After any required orke support room cleanup, you may separately delete that directory if you want to erase the saved settings and records:

```bash
rm -rf -- "$HOME/.orke"
```

These paths assume the default data location. If you used `ORKE_HOME`, use the actual settings location you recorded before uninstalling, after inspecting its contents. Keep any work files you need. Do not delete your entire `~/.claude`, `~/.codex` or shared `~/.aspnet` directory as part of this guide.

</details>

## Service readiness and recovery

**Linux 3.4.8.** `service install` and `service restart` verify that the intended installed daemon answers; `wait-ready` performs the same check without installing or restarting it:

```bash
orke service wait-ready --timeout 120
```

The default daemon wait is 90 seconds; `--timeout` accepts 1–600 seconds. A separate server-browser startup/status check can take additional time. ngrok and browser availability are reported separately and are not conditions for daemon readiness.

If an update replaces files and starts the new service but readiness fails, the new installer **retains the old payload** under a run-specific `~/.local/share/.orke-previous.*/orke` directory. It prints the exact previous location and the commands to restore it. It does not automatically roll back after starting the replacement service.

1. Read the printed error and keep the retained directory.
2. Inspect the service and detailed log:

   ```bash
   orke service status
   journalctl --user -u orke.service -n 100 --no-pager
   tail -n 100 ~/.orke/logs/daemon.log
   ```

3. To restore the older payload, use the **exact stop/move/move/start commands printed by that installation**. Its folder names are specific to the run; do not substitute a wildcard.

| Result | Meaning |
| --- | --- |
| Installer exit 0 | Installation succeeded; a replaced running service passed the new readiness check |
| Installer exit 1 | Installation was refused, failed or restored before completion |
| Installer exit 3 | New files were placed, but the service did not become ready; the previous payload was retained |

The older 3.4.1 package does not provide these readiness/retention guarantees. The current package and website bootstrap preserve exit 3; read the printed recovery instructions if the service is not ready.

## Session commands and server web windows

**Linux 3.4.8.** The package includes a control CLI and a server-owned virtual display for web windows. These capabilities are absent from the older October 9 archive.

Useful read-only commands after the daemon starts:

```bash
orke help
orke status --json
orke session list --json
orke package list --json
```

`orke help` lists the installed build's management CLI syntax, including creating/sending to/waiting for/ending sessions and managing packages. Remote/service commands have their own help, as listed above. Session commands act on the same local daemon and server files; they are not commands for another computer's orke.

<details>
<summary>Example: create and inspect a command-line terminal session</summary>

This registers an ordinary shell without opening a new desktop window:

```bash
mkdir -p "$HOME/orke-work"
orke session create --runtime host --cwd "$HOME/orke-work" \
  --title 'README shell' --json
```

Copy `data.sessionId` from the response. Use that actual value below:

```bash
SESSION_ID='copy_the_returned_sessionId'
orke session status --id "$SESSION_ID" --json
orke session screen --id "$SESSION_ID" --json
```

`screen` provides the current terminal text. Use these command forms as needed:

| Task | Command |
| --- | --- |
| Send a command to a ready shell | `orke session send --id "$SESSION_ID" --text 'pwd' --submit true --json` |
| Wait for the session's native process to finish | `orke session wait --id "$SESSION_ID" --json` |
| Observe output | `orke session output --id "$SESSION_ID" --follow --json` |
| Explicitly end this session | `orke session end --id "$SESSION_ID" --json` |

Inspect the session before sending input. `--submit true` requests Enter; success confirms input submission, not completion of the command or AI task. If input is refused or its result is uncertain, inspect the screen/status before retrying. Output JSON carries encoded terminal-byte records; `screen` is simpler for reading current text. Ctrl+C stops a wait/output observer without ending the session.

</details>

### Browser dependencies and status

Linux 3.4.8 requires these additional Ubuntu packages:

```bash
sudo apt-get install -y xvfb libnss3 libgtk-3-0t64 libatk1.0-0t64 \
  libatk-bridge2.0-0t64 libcups2t64 libdrm2 libxkbcommon0 libxcomposite1 \
  libxdamage1 libxfixes3 libxrandr2 libgbm1 libasound2t64
```

Check actual availability with `orke status --json`: **`serverBrowser.state` is `ready` and `browserAvailable` is `true`** when the browser is available. Package installation and a running terminal service alone do not prove this.

Ubuntu's AppArmor/user-namespace policy can block Chromium's sandbox on a particular installation path. The status and daemon log explain the failure. An administrator must review that exact path and the [Ubuntu AppArmor guidance](https://documentation.ubuntu.com/security/security-features/privilege-restriction/apparmor/); the installer does not automatically change root ownership or setuid permissions.

To choose terminal-only operation on a build supporting server-browser settings:

```bash
orke setup server-browser get --json
orke setup server-browser --mode disabled --json
orke service restart
```

Restart ends running sessions and applies the saved mode. To request the virtual browser again, set `--mode virtual` and restart when work can be stopped. A browser failure can leave terminals and the daemon usable.

## Use orke support from your own service

orke support lets a trusted backend send HTTP questions to Claude or ChatGPT signed in on the server. Its listener is **separate from the browser UI**: the default is `http://127.0.0.1:48700`, not the ngrok address.

### Create an AI and access key

In a computer browser:

1. Open **orke support** from the Dock and select **첫 AI 만들기** / **새 AI** — **Create first AI / New AI**.
2. Enter a name, choose Claude or ChatGPT and add any instructions you want.
3. Leave **기본 도구 쓰기** — **Use built-in tools** — off to start. Leave **답 형식** — **Response format** — empty for a simple `answer` response.
4. Select **만들기** — **Create**. On **연결** — **Connection**, save the access key shown once.
5. Confirm the ingress indicator is **입구 열림** — **Ingress open**. The **시험** — **Test** — tab lets you try a question before connecting your service.

The key selects the AI. Keep it in your backend's configuration; replacing the key invalidates the previous one. orke support uses subscription sign-in; an API-key-only Codex login is not accepted.

### Send a first question

Run the following in the server's Bash/SSH terminal. Paste the copied access key into the hidden prompt:

```bash
read -rsp 'orke support access key: ' ORKE_KEY
printf '\n'
ORKE_SUPPORT='http://127.0.0.1:48700'

curl --fail-with-body -sS "$ORKE_SUPPORT/v1/health"
curl --fail-with-body -sS -X POST "$ORKE_SUPPORT/v1/rooms" \
  -H "Authorization: Bearer $ORKE_KEY" \
  -H 'Content-Type: application/json' \
  -d '{}'
```

The first response reports health. The second creates a room and returns **`room_id`**. Copy that value into the assignment below, replacing the example text:

```bash
ROOM_ID='r_copy_the_returned_room_id'

curl --fail-with-body -sS --max-time 360 \
  -X POST "$ORKE_SUPPORT/v1/rooms/$ROOM_ID/messages" \
  -H "Authorization: Bearer $ORKE_KEY" \
  -H 'Content-Type: application/json' \
  -H 'Idempotency-Key: readme-first-question' \
  -d '{"text":"Reply with Hello."}'
```

The request waits for the answer. A successful default response includes `status` and `output.answer`. Keep the room ID to continue that conversation. If your HTTP connection drops, the accepted question can continue; inspect the history before submitting another question:

```bash
curl --fail-with-body -sS "$ORKE_SUPPORT/v1/rooms/$ROOM_ID/messages" \
  -H "Authorization: Bearer $ORKE_KEY"
```

Use a new idempotency key for a new question. Retrying the same room, key and text refers to the same question; changing the text under that key produces a conflict.

| Endpoint | Purpose |
| --- | --- |
| `GET /v1/health` | Health; no access key required |
| `POST /v1/rooms` | Create a room for the key's AI |
| `POST /v1/rooms/{room_id}/messages` | Ask a question |
| `GET /v1/rooms/{room_id}/messages` | Read the conversation/results |
| `GET /v1/rooms/{room_id}/messages/{message_id}` | Read one question/result |
| `DELETE /v1/rooms/{room_id}` | Delete a room; associated cleanup continues afterward |

### Connect a backend on another computer

Use an HTTPS reverse proxy such as nginx or Caddy on the orke server, forwarding to `127.0.0.1:48700`. Open **입구 열림 → orke support 설정** for the generated proxy examples. Its **바깥 주소** — **External URL** — field updates connection examples; it does not install a proxy, configure DNS or obtain a certificate.

The default question limit is five minutes including queue time; nginx needs a suitably long `proxy_read_timeout` (the supplied example uses 360 seconds). Keep the access key on the backend rather than in visitor-facing JavaScript. Enabling Claude's built-in shell tools gives that AI the server user's OS permissions, so choose the tool setting to match the requests you accept.

## Connect through SSH without ngrok

This is an alternative for reaching the browser UI privately. Start orke first, then inspect its startup line on the **server**:

```bash
journalctl --user -u orke.service --no-pager | grep 'ssh -L'
```

On **your own computer**, run the printed SSH forwarding command with your real SSH username/server. For example, **only if the reported UI port is 47643**:

```bash
ssh -N -L 47643:127.0.0.1:47643 user@server
```

Keep that SSH connection open. Use the same local and server port; orke checks its own UI port. If that local port is occupied, free it or use the ngrok route.

In a second SSH connection, read the private address on the server:

```bash
sed -n 's/^url=//p' ~/.orke/run/ui.url
```

Open that complete `http://127.0.0.1:…` address in your own computer's browser. It contains a private access token. It changes when the daemon restarts; read it again after a restart. Use your actual data path if `ORKE_HOME` was customized.

## Files, logs and backups

| Location | Contents |
| --- | --- |
| `~/.local/share/orke` | Installed program; replace through the installer/update command |
| `~/.local/bin/orke` | Command link |
| `~/.config/systemd/user/orke.service` | User service definition |
| `~/.orke` | Default settings, accounts, remote-access settings and support records |
| `~/.orke/logs/daemon.log` | Detailed log; older rotated files are alongside it |
| `~/.orke/run/ui.url` | Private, current browser entry address |

`ORKE_HOME` can select another **absolute** data directory. Configure remote access and install the service with the same value; the service records it. `orke service status` shows the service's location. Changing only the variable in a later SSH shell does not move existing data or rewrite the installed service.

### Back up settings and records

When running work can be stopped, use SSH to stop the service before copying its data. For the default location:

```bash
systemctl --user stop orke.service && (
  umask 077
  tar -czf "$HOME/orke-data-$(date +%Y%m%d-%H%M%S).tar.gz" -C "$HOME" .orke
)
systemctl --user start orke.service
```

The archive command runs only if stopping the service succeeds. Check any archive errors before treating the file as a backup. Keep it private: it can contain account information, stored keys and conversation records. Use the actual configured directory for a custom `ORKE_HOME`. This is a data backup, not a snapshot of running terminals, working folders or the provider's separate `~/.claude`/`~/.codex` files.

## Troubleshooting

| Symptom | Check and next action |
| --- | --- |
| `orke: command not found` | Run `export PATH="$HOME/.local/bin:$PATH"`, or use `~/.local/bin/orke`. Log out/back in for Ubuntu's normal PATH setup. |
| Installer refuses root | SSH in as the ordinary user who will run the workspace; do not prefix orke/install commands with `sudo`. |
| Missing system package | Run the exact `apt-get` command printed by the installer, then retry. |
| Service commands cannot reach the user manager | SSH directly as the correct user. A shell entered with `su`/`sudo -u` may lack its user bus. Confirm this server runs systemd. |
| `orke is already running.` | Inspect `orke service status`. Do not run foreground `serve` alongside the service. |
| Service stops after logout | Check `loginctl show-user "$(id -un)" -p Linger`; enable linger as described above. |
| New `update` / `uninstall` / `wait-ready` command is unavailable | The public October 9 archive predates it. Use the older-version update/removal instructions until a release containing the command is installed. |
| Update/removal says the installation is busy | Let the other installation finish. If a standalone `serve` is listed, stop it in its own terminal. Retry from SSH. |
| `--purge` refuses because support rooms remain | Reopen orke, delete its support rooms/AIs and let cleanup finish before retrying. If the message says the service was stopped during its final check, follow its restart/recovery instructions first. |
| `--purge` refuses with `Contains entries not created by orke: employees` | Only the first 3.4.6 build, replaced on October 13, 2026, with a custom `ORKE_HOME`. Nothing is removed. Upgrade to 3.4.8 with `curl -fsSL https://orke.bomtobe.com/install.sh \| bash -s -- --version 3.4.8` or run `orke update`, then retry. |
| New installer replaced files but readiness failed | Preserve the printed previous-payload directory; inspect logs and follow that run's exact restoration commands. |
| ngrok came from snap | Install ngrok from the apt instructions in this README; rerun `orke remote setup` to pin the correct executable. |
| ngrok error 334 / another orke uses this address | Disconnect the other computer's tunnel, then restart this service when work can be stopped. |
| ngrok error 105/107 / authentication failed | Rerun `orke remote setup` with the valid authtoken, then restart. These failures do not retry indefinitely. |
| Agent command is missing in orke | Verify `command -v claude` / `command -v codex` in the same user's login shell. Correct PATH/install location; restart orke if its environment still cannot find the tool. |
| Codex is signed in, but support rejects its version or authentication | Use ChatGPT subscription sign-in and a CLI version compatible with the installed orke package; 0.160.1 was tested with the public server package. |
| Browser says `연결하지 못했습니다` | Select **다시 연결** — **Reconnect**. Check `orke service status` and the daemon log if it persists. |
| New server web browser is unavailable | Inspect `orke status --json` and the detailed log. Daemon readiness does not validate the Chromium sandbox; terminal-only mode is available on the new builds. |
| Linux 3.4.4 `update` refuses with an Electron PID while the virtual browser is ready | Fixed in **3.4.5**: the CLI recognizes the service's browser even after Chromium moves it into an application scope. To get past the old CLI's refusal, rerun the one-line installer from SSH when sessions can end; it stops the service and checks for remaining processes before replacement. The refusal itself changes no files. Administrator sandbox setup may need to be reapplied after replacement. Separate standalone programs and processes surviving service stop still block update/removal. |

When asking for help, include the version from **설정 → 정보** — **Settings → About**, your Ubuntu version, architecture, the failed command and its error. Remove access tokens, keys and private UI URLs from any shared log.

## Download manually

| Version | Package | Checksum | Original release |
| --- | --- | --- | --- |
| **3.4.8** | [Linux x64 tar.gz](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.8/orke-3.4.8-linux-x64.tar.gz) | [SHA-256](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.8/orke-3.4.8-linux-x64.tar.gz.sha256) | October 15, 2026; phone and tablet screens, AI employee and repeat-task controls, authoring conversations, worker browser-tab cleanup, terminal glyph and Korean input fixes |

As of October 15, 2026, 3.4.8 is the only published Linux package; all earlier packages have been withdrawn.

For manual 3.4.8 installation, run in an empty download directory in SSH:

```bash
mkdir -p "$HOME/orke-downloads/3.4.8"
cd "$HOME/orke-downloads/3.4.8"
curl -fLO https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.8/orke-3.4.8-linux-x64.tar.gz
curl -fLO https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.8/orke-3.4.8-linux-x64.tar.gz.sha256
sha256sum -c orke-3.4.8-linux-x64.tar.gz.sha256 && \
  tar xzf orke-3.4.8-linux-x64.tar.gz && \
  ./orke-3.4.8-linux-x64/install.sh
```

Continue with ngrok/service setup above after the installer succeeds. The checksum must report `OK`; a failed check stops the `&&` chain. You can remove the downloaded archive/extracted package after installation; run the installed `~/.local/bin/orke` for service commands.

## Validation and help

Linux **3.4.8** was built and checked on Ubuntu 24.04.5 x86_64 from product source `c8a07c32ae664a8b757541ff725594456e4c6042`. The Release build succeeded; the ordinary .NET gate passed 21,632 tests with 377 skipped and no failures. All 19 Linux-specific checks, 13 slow JavaScript checks and 6 actual-Electron checks passed. Packaging verified the CLI distribution, native runtime files and Electron 44.4.3 payload. The 188,024,047-byte archive has SHA-256 `001a53ba6ce84379d52381408f2afa60dd31bd616c2a8b8306877e51522d69b3`. GUI, real-provider questions and external sign-in flows were not retested for this release. Earlier measurements below remain historical.

Linux **3.4.6** (the build republished on October 13, 2026 with the `--purge` fix) was checked on Ubuntu 24.04.5 x86_64. The ordinary gates passed on Windows (21,693 tests, on the fix branch) and Ubuntu (21,316 passed; one support-shutdown timing check failed once in the full run and passed five repeated runs alone), alongside 19 Linux-specific checks on Ubuntu and 13 slow JavaScript and 5 actual-Electron checks on each platform. The package replaced a running 3.4.5 user service, kept its data, answered management commands, ran a shell session through the CLI, served the UI document and 13 critical assets identical to the package, and reached a ready virtual browser after an administrator configured that installation's SUID sandbox helper. `orke uninstall --purge` removed a custom `ORKE_HOME` that held the `employees` folder, which the first 3.4.6 build refused, and the default `~/.orke`. After publication, a fresh one-line installation installed this build's program, `orke update` changed nothing, and `--purge` removed the custom `ORKE_HOME`. Korean terminal input with IBus Hangul was measured separately in Electron on the same VM. GUI and real-provider/ngrok acceptance were not repeated for this release.

Linux 3.4.5 was checked on Ubuntu 24.04.5 x86_64 with a ready virtual browser. The old 3.4.4 CLI refusal was reproduced; the fixed CLI's same-version update returned 0 and preserved the daemon PID. Actual replacement, an installed 3.4.5 no-op, removal with the browser active and retained data passed. A separate executable running from the installation still blocked update. The ordinary gates passed on Windows (21,390 tests) and Ubuntu (21,032), alongside 19 Linux-specific checks, 13 slow JavaScript checks and 5 actual-Electron checks per platform. GUI and real-provider/ngrok acceptance were not repeated for this correction.

Linux 3.4.4 was checked on Ubuntu 24.04.5 x86_64. The packaged program upgraded an actual 3.4.1 user service, kept its data, answered management commands and ran a shell session. The UI document and 13 critical assets were served successfully and matched the package bytes. The virtual browser reached `ready` after an administrator configured the exact installation's SUID sandbox helper; this does not mean its GUI, IME or external sign-in flows were visually accepted. The installer never applies that administrator change automatically.

The public 3.4.0 package was exercised on one Ubuntu 24.04.5 x86_64 Google Cloud VM for installation, services, ngrok, agents and support. Public 3.4.1 specifically rechecked upgrading and the ChatGPT tab-state fix. Those provider and ngrok measurements remain historical; they were not repeated with real provider questions for 3.4.4.

The [earlier Linux installation guide](docs/linux.md#what-has-been-tested) preserves the older package's measured scope. Those older packages are no longer published.

[Homepage](https://orke.bomtobe.com) · [Support & FAQ](https://orke.bomtobe.com/support/) · [Mac downloads](https://github.com/bomtobe-hgkim/orke-mac) · [Windows Store](https://apps.microsoft.com/detail/9N7G0NK63QGR) · [Contact: bomtobe.sw@gmail.com](mailto:bomtobe.sw@gmail.com)
