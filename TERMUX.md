# Run the bot on an Android phone with Termux

This method is free and does not require a cloud account. The phone must remain
powered on and connected to the internet. Android may stop Termux unless its
battery optimization is disabled.

## 1. Prepare Termux

Open Android **Settings > Apps > Termux > Battery** and allow unrestricted
background usage or disable battery optimization for Termux.

Then run these commands in Termux:

```sh
pkg update
pkg upgrade
pkg install git nodejs-lts npm tmux
```

Check that Node.js is version 20 or newer:

```sh
node --version
```

## 2. Download the bot

```sh
cd ~
git clone https://github.com/vublich/wa-bot.git
cd wa-bot
npm ci
```

The latest local changes must be committed and pushed from the laptop before
running `git clone`, or the phone will receive the older GitHub version.

## 3. Pair WhatsApp on the same phone

Start a persistent terminal session:

```sh
termux-wake-lock
cd ~/wa-bot
tmux new -s wa-bot
PAIRING_METHOD=phone npm start
```

Enter the WhatsApp number when prompted. Use digits only and include the
country code; do not include `+`, spaces, parentheses, or hyphens. For example,
an Egyptian number beginning `012...` becomes `2012...`.

The bot prints an eight-character pairing code. Switch to WhatsApp and open:

**Settings > Linked Devices > Link a Device > Link with phone number instead**

Enter the code. Return to Termux and wait for `Connected`.

The phone number and pairing code are not written to `diagnostic.log`. Never
share the pairing code with anyone.

## 4. Leave it running

Detach from the terminal without stopping the bot by pressing `Ctrl+B`, then
`D`. You can now close the Termux window, but do not force-stop the app.

To see the bot again:

```sh
tmux attach -t wa-bot
```

## Start and stop from the phone

To stop the bot, attach to it and press `Ctrl+C`:

```sh
tmux attach -t wa-bot
```

Then release the wake lock:

```sh
termux-wake-unlock
```

To start it later, run:

```sh
termux-wake-lock
cd ~/wa-bot
tmux new -s wa-bot
npm start
```

After the first successful pairing, `PAIRING_METHOD=phone` is no longer needed.
The saved session stays in `~/wa-bot/auth/`.

