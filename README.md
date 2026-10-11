<p align="center">
  <img src="docs/images/orke-mark.png" width="88" alt="orke logo">
</p>

<h1 align="center">orke for Linux</h1>

<p align="center">
  Run your AI coding workspace on a Linux server and use it from a browser.
</p>

<p align="center">
  <a href="https://orke.bomtobe.com">Homepage</a> ·
  <a href="docs/linux.md">Install guide</a> ·
  <a href="https://github.com/bomtobe-hgkim/orke-linux/releases">Linux releases</a> ·
  <a href="https://github.com/bomtobe-hgkim/orke-mac">macOS downloads</a>
</p>

This repository contains **Linux server downloads and installation instructions**. The orke interface and command-line messages are in Korean; the [English install guide](docs/linux.md) explains the labels you will see.

## Install

**Requirements:** Ubuntu 24.04 LTS, x86_64, systemd and a regular user account. Other distributions and ARM servers have not been tested. No screen is needed.

Run over SSH as the user who will use orke:

```bash
curl -fsSL https://orke.bomtobe.com/install.sh | bash
```

It downloads the newest Linux package from this repository, checks its SHA-256 and installs into `~/.local/share/orke`, with the command at `~/.local/bin/orke`. Settings stay in `~/.orke`.

Then install ngrok, connect your own ngrok address with `orke remote setup`, and run `orke service install` to keep orke running after logout and reboot. Open your address in a browser and sign in with the Google account you allowed.

**[Follow the full install guide](docs/linux.md)** for system packages, ngrok, sign-in, automatic startup, updates, uninstalling and troubleshooting.

## Download manually

| Version | Package | Checksum | Notes |
| --- | --- | --- | --- |
| **3.4.1** | [Linux x64 tar.gz](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.1/orke-3.4.1-linux-x64.tar.gz) | [SHA-256](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.1/orke-3.4.1-linux-x64.tar.gz.sha256) | Fixes a ChatGPT tab that kept showing working after Codex answered. |
| 3.4.0 | [Linux x64 tar.gz](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.0/orke-3.4.0-linux-x64.tar.gz) | [SHA-256](https://github.com/bomtobe-hgkim/orke-linux/releases/download/v3.4.0/orke-3.4.0-linux-x64.tar.gz.sha256) | First Linux server package. |

Both packages were originally published on October 9, 2026. They moved here on October 11, 2026 with their original filenames and bytes.

To check a download, put its `.sha256` file in the same folder and run:

```bash
sha256sum -c orke-3.4.1-linux-x64.tar.gz.sha256
```

## Agents and accounts

Install the coding agents you want on the server and sign in with your own accounts. Ordinary terminals work without an AI account. orke includes its NVIDIA agent, but that agent has not been tested on a Linux server.

The [guide's agent setup section](docs/linux.md#7-install-and-sign-in-to-claude-code-and-codex) covers Claude Code and Codex. Other providers supply their own installers:

```bash
curl -fsSL https://opencode.ai/install | bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
```

OpenCode, Hermes and Antigravity have not been tested on the published server package. Each provider's terms, usage limits and charges apply.

## Update and uninstall

Run the install command again to update. **An update restarts orke and ends running terminal sessions, agents and orke support questions.** Run it from SSH when those sessions can be stopped.

For uninstalling, removing the service and handling saved data, follow the [uninstall instructions](docs/linux.md#uninstall).

## Other platforms and help

- [macOS downloads](https://github.com/bomtobe-hgkim/orke-mac)
- [Windows — Microsoft Store](https://apps.microsoft.com/detail/9N7G0NK63QGR)
- [Homepage](https://orke.bomtobe.com) · [Support](https://orke.bomtobe.com/support/)
- [What has been tested on Linux](docs/linux.md#what-has-been-tested)
- Contact: [bomtobe.sw@gmail.com](mailto:bomtobe.sw@gmail.com)
