# Secure Remote Access Setup: Tailscale + VNC

This guide documents how to set up secure remote access to an Ubuntu machine (server or desktop) using [Tailscale](https://tailscale.com/) for networking, plus VNC for GUI access when needed. All values below are placeholders — replace them with your own before use.

> **Why Tailscale?** It creates a private mesh VPN between your devices using WireGuard. There's no need to open ports on your router or expose SSH to the public internet. Devices can only reach each other if they're authenticated members of the same private network (a "tailnet").

---

## 1. Install Tailscale on the Ubuntu machine

SSH into the machine (locally, or over your home network before you leave), then run:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Bring it up with Tailscale SSH enabled and a clear hostname:

```bash
sudo tailscale up --ssh --hostname=<your-machine-name>
```

This prints an authentication URL:

```
To authenticate, visit:
        https://login.tailscale.com/a/<auth-token>
```

Open that URL in any browser and sign in with the account you want to use for your tailnet (Google, GitHub, Microsoft, or email). Use the **same account** for every device you want in the same private network.

Repeat this install step on every machine you want to reach remotely (servers, laptops, etc.) — each gets its own `--hostname`.

---

## 2. Enable `--ssh` mode

Passing `--ssh` (as above) lets Tailscale itself broker SSH authentication using your tailnet identity, rather than relying solely on traditional SSH key files. This means access is tied to devices authorized in your tailnet — not to a public IP or open port.

Traditional SSH (`sshd`) can still run alongside this as a fallback; it's not required to disable it.

---

## 3. Important: disable key expiry for long absences

By default, Tailscale device keys expire after ~180 days, requiring re-authentication. If you're going to be away and unable to physically access a machine, this can lock you out.

1. Go to the [Tailscale admin console → Machines](https://login.tailscale.com/admin/machines)
2. For each machine you depend on, open the `...` menu → **Disable key expiry**

Do this for every device you won't be able to physically reach if it were to expire.

---

## 4. (Optional) Advertise a subnet route

If you also need to reach *other* devices on the same LAN that don't have Tailscale installed (e.g. a router admin page, a NAS, a smart plug), advertise the local subnet from one machine:

```bash
sudo tailscale up --ssh --hostname=<your-machine-name> --advertise-routes=<your-lan-subnet>/24 --accept-routes
```

Find your LAN subnet with:

```bash
ip route | grep default
```

Then approve the route in the admin console: select the machine → **Edit route settings** → approve the advertised route.

If every machine you need already has Tailscale installed directly, this step is optional.

---

## 5. Ensure Tailscale survives a reboot

### Linux (Ubuntu server/desktop)

Tailscale installs as a systemd service. Confirm it's enabled:

```bash
systemctl is-enabled tailscaled
```

Should return `enabled`. If not:

```bash
sudo systemctl enable --now tailscaled
```

Auth state persists in `/var/lib/tailscale/` across reboots — a restart alone won't log the device out.

Test it for real, not just in theory:

```bash
sudo reboot
```

Wait ~60 seconds, then from another device on the tailnet:

```bash
ssh <username>@<tailscale-ip>
tailscale status   # should show a logged-in state
```

### Windows

Tailscale installs as a Windows service that starts automatically. Verify:

```powershell
Get-Service Tailscale
```

`Status` should be `Running`, `StartType` should be `Automatic`. If not:

```powershell
Set-Service -Name Tailscale -StartupType Automatic
```

### macOS

The App Store version of Tailscale runs as a login item, so it only starts once a **user session logs in** — not purely on boot.

- Check **System Settings → General → Login Items** to confirm Tailscale is listed and enabled
- If the Mac requires manual login after reboot, Tailscale won't start until someone logs in. For a machine that must be reachable after an unattended reboot, consider enabling automatic login (**System Settings → Users & Groups → Login Options**), weighing this against the physical-security tradeoff of doing so.

---

## 6. Connect from a Windows client

Install the [Tailscale Windows client](https://tailscale.com/download/windows) and sign in with the same account used on your Ubuntu machine.

Then, from a terminal (PowerShell, or an SSH client like PuTTY):

```powershell
ssh <username>@<tailscale-ip>
```

Find `<tailscale-ip>` by running `tailscale status` on the Windows machine — it will list all devices in your tailnet along with their `100.x.x.x` addresses.

---

## 7. Connect from a Mac client

Install Tailscale via Homebrew or the Mac App Store:

```bash
brew install --cask tailscale
```

Sign in with the same account, then confirm connectivity:

```bash
tailscale status
```

Connect over SSH:

```bash
ssh <username>@<tailscale-ip>
```

---

## 8. VNC setup on the Ubuntu machine (view existing desktop session without disrupting it)

If you need to see the machine's actual running desktop — including any already-open applications — without spawning a new session or disturbing what's running, use `x11vnc` attached to the existing display, rather than a tool that creates a fresh virtual session.

Install it:

```bash
sudo apt install x11vnc -y
```

Set a password:

```bash
x11vnc -storepasswd <your-vnc-password> ~/.vnc/passwd
```

Start it attached to the live desktop session:

```bash
x11vnc -display :0 -auth /run/user/<uid>/gdm/Xauthority -forever -shared -rfbauth ~/.vnc/passwd -bg
```

Flag reference:
- `-display :0` — attaches to the real, already-running desktop rather than creating a new one
- `-shared` — allows your remote connection without kicking out any existing local session
- `-forever` — keeps the server running after you disconnect
- `-bg` — runs in the background

> Note: `<uid>` is typically `1000` for the first user account on Ubuntu. Confirm with `id -u <username>`.

### Run it as a systemd service (so it survives reboots)

```bash
sudo nano /etc/systemd/system/x11vnc.service
```

```ini
[Unit]
Description=x11vnc server
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -display :0 -auth /run/user/<uid>/gdm/Xauthority -forever -shared -rfbauth /home/<username>/.vnc/passwd
Restart=on-failure
User=<username>

[Install]
WantedBy=multi-user.target
```

Enable and start it:

```bash
sudo systemctl enable --now x11vnc
```

> **Important:** for `-display :0` to show anything, the machine must already be logged into an active desktop session (not sitting at a lock/login screen). If the screen locks after inactivity, VNC will still connect but will show the lock screen rather than the desktop.

---

## 9. Connect to VNC from a Mac

Using Finder:

```
Cmd+K → vnc://<tailscale-ip>
```

Or directly:

```bash
open vnc://<tailscale-ip>
```

Enter the VNC password set in step 8 when prompted.

---

## 10. Verification checklist before relying on this setup

- [ ] `tailscale status` shows all expected devices from every client
- [ ] SSH connects successfully to each machine using its Tailscale IP
- [ ] Key expiry disabled for every machine you can't physically reach
- [ ] A real reboot test confirms Tailscale reconnects automatically
- [ ] VNC connects and shows the live desktop, not a blank/new session
- [ ] Tested from a network other than the machine's home network (e.g. mobile hotspot), to confirm it isn't accidentally relying on local-network access

---

## Security notes

- Tailscale IPs (`100.x.x.x` range) are only reachable by devices already authenticated into your tailnet — they are not publicly routable addresses.
- The real security boundary is your Tailscale account login. Use a strong, unique password and enable two-factor authentication on it.
- Avoid exposing SSH or VNC directly to the public internet (e.g. via router port forwarding) — Tailscale is specifically meant to avoid that exposure.
