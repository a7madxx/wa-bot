# Safe WhatsApp View-Once Recovery Bot

## Overview

This bot recovers view-once images, videos, and voice messages from private
WhatsApp chats.

WhatsApp normally withholds view-once media from linked devices. To recover an
item without sending a manual reply, keep the bot running and open the view-once
message from the WhatsApp account linked to this bot. The bot watches for the
owner view/read update, uses cached metadata to download the media into memory,
and sends a recovered copy to your own WhatsApp chat.

The bot uses the account that scans the QR code. You do not manually enter a
phone number.

## Important Warning

This project uses Baileys, an unofficial WhatsApp client. Using unofficial
clients may violate WhatsApp's terms and could put the linked account at risk.

The local `auth/` directory contains sensitive linked-device credentials. Anyone
who obtains this directory may be able to use the linked WhatsApp session.

## Requirements

- Node.js 20 or newer
- npm
- A trusted and updated computer
- WhatsApp installed on your phone

## First-Time Setup

Open PowerShell in the project directory:

```powershell
cd "D:\Users\ahmed\Downloads\wa-bot"
```

Install the pinned dependencies:

```powershell
npm ci
```

Verify the project:

```powershell
npm test
npm audit
npm audit signatures
```

The tests should pass, and `npm audit` should report zero known vulnerabilities.

## Connect WhatsApp

Start the bot:

```powershell
npm start
```

When a QR code appears:

1. Open WhatsApp on your phone.
2. Open **Settings > Linked Devices**.
3. Select **Link a Device**.
4. Scan the QR code shown in PowerShell.
5. Wait for the terminal to display `Connected`.

Never share the QR code or a screenshot of it.

## Recover View-Once Media

1. Receive a view-once image, video, or voice message in a private chat. It may
   have arrived before the bot started if WhatsApp includes it in history sync.
2. Keep the bot running and wait for the message to be cached.
3. Open or play the view-once message from the linked WhatsApp account.
4. Wait for the recovered media to appear in your own WhatsApp chat.

If viewing does not trigger recovery, reply to the unopened view-once message
from the linked WhatsApp account as a fallback.

Expected diagnostic messages:

```text
Detected owner view of cached view-once media
Recovering incoming view-once media
Recovered media sent to the authenticated account
```

Only views or replies synchronized from the linked account can trigger recovery.
Group messages, status updates, and actions from other people are ignored. Old
synced view-once messages are cached only in memory, so wait for history sync to
finish before opening or replying while the bot is running.

## Start And Stop

Start the bot:

```powershell
npm start
```

Stop the bot by pressing:

```text
Ctrl+C
```

Stopping the process does not unlink the WhatsApp session. Starting it again
normally reconnects without another QR scan.

Restart the bot after changing its code:

```text
Ctrl+C
```

```powershell
npm start
```

## Change The Linked Phone Number

The bot always sends recovered media to the WhatsApp account that scanned its QR
code.

To change accounts:

1. Stop the bot with `Ctrl+C`.
2. On the old account, open **WhatsApp > Linked Devices** and log out the bot.
3. Delete the old local authentication session:

```powershell
Remove-Item -Recurse -Force -LiteralPath ".\auth"
```

4. Start the bot:

```powershell
npm start
```

5. Scan the new QR code using the new WhatsApp account.

Never manually edit files inside `auth/`.

## Completely Revoke Access

1. Stop the bot with `Ctrl+C`.
2. Open **WhatsApp > Linked Devices**.
3. Select the device created by the bot and log it out.
4. Delete the local authentication session:

```powershell
Remove-Item -Recurse -Force -LiteralPath ".\auth"
```

The bot cannot reconnect until a new QR code is scanned.

## Diagnostics

Privacy-safe operational messages are written to `diagnostic.log`.

Display recent messages:

```powershell
Get-Content .\diagnostic.log -Tail 50
```

Clear the diagnostic log:

```powershell
Remove-Item .\diagnostic.log -ErrorAction SilentlyContinue
```

The diagnostic log does not contain phone numbers, names, message text, media,
or authentication credentials.

### Common Diagnostic Messages

