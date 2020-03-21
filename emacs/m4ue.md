---
description: Configure all the pieces to supplement my mu4e emacs config
---

# M4Ue

```text
brew install mu isync
```

Create a file: `.mbsyncrc`.  This configuration uses the macOS keychain.

```text
# Based on http://www.macs.hw.ac.uk/~rs46/posts/2014-01-13-mu4e-email-client.html
IMAPAccount icloud
Host imap.mail.me.com
User sethmdoty #not XXX@me.com etc.
PassCmd "security find-generic-password -s mbsync-icloud-password -w"
Port 993
SSLType IMAPS
SSLVersions TLSv1.2
AuthMechs PLAIN

IMAPStore icloud-remote
Account icloud

MaildirStore icloud-local
SubFolders Verbatim
Path ~/.mbox/icloud/
Inbox ~/.mbox/icloud/inbox

#
# Channels and Groups 
# (so that we can rename local directories and flatten the hierarchy)
#
#
Channel icloud-folders
Master :icloud-remote:
Slave :icloud-local:
Patterns *
Create Both
Expunge Both
SyncState *

###work email
IMAPAccount gmail
Host imap.gmail.com
User full.email@gmail.com
PassCmd "security find-generic-password -s mbsync-gmail-password -w"
SSLType IMAPS
AuthMechs LOGIN

IMAPStore gmail-remote
Account gmail

MaildirStore gmail-local
Path ~/.mbox/gmail/
Inbox ~/.mbox/gmail/inbox

Channel gmail-default
Master :gmail-remote:
Slave :gmail-local:Inbox

Channel gmail-sent
Master :gmail-remote:"[Gmail]/Sent Mail"
slave  :gmail-local:Sent

Channel gmail-trash
Master :gmail-remote:"[Gmail]/Trash"
slave  :gmail-local:Trash

Channel gmail-archive
Master :gmail-remote:"[Gmail]/All Mail"
slave  :gmail-local:All

Channel gmail-junk
Master :gmail-remote:"[Gmail]/Spam"
slave  :gmail-local:JunkCreate

Create Both
Expunge Both
SyncState *

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
logfile        ~/.config/msmtp.log

account icloud
host smtp.mail.me.com
from fullemail@icloud.com
user account
passwordeval "security find-generic-password -s mbsync-icloud-password -w"

account default : icloud

account gmail
host smtp.gmail.com
from full.email@gmail.com
user full.email@gmail.com
passwordeval "security find-generic-password -s mbsync-gmail-password -w"
```

