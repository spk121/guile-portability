# Setting Up Email in Emacs on Fedora

I’m unsure about best practices, so I’ve pieced together this setup based
on what search engines suggest is commonly used. Here’s how I configured
email fetching and reading in Fedora, targeting Emacs as my client.

## Fetching Email with IMAP

First up, need to be able to fetch my e-mail from upstream. My e-mail
provider has an `imaps` server from which I can receive my e-mail.

### Obtaining an App Password from Yahoo

Yahoo requires app-specific credentials for third-party apps like `isync`.
I followed their support article, "Generate and manage 3rd-party app
passwords," which led me to a webpage where I generated a unique app
password for this setup.

App passwords are safer than using your main account password, as they’re
limited to specific applications and can be revoked independently.

### Using GPG to Avoid Plaintext Password Storage

To avoid storing the App Password for `isync` in plain text, GPG can be
used to offer some protection.  One could store the app password in a file
using GPG's symmetric encryption, so that opening the file requires
its own password.  Assuming the unencrypted app password was saved
in a file named `.mailpass`, encrypting the file would be:

```shell
gpg --symmetric .mailpass
```

and decrypting the file would be:

```shell
gpg --decrypt --quiet .mailpass.gpg
```

**TODO:** consider doing this.  I would need to figure out how to store
symmetric file passwords in the password manager.  Apparently you
can do this using Gnome's Seahorse.

### Configuring `isync`

`isync` (also known as `mbsync`) syncs a remote IMAP mailbox to a local
Maildir. dir. I installed it in Fedora with:

```
sudo dnf install isync
```

I created an config file in the `~/.config/isyncrc` directory, which
is the default location for that file in Fedora:

``` text
IMAPAccount yahoo
Host imap.mail.yahoo.com
Port 993
TLSType IMAPS
User spk121@yahoo.com
PassCmd "cat ~/.mailpass"

IMAPStore yahoo-remote
Account yahoo

MaildirStore yahoo-local
Path ~/Maildir/yahoo/
Inbox ~/Maildir/yahoo/INBOX
SubFolders Verbatim

Channel yahoo
Far :yahoo-remote:
Near :yahoo-local:
Patterns INBOX Sent
MaxMessages 60
ExpireUnread yes
Create Near
Expunge Both
```

**Note:** In this `isyncrc` file it still shows that I'm using a
plaintext file containing the password. `PassCmd "cat ~/.mailpass"` reads
the app password from a separate file (`~/.mailpass`), which I created
and secured (e.g., `chmod 600 ~/.mailpass`).  If I figure out Seahorse
I should change this to:

```bash
PassCmd gpg --decrypt --quiet ~/.mailpass.gpg
```

I set up the local Maildir with:
```bash
mkdir -p ~/Maildir/yahoo
```

### Running `isync` from the Command Line

After saving the `isyncrc` file, the `mbsync` command can be run to
pull down messages from Yahoo's IMAP server.

``` bash
mbsync yahoo
```

This command synchronizes just the two e-mail boxes I've set in the
`Patterns` entry in the `isyncrc` file, namely `INBOX` and `Sent`.

### Running `isync` Periodically

To run this on a regular schedule, I'm going to make a `systemd` user
service.  This requires creating two files: a `.service` file that
describes the commands to be run, and a `.timer` file that describes
how often the command will be run.

`systemd` user service files are stored in the user's local
`systemd config directory.

1. Create the `systemd` user service directory if it doesn't exist:

   ```bash
   mkdir -p ~/.config/systemd/user
   ```

2. In that directory, create an `isync.service` file with the following
   contents:

   ```
   [Unit]
   Description=Sync mail with isync
   After=network-online.target
   Wants=network-online.target

   [Service]
   Type=oneshot
   ExecStart=/usr/bin/mbsync -a
   Restart=no

   [Install]
   WantedBy=default.target
   ```
   Notes:
   * `After` and `Wants`: Ensures that this service only runs if the
     network is running
   * `Restart`: Don't restart `mbsync` after it shuts down, since the
     program is supposed to end.

3. In that directory, create an `isync.timer` file with the following
   contents.

   ```
   [Unit]
   Description=Sync mail every 30 minutes

   [Timer]
   OnCalendar=*-*-* *:2,32:0
   Persistent=true

   [Install]
   WantedBy=timers.target
   ```
   
   Notes:
   * `OnCalendar`: This schedules it to run on the 2nd and 32nd
      minute of every hour

4. Reload `systemd` to recognize the new files

   ```bash
   systemctl --user daemon-reload
   ```

5. Run service on demand once to make sure its working

   ```bash
   systemctl --user start isync.service
   ```

   To see its output, use `journalctl` or `systemctl`

   ```
   journalctl --user --unit=isync.service
   systemctl --user status isync.service
   ```

6. Set service to run on schedule. This involved enabling the timer
   service.

   ```
   systemctl --user enable isync.timer
   ```

   And to start it now without rebooting:

   ```
   systemctl --user start isync.timer
   ```

   To check the status of the timer:

   ```
   systemctl --user staus isync.timer
   ```

## Setting Up Mail Notifications and Indexing

*Gah. I just want `xbiff`.  Give me a cute mailbox and a beep!  Why is
this hard?*

It isn't enough just to fetch the e-mail via IMAP.  It also needs
to be indexed, and there needs to be a new e-mail notification. *Of
course* this is also manual and obscure.  This involves two more
programs:

* `mu` is a program that does the indexing of e-mail. `mu` is a
  prerequisite for the Emacs e-mail reader I'll be using: `mu4e`
* `notify-send` is the command that sends a notification in Gnome

### Setting up `mu`

`mu` is provided by the `maildir-utils` package in Fedora:
```bash
sudo dnf install maildir-utils
```
*NOTE*: in many distributions `mu` is part of a package called `mu`.
This is not the case for Fedora.


Following the `mu4e` documentation at
[Initializing the Message Store](https://www.djcbsoftware.nl/code/mu/mu4e/Initializing-the-message-store.html),
I set up `mu`.

The docs recommend initializing `mu` with the Maildir path:

```bash
mu init --maildir=~/Maildir
```

They also suggest adding one's e-mail address so `mu` recognizes them as
onesself (e.g., for filtering and highlighting):

```bash
mu init --maildir=~/Maildir --my-address=spk121@yahoo.com --my-address=secret-email-mike@cia.gov
```

Then index the Maildir to make e-mails searchable:

```bash
mu index
```

### Setting up automated indexing and notification

When `mbsync` is called every 30 minutes by my `systemd` timer,
I want to do additional steps:

1. re-index the e-mail
2. upon receipt of new e-mail since the last index request, send a
   notification to the screen.

In the `systemd` `isync.service` file, I'm going to replace the
call to `mbsync -a` with a call to a bash script that does the
reindexing and the notification.  Save the following script as
`~/.local/bin/isync_notify.sh`:

```bash
#!/bin/bash

