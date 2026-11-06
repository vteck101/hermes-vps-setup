# Tailscale setup and private dashboard access

## Observed source deployment

The client was online with Tailscale SSH enabled, automatic client updates enabled, Tailscale DNS accepted, and other devices' subnet routes not accepted. No exit node or subnet-route advertisement was configured. Tailscale Serve/Funnel had no local configuration. The dashboard listened on this machine's Tailscale IPv4 address, port 9119, not a public wildcard address.

The export intentionally omits the tailnet identity, device name, addresses, node keys, authentication URLs, account email, and Tailscale state directory. See `inventory/security.json` for sanitized observations and verification limits. The local client's preferences do not prove the tailnet access policy is restrictive.

## New Ubuntu VPS

Use a new non-root account with sudo access. Keep provider-console access and an existing working administrative session before changing any remote-access setting. The following is a manual deployment guide; none of these network changes were applied to the source VPS during export.

1. Install the official client, reviewing the installer first:

   ```bash
   curl -fsSL https://tailscale.com/install.sh -o "$HOME/tailscale-install.sh"
   less "$HOME/tailscale-install.sh"
   sudo sh "$HOME/tailscale-install.sh"
   sudo systemctl enable --now tailscaled
   sudo tailscale up
   ```

   Open the authentication URL locally and authorize this new device. Never save that URL, an auth key, or `/var/lib/tailscale/` in this repository.

2. On a new dedicated host, configure the source deployment's relevant preferences:

   ```bash
   sudo tailscale set --ssh=true --auto-update=true --accept-dns=true --accept-routes=false
   ```

   Tailscale SSH changes how connections to the node's Tailscale address on port 22 are handled and can interrupt existing sessions on that address. Keep a console/fallback session. Do not run these commands on a shared router or existing host without reviewing its requirements. This setup does not require an exit node, subnet router, Serve, or Funnel.

3. Review the tailnet access policy in the Tailscale admin console. Adapt `tailscale/policy.example.json` using your actual administrator identity and local Unix username. It uses a dedicated server tag, grants only SSH/dashboard ports, and requires an SSH check action. Grant rules and SSH rules are both required for Tailscale SSH. Replace the example identity before saving, and assign the server tag after authorizing tag ownership.

   This is a sample, not an export of the live policy and not a patch to append blindly. Existing broad grants/ACLs can continue to allow unwanted access. Merge deliberately and use the admin console's policy validation and tests. In localhost/tunnel mode, remove port 9119 from the grant: only SSH is needed. Restrict local login to the intended non-root account; do not add root casually.

4. Test from a separate trusted tailnet device:

   ```bash
   tailscale status
   tailscale ping VPS_TAILSCALE_NAME
   ssh hermes@VPS_TAILSCALE_NAME
   ```

   Replace both the name and Unix username. Confirm the new path really works before removing any public SSH fallback. Review device approval, key expiry, and account MFA in the admin console; these were not verified/exported. Do not disable key expiry automatically.

## Dashboard: choose one mode

### A. Localhost + SSH tunnel (default repository template)

Keep `systemd/hermes-dashboard.service` bound to `127.0.0.1`. From your trusted computer:

```bash
ssh -N -L 9119:127.0.0.1:9119 hermes@VPS_TAILSCALE_NAME
```

Open `http://127.0.0.1:9119` on that computer. No inbound tailnet dashboard-port grant is required. Configure application authentication as described in `security.md`.

### B. Direct tailnet-only access (matches the source binding pattern)

Use `systemd/hermes-dashboard-tailscale.service.example`. Read `tailscale ip -4` on the new VPS, and replace the unit's `REPLACE_WITH_THIS_VPS_TAILSCALE_IPV4` placeholder. Copy it to `~/.config/systemd/user/hermes-dashboard.service` instead of the localhost template; never run both units. Keep the existing `.env` local and restrict dashboard access in the tailnet policy.

```bash
tailscale ip -4
# Review and substitute the example's bind-IP placeholder first.
mkdir -p ~/.config/systemd/user
cp systemd/hermes-dashboard-tailscale.service.example ~/.config/systemd/user/hermes-dashboard.service
systemctl --user daemon-reload
systemctl --user enable --now hermes-dashboard.service
systemctl --user restart hermes-dashboard.service
systemctl --user status hermes-dashboard.service
```

Do not execute the copy/start commands with the placeholder still present. Ensure Tailscale is online before starting this user service; its restart policy handles a temporarily absent bind address. It does not depend on a system unit from within the user systemd manager.

Access `http://VPS_TAILSCALE_IPV4:9119` from an allowed tailnet device. Tailnet transport is encrypted, but this is still an HTTP application endpoint; do not expose it through public forwarding or assume a tailnet replaces application authentication. Never bind it to `0.0.0.0` or `::` for convenience. No Serve/Funnel configuration is necessary.

## Verification

- Confirm `tailscale status`, the device's online status, and its policy in the admin console.
- Confirm `ss -lnt` shows the dashboard only on the chosen localhost/Tailscale address.
- Without credentials, `/api/config` and `/api/config/raw` must be rejected. A 200 response for `/` or `/login` alone is not an authentication failure; the login page is public.
- Test actual login and dashboard access from an authorized device, and denial from an unauthorized one. Never paste credentials into a command or chat log.
- Check SSH/firewall and Docker limitations in `security.md` before claiming the host is locked down.

## Official references

- Linux installation: https://tailscale.com/kb/1031/install-linux
- Tailscale SSH: https://tailscale.com/kb/1193/tailscale-ssh
- Tailnet policy syntax: https://tailscale.com/kb/1337/acl-syntax
- Grants: https://tailscale.com/kb/1324/grants
- CLI reference: https://tailscale.com/kb/1080/cli
- Ubuntu firewall walkthrough: https://tailscale.com/kb/1077/secure-server-ubuntu