| Message | Meaning |
|---|---|
| `Connected` | The linked-device session is connected. |
| `Using current WhatsApp Web version ...` | The bot fetched the client revision required for the connection handshake. |
| `Detected view-once media, but WhatsApp withheld it from this linked device` | The linked bot received only a view-once stub and cannot download that item automatically. Reply to the unopened item from the linked account to attempt fallback recovery. |
| `Ignored message without decryptable content` | A non-view-once message could not be decrypted. |
| `Detected owner view of cached view-once media` | Your account opened or played a cached view-once message. |
| `Detected owner reply to view-once media` | Your reply included usable quoted media metadata. |
| `Cached ... old private view-once message(s) for reply recovery` | Old media metadata was received through history sync and cached in memory. |
| `Recovering incoming view-once media` | The bot is downloading the quoted media. |
| `Recovered media sent to the authenticated account` | WhatsApp accepted the recovered media message. |
| `Recovery failed: ...` | Recovery failed; inspect the error text. |

## Troubleshooting

### Nothing Happens

Confirm that:

- The terminal displays `Connected`.
- You restarted the bot after changing its code.
- The view-once message is in a private chat, not a group.
- You opened, played, or replied from the WhatsApp account linked to the bot.
- The bot cached the view-once message before you opened it.
- You sent a fresh view or reply action after the bot connected.

Then inspect:

```powershell
Get-Content .\diagnostic.log -Tail 50
```

### QR Code Appears Again

The existing session is missing, invalid, or logged out. Scan the QR using the
intended WhatsApp account.

### Connection Closes With 405

Status 405 means WhatsApp rejected the client handshake, usually because the
WhatsApp Web revision is stale. The bot now fetches the current revision before
opening each socket and stops after three consecutive rejections instead of
retrying forever. Confirm that `https://web.whatsapp.com` is reachable, then
restart the bot. If the rejection remains, update the pinned Baileys dependency.

### Recovered Media Does Not Appear Immediately

Wait several seconds and open your WhatsApp self-chat. The terminal should show:

```text
Recovered media sent to the authenticated account
```

### Tests Fail With `spawn EPERM`

This can occur when Windows security software blocks Node's isolated test
process. Run PowerShell with the required permissions, then retry:

```powershell
npm test
```

## Security Practices

- Keep the project on a trusted, encrypted local drive.
- Do not store it in OneDrive, Dropbox, or another cloud-synced folder.
- Never upload or share `auth/`.
- Never share QR-code screenshots.
- Regularly inspect WhatsApp's **Linked Devices** list.
- Stop and unlink the bot when it is not needed.
- Do not run untrusted software while the linked session exists.
- Keep Node.js and dependencies updated only after reviewing changes.

## Built-In Safety Controls

- Recovery can be triggered only by the authenticated account's view or reply.
- Groups, broadcasts, status updates, newsletters, and other senders are ignored.
- Recovered media is sent only to the authenticated account.
- Media is never intentionally saved to disk.
- Contact identifiers and message contents are not logged.
- Media downloads are limited to 50 MB.
- Recovery is rate-limited.
- Downloads are restricted to WhatsApp media infrastructure.
- Synced history is scanned only for private view-once media and cached in
  memory, with a 1,000-item limit.
- Online-presence announcements are disabled.
- Authentication files and logs are excluded from Git.
- Dependency versions are pinned.

## Limitations

- View-triggered recovery works only when WhatsApp emits a read/play update and
  the bot already cached usable view-once media metadata.
- Direct automatic recovery at receipt time may not work because WhatsApp
  intentionally withholds view-once payloads from linked devices.
- A withheld view-once stub contains no media key or download location. The bot
  cannot reconstruct that media; reply recovery works only if WhatsApp includes
  usable quoted-media metadata in the reply event.
- Old recovery works only when WhatsApp supplies usable media metadata during
  history sync and the media is still available on WhatsApp's servers.
- Reply recovery is still available as a fallback for unopened view-once messages.
- Group messages are intentionally ignored.
- Only one recovery is processed at a time.
- Media larger than 50 MB is rejected.
- WhatsApp or Baileys protocol changes may stop the bot from working.
