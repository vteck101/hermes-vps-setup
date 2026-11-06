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

For services to keep running after logout, an administrator can enable lingering: `sudo loginctl enable-linger "$USER"`.
