# Hermes + VPS reusable setup

Private reusable setup for **vteck101**. This is a configuration/source reference, not a server image or a backup of account access.

## Included

- Sanitized Hermes settings and generated CLI configuration commands.
- Empty environment-variable template (names only).
- Exact upstream source reference through the `hermes-agent` Git submodule, including the bundled skills and package lockfiles.
- Portable optional localhost-only dashboard systemd unit; gateway unit regenerated on the destination.
- Observed Ubuntu/package inventory and deployment notes.

The existing installation had no source changes, customized skills, custom plugins/hooks, or modified SOUL file. Those are preserved by the upstream commit rather than duplicated.

## Excluded

API keys, OAuth/login tokens, GitHub credentials, Telegram credentials/user IDs/channel IDs, SSH keys, Tailscale identity, personal memories, conversations, databases, pairing state, logs, cron jobs, workspaces, backups, machine IP addresses, and browser sessions. Do not copy the old `.env` or `auth.json` into Git.

## Recreate on a fresh VPS

1. Start with Ubuntu 24.04 x86_64 and a non-root account. Install prerequisites:

   ```bash
   sudo apt update
   sudo apt install -y git curl ca-certificates
   ```

2. Authenticate to GitHub on that VPS, then clone this private repository. The submodule is public; recursive checkout includes the exact source snapshot:

   ```bash
   git clone --recurse-submodules https://github.com/vteck101/hermes-vps-setup.git
   cd hermes-vps-setup
   ```

3. Install Hermes using the official installer (review it before running):

   ```bash
   curl -fsSL https://hermes-agent.nousresearch.com/install.sh -o "$HOME/hermes-install.sh"
   less "$HOME/hermes-install.sh"
   bash "$HOME/hermes-install.sh"
   ```

   This normal installer follows the current release; it does **not** promise to install the archived commit. The submodule preserves the original source and PM lockfiles for reference or a separately reviewed pinned-source deployment. Do not overwrite an existing working installation with the submodule.

4. Open a new shell if needed, then configure provider credentials locally:

   ```bash
   hermes setup
   hermes model
   mkdir -p "$HOME/workspace"
   ```

   The saved model/provider is `gpt-6.1-sol` / `openai-codex`. Account access and model availability must be recreated separately. If unavailable, choose an accessible model and omit the saved `model` command below.

5. Review `config/config.example.yaml` and `config/apply-settings.txt`. Apply desired commands from the latter using `hermes config set`; it contains the complete sanitized settings grouped by top-level section. It replaces those sections, so use it only for a new installation. Do not run it against a working deployment without reviewing differences. The final command sets the new account's workspace path.

6. Use `config/.env.example` only as a checklist. Fill needed secrets locally, with restrictive permissions; do not replace a `.env` created by setup. Recreate Telegram bot credentials, allowed-user IDs, and home-channel routing for the new deployment. Leave unauthorized access disabled. Disable Telegram until configured if it is not needed.

7. Set up messaging and generate the gateway unit:

   ```bash
   hermes gateway setup
   hermes gateway install
   ```

   Follow the installed CLI's service prompts. For the optional dashboard, see `systemd/README.md`. Its safer localhost binding intentionally differs from the old private-network binding.

8. Validate on the new VPS:

   ```bash
   hermes doctor
   hermes config check
   systemctl --user status hermes-gateway.service
   ```

   Send a test message from an allowed Telegram account and verify the response. Test the dashboard through your tunnel if enabled.

## VPS-specific services

The source VPS had Docker, Tailscale, and SSH active. `inventory/packages.tsv` records selected observed versions, not a complete OS lockfile. Install those services only if needed using their official instructions. Reauthenticate Tailscale and recreate SSH/firewall policy locally; private keys and machine identity are intentionally not included. Docker container inventory could not be read due to socket permissions, so this repository does not reproduce Docker workloads or volumes. No firewall, network, or system-service configuration has been copied or changed.

## Verification and limitations

The export is checked for tracked credential/state files, known live secrets and personal routing values, configuration parsing, generated settings consistency, and source commit availability. The GitHub repository is checked after upload for private visibility, file tree, and commit identity. A full fresh-VPS install is **not** performed on the live server; installation, login, systemd startup, and messaging must still be verified on the destination.

Upstream source retains its own license. Official documentation: https://hermes-agent.nousresearch.com/docs/
