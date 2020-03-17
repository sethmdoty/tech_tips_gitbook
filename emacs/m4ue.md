# M4Ue

```text
brew install mu
```

Create a file: `.mbsyncrc`.  This configuration uses the macOS keychain.

```text
IMAPAccount icloud
Host imap.mail.me.com
User email_username #not XXX@me.com etc.
PassCmd "security find-generic-password -s mbsync-icloud-password -w"
Port 993
SSLType IMAPS
SSLVersions TLSv1.2
AuthMechs PLAIN

IMAPStore icloud-remote
Account icloud

MaildirStore icloud-local
Path ~/.mbox/icloud/
Inbox ~/.mbox/icloud/inbox
Trash Trash

#
# Channels and Groups 
# (so that we can rename local directories and flatten the hierarchy)
#
#
Channel icloud-folders
Master :icloud-remote:
Slave :icloud-local:
Patterns "INBOX" "Saved" "Drafts" "Archive" "Sent*" "Trash"
Create Both
Expunge Both
SyncState *

Group icloud
Channel icloud-folders


```

To send email we need to install msmtp

```text
brew install msmtp
```

Then create a `.msmtprc` file

```text
defaults
auth on
port 587
tls on
tls_certcheck on
tls_starttls on
logfile  ~/.config/msmtp.log

account icloud
host smtp.mail.me.com
from email@icloud.com
user email ##not user@icloud.com
passwordeval "security find-generic-password -s mbsync-icloud-password -w"

account default : icloud
```

