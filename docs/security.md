# Security baseline and deployment checks

This document distinguishes source-host observations from optional hardening steps for a new VPS. Updating this repository does not change the live host's firewall, SSH, tailnet policy, secrets, or running services.

## Verified observations

- Tailscale was online; Tailscale SSH and automatic client updates were enabled. Local Serve/Funnel configuration was empty.
- The dashboard listened on its Tailscale IPv4 address at TCP 9119. Username, password, and session-signing-secret environment variables were configured; values are not exported.
- Unauthenticated requests to `/api/config` and `/api/config/raw` returned HTTP 401. The root/login page returned HTTP 200; public login UI does not by itself bypass the API gate. Credentialed login, every API endpoint, and cross-device policy enforcement were not tested.
- Hermes configuration had `gateway.allow_all_users: false`. Telegram allow-list and routing identities are intentionally omitted and must be recreated locally.
- One readable SSH drop-in contained `PasswordAuthentication no`. Another drop-in was unreadable, so effective sshd policy could not be checked. This is not proof that password authentication is disabled in the effective configuration.
- Wildcard TCP listeners existed on ports 22, 80, and 443. A listener is not evidence of external reachability: owner, firewall/cloud rules, and purpose were not verified. Do not stop/delete them without identifying the services.

## Unverified: do not treat as hardened

The export account lacked passwordless sudo. UFW status and effective `sshd -T` could not be read. Cloud firewall policy, tailnet grants/ACLs, account MFA, device approval/expiry policy, Internet reachability, and SSH authentication behavior were not verified. Docker socket access was denied, so container workloads, published-port ownership, and Docker firewall behavior are not inventoried. No SSH/firewall or root-owned configuration files were copied.

## Local credentials and Hermes settings

- Use an independent provider login on the destination. Never commit `.env`, `auth.json`, GitHub credential stores, SSH keys, Tailscale node state, or browser sessions.
- Preserve `.env` created by setup; use `config/.env.example` only as an empty checklist. Configure a unique dashboard username, strong password, and independent high-entropy signing secret locally. Do not reuse a bot token as a dashboard secret. Prefer the dashboard's supported password-hash configuration when available; follow the installed version's documentation.
- Protect local files. On a new default-profile install, after confirming ownership and file locations:

  ```bash
  chmod 700 "$HOME/.hermes"
  chmod 600 "$HOME/.hermes/.env"
  # Only if this file exists:
  chmod 600 "$HOME/.hermes/auth.json"
  ```

  These are suggested destination permissions, not a claim that source permissions were audited.

- Explicit optional baseline commands, using Hermes' supported writer rather than editing live YAML:

  ```bash
  hermes config set security.redact_secrets true
  hermes config set approvals.mode smart
  hermes config set gateway.allow_all_users false
  ```

  Start a fresh session/restart the relevant service for settings that are loaded at startup. Secret redaction and approvals reduce risk; neither makes arbitrary tool execution safe. Do not switch on `allow_all_users` or disable approvals just to bypass setup errors.

## SSH and firewall: lockout-safe order

1. Keep your VPS provider console available, preserve an existing administrative session, and confirm a separate new Tailscale SSH session works.
2. Identify services using ports 22/80/443 and every required public service before changing rules. Preserve any required web traffic; do not assume those listeners belong to Hermes.
3. Ask an administrator to inspect the effective sshd configuration (`sudo sshd -T`), UFW rules (`sudo ufw status verbose` and `sudo ufw status numbered`), and the provider firewall. Confirm intended key-based/OpenSSH fallback access separately from Tailscale SSH, which uses its own policy.
4. Restrict tailnet access through reviewed grants and SSH rules; the example policy is not the live policy. Verify unauthorized-device denial.
5. If using UFW, review interface-scoped rules appropriate to the chosen dashboard mode. Examples for a new host, not commands run by this export:

   ```bash
   sudo ufw allow in on tailscale0 to any port 22 proto tcp
   # Only for direct tailnet-bound dashboard mode:
   sudo ufw allow in on tailscale0 to any port 9119 proto tcp
   ```

   Rules alone do not enable UFW or remove existing broad allows. Have the administrator plan default policy, necessary public-service exceptions, IPv6 coverage, and cloud rules, then validate access before removing public SSH rules or enabling/reloading a firewall. Do not run `ufw reset`, delete numbered rules blindly, or assume a port-22 listener means public SSH is permitted.
6. Re-test allowed and denied access from independent devices, plus provider-console recovery. Record actual results before calling the host hardened.

Docker-published ports can bypass UFW's normal filtering path. Audit Docker publishing and its supported filtering controls separately; UFW status alone cannot establish that containers are private.

## Dashboard deployment choices

Use localhost + a Tailscale SSH tunnel by default. A separate optional template preserves direct Tailscale-IP binding without exporting the source address. Both require local authentication configuration. Do not enable public Funnel or expose dashboard port 9119 in the provider firewall. See `tailscale.md` and `../systemd/README.md`.

## Repository publication checks

Before each push, inspect the full diff, validate JSON/YAML/templates, and check new content against known local secret values without printing those values. Exclude identity-bearing raw Tailscale debug/status dumps. After pushing, read back GitHub private visibility, branch commit, and changed files; a successful push alone is insufficient. Do not describe a fresh clone or static template check as a fresh-server deployment test.

## Official references

- Hermes configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- Tailscale SSH: https://tailscale.com/kb/1193/tailscale-ssh
- Ubuntu firewall walkthrough: https://tailscale.com/kb/1077/secure-server-ubuntu
- Docker firewall/UFW behavior: https://docs.docker.com/engine/network/packet-filtering-firewalls/
