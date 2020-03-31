# Installation

## If using OSX

I'm using the emacs-mac option in most cases following the instructions here:  [https://github.com/railwaycat/homebrew-emacsmacport](https://github.com/railwaycat/homebrew-emacsmacport)

After this port is added be sure to include module support.  It looks like its there by default, but vterm gave me some issues without adding it in specifically.  I also like the modern icon.

```text
brew tap railwaycat/emacsmacport
brew install emacs-mac --with-modules --with-modern-icon
```

### If using a Nitrokey or YubiKey for authentication

This was overly complicated in OSX land, and the bulk of this working is due to this website: [https://evilmartians.com/chronicles/stick-with-security-yubikey-ssh-gnupg-macos](https://evilmartians.com/chronicles/stick-with-security-yubikey-ssh-gnupg-macos). Here is what it boils down to.  We need to override the SSH agent configs to use the gpg-agent.  If you don't, emacs won't see your SSH keys at all unless you start it from a shell.  

First I added the following to the top of my .profile. You need to make sure your shell uses gpg-agent.  I just used the ohmyzsh gpg-agent plugin for this, but you could also add the following to the top of your .profile:

```text
gpgconf --launch gpg-agent
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
```

This only works for shell apps though.  I care more about GUI apps knowing too.  Here is where it sucks.  First create a `homebrew.gpg.gpg-agent.plist` in `~/Library/LaunchAgents.`  It should contain the following:

```text
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <!-- Sets a name for a task -->
  <key>Label</key>
  <string>homebrew.gpg.gpg-agent</string>
  <!-- Sets a command to run and its options -->
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/gpgconf</string>
    <string>--launch</string>
    <string>gpg-agent</string>
  </array>
  <!-- Tells it to run the task once the XML is loaded -->
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

This file ensures that we have a gpg-agent launched when we log in.  You can test it by running: 

```text
launchctl load -F ~/Library/LaunchAgents/homebrew.gpg.gpg-agent.plist
```

Then we need to ensure the SSH\_AUTH\_SOCK is correct.  The problem here is we need this global one to be symlinked to our home directory.  so...another launchd file.  Lets create`~/Library/LaunchAgents/link-ssh-auth-sock.plist`

Inside it put:

```text
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>link-ssh-auth-sock</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/sh</string>
    <string>-c</string>
    <string>/bin/ln -sf $HOME/.gnupg/S.gpg-agent.ssh $SSH_AUTH_SOCK</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

Test it with 

```text
launchctl load -F ~/Library/LaunchAngents/link-ssh-auth-sock.plist
```

## Configuration

You can check out my config and settings here: [https://github.com/sethmdoty/emacs.d/blob/master/emacs.org](https://github.com/sethmdoty/emacs.d/blob/master/emacs.org)