# Path to your Maildir (for mbsync and mu)
MAILDIR="$HOME/Maildir"
# State file to track the last check time
STATE_DIR="$HOME/.cache/isync"
STATE_FILE="$STATE_DIR/isync_notify_time"

# Create the STATE_DIR if it doesn't exist
if [ ! -d "$STATE_DIR" ]; then
    mkdir -p "$STATE_DIR"
fi

# Fetch mail with mbsync
mbsync -a

# Update mu index
mu index

# Get the current timestamp (seconds since epoch)
CURRENT_TIME=$(date +%s)

# Read the last check time (default to 30 minutes ago if file doesn’t exist)
if [ -f "$STATE_FILE" ]; then
    LAST_TIME=$(cat "$STATE_FILE")
else
    # Default to 30 minutes ago (1800 seconds)
    LAST_TIME=$((CURRENT_TIME - 1800))
fi

# Convert LAST_TIME to the format YYYY-MM-DD/HH:MM for mu date: query                                         
LAST_TIME_FORMATTED=$(date -d "@${LAST_TIME}" +%Y-%m-%d/%H:%M)                                                
                                                                                                              
# Query mu for new mail since the last check (in INBOX, unread)                                               
NEW_MAIL_COUNT=$(mu find --fields "i" "date:${LAST_TIME_FORMATTED}..now" \                                    
                    "maildir:/INBOX" "flag:unread" 2>/dev/null | wc -l)

# If there are new emails, notify
if [ "$NEW_MAIL_COUNT" -gt 0 ]; then
    notify-send -u normal "You've got mail!" "$NEW_MAIL_COUNT new email(s)"
fi

# Update the state file with the current time
echo "$CURRENT_TIME" > "$STATE_FILE"
```

Set the shell script as executable with `chmod`.

Now, update the `~/.config/systemd/user/isync.service`

```
ExecStart = /home/mike/.local/bin/isync_notify.sh
```

## Setting up an Emacs E-mail reader

Again, not knowing what is best practice but using what seems common
with the search engines for my Emacs e-mail reader, I'm going with
`mu4e`. It seems popular among Emacs users and is compatible with `Maildir`.

### Installation
`mu4e` is provided by the `maildir-utils` that already provided the `mu`
package in Fedora:

###  Initializing `mu4e` and Basic Emacs Configuration

Now that `mu` is awake and alive, in my Emacs config (e.g., `~/.emacs`),
I added:

```lisp
;; E-mail reader
(require 'mu4e)
(setq mu4e-maildir "~/Maildir/yahoo")
(setq mu4e-get-mail-command "isync_notify.sh")
(setq user-mail-address "spk121@yahoo.com")
(setq user-full-name "Mike Gran")
```

This directs `mu4e` to my Maildir. The `isync_notify.sh` is the same
script that I made for my `systemd` timer job.

I set `user-mail-address` explicitly because my laptop's
default address in an intranet address.

This is sufficient to get `mu4e` mode running on Emacs.  It
can now read e-mail, but, sending isn't enabled yet.

## Setting up `msmtp` for SMTP
To send e-mail, I need an SMTP client for relay messages to
my e-mail provider's SMTP server.

`msmtp` is the SMTP program suggested for use with `mu4e` to send
e-mail.  I installed it as:

```bash
sudo dnf install msmtp
```

### Configuring `msmtp`
`msmtp` looks for a configuration file in `~/.msmtprc`. I added Yahoo's
host and port, plus my account info in that file:

```
defaults
auth            on
tls             on
tls_starttls    on
tls_trust_file  /etc/pki/tls/certs/ca-bundle.crt
logfile         ~/.msmtp.log

account         yahoo
host            smtp.mail.yahoo.com
port            587
from            spk121@yahoo.com
user            spk121@yahoo.com
passwordeval    "cat ~/.mailpass"
account default : yahoo
```

### Configuring `mu4e` on Emacs to use `msmtp`

Now back to adding message sending functionality to Emacs and `mu4e`.
I updated my `~/.emacs` file with the following lines:

```lisp
(setq message-send-mail-function 'message-send-mail-with-sendmail
      sendmail-program "/usr/bin/msmtp"
      mail-specify-envelope-from t
      mail-envelope-from "spk121@yahoo.com")
```

## Conclusion
With this, I’ve got email working in Emacs on Fedora: `isync` fetches
mail, `mu4e` reads it, and `msmtp` sends it via Yahoo’s SMTP. The process
was, as always, a bit obscure and fiddly — typical of Emacs email
setups — but it’s functional now. Next steps might include GPG
encryption for passwords or a notification system like xbiff.
