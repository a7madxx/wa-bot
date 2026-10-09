# Free cloud hosting

## Recommended setup

Run this bot on one Oracle Cloud Infrastructure (OCI) Always Free Ubuntu VM.
This is a better fit than a free web host because the bot:

- is a continuously running background process;
- needs its `auth/` directory to survive restarts; and
- does not need a public HTTP endpoint.

The OCI mobile app can start, stop, and restart the VM. When the VM starts,
systemd starts the bot automatically.

OCI usually requires a real credit or debit card for identity verification. It
may place a small temporary authorization hold, but Oracle says a Free Tier
account is not charged unless it is upgraded. Do not upgrade the account, and
create only resources marked **Always Free eligible**.

Always Free is not a reliability guarantee. Oracle may reclaim a compute
instance it classifies as idle, and free-shape capacity is sometimes
unavailable. Keep the source in GitHub; if OCI reclaims the VM, revoke that
linked WhatsApp device and create a new VM/session.

> [!WARNING]
> Baileys is an unofficial WhatsApp client. Hosting it does not remove the risk
> that WhatsApp may reject or restrict the linked account. The cloud VM also
> holds credentials that can access the linked WhatsApp session.

## 1. Put the current code on GitHub

The current local changes must be reviewed, committed, and pushed before the
VM can clone them. The `auth/` directory and logs are ignored by Git and must
never be committed.

From PowerShell in this project directory, review the files first:

```powershell
git status
git diff
```

Then stage only the intended source and deployment files:

```powershell
git add .gitignore README.md HOSTING.md index.js package.json package-lock.json test deploy
git commit -m "Prepare WhatsApp bot for cloud hosting"
git push origin main
```

## 2. Create the free VM

In the OCI web console:

1. Create a compute instance named `wa-bot` in the account's home region.
2. Choose an Ubuntu image.
3. Choose an **Always Free eligible** shape. Prefer Ampere A1 Flex with 1 OCPU
   and 6 GB memory. The E2.1.Micro shape also works but has only 1 GB memory.
4. Assign a public IPv4 address for the initial SSH setup.
5. Generate and safely download the SSH private key. Never upload this key to
   GitHub or share it.
6. Do not add an inbound rule for the bot. It makes outbound connections only.
   Keep only SSH access needed for administration.

Always verify that the chosen shape, boot volume, and region are labeled
**Always Free eligible** before creating the instance.

## 3. Connect and install Node.js

From PowerShell, replace the key path and IP address:

```powershell
ssh -i "C:\path\to\ssh-key.key" ubuntu@203.0.113.10
```

On the VM, install Git and Node.js 22:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git
curl -fsSL https://deb.nodesource.com/setup_22.x -o /tmp/nodesource_setup.sh
sudo -E bash /tmp/nodesource_setup.sh
sudo apt-get install -y nodejs
node --version
```

The reported Node.js version must be 20 or newer.

## 4. Download and pair the bot

Clone the repository and install the pinned dependencies:

```bash
cd /home/ubuntu
git clone https://github.com/vublich/wa-bot.git
cd wa-bot
npm ci --omit=dev
mkdir -p auth
chmod 700 auth
npm start
```

Scan the displayed QR code from **WhatsApp > Settings > Linked Devices > Link
a Device**. Wait for `Connected`, then press `Ctrl+C` once. This creates the
cloud VM's own authentication session; do not copy the laptop's `auth/`
directory to the cloud.

## 5. Enable automatic startup

Install the included systemd service:

```bash
sudo cp deploy/wa-bot.service /etc/systemd/system/wa-bot.service
sudo systemctl daemon-reload
sudo systemctl enable --now wa-bot.service
sudo systemctl status wa-bot.service --no-pager
```

View recent privacy-safe diagnostic messages with:

```bash
sudo journalctl -u wa-bot.service -n 50 --no-pager
```

## 6. Turn it on or off from a phone

Install the **Oracle Cloud Infrastructure** app for Android or iPhone and sign
in to the same OCI account. Open **Resources > Compute instances > wa-bot**:

- **Stop** powers off the VM and therefore the bot.
- **Start** powers on the VM; systemd starts the bot automatically.
- **Restart** reboots the VM and automatically starts the bot again.

Use the normal graceful **Stop** action, not force stop. Stopping the VM does
not delete the boot disk, so the `auth/` session remains available at the next
start. Never select **Delete** or **Terminate**.

## Updating the bot later

After pushing reviewed changes to GitHub, connect to the VM and run:

```bash
sudo systemctl stop wa-bot.service
cd /home/ubuntu/wa-bot
git pull --ff-only
npm ci --omit=dev
sudo systemctl start wa-bot.service
sudo systemctl status wa-bot.service --no-pager
```

## Completely revoke the cloud bot

1. In WhatsApp, open **Settings > Linked Devices** and log out the cloud bot.
2. Stop the OCI VM.
3. Terminate the VM only if its files are no longer needed.
