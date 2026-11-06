# User services

Generate the gateway unit with `hermes gateway install` on the new VPS; do not copy the old machine-specific executable paths.

The dashboard template assumes the default profile and a launcher at `~/.local/bin/hermes`. Confirm that location using `command -v hermes`. Install this optional unit only after filling dashboard authentication variables in your local `.env`:

```bash
mkdir -p ~/.config/systemd/user
cp systemd/hermes-dashboard.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now hermes-dashboard.service
```

It binds to localhost rather than the old VPS private-network address. Use an SSH tunnel from your computer (`ssh -L 9119:127.0.0.1:9119 USER@VPS`) to access it. Do not expose this port publicly without a separate security review.

## Optional direct Tailscale binding

`hermes-dashboard-tailscale.service.example` documents the source deployment's direct-tailnet pattern without storing its IP address. Replace `REPLACE_WITH_THIS_VPS_TAILSCALE_IPV4` with the destination's `tailscale ip -4` value and install it as `hermes-dashboard.service`, instead of the localhost template. Do not start both templates. Restart the service if replacing an already-running unit; `enable --now` alone does not reload its running process.

The unit keeps authentication values in the local `.env`; never put them in the unit or Git. Tailnet grants must restrict port 9119 to trusted administrators. This user unit deliberately does not declare a dependency on the system-level `tailscaled.service`. Ensure the destination's Tailscale daemon is online before starting it.

See `../docs/tailscale.md` for installation/access checks and `../docs/security.md` for firewall and SSH verification limits.

For services to keep running after logout, an administrator can enable lingering: `sudo loginctl enable-linger "$USER"`.
