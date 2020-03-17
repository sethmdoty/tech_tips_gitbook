---
description: Setting up client/server mode on OSX
---

# Emacs- Server Mode

```text
brew cask install emacs
```

After adding your configuration, create a Launch Script at the following location: `~/Library/LaunchAgents/my.emacsdaemon.plis`. It should contain this:

```text
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>my.emacsdaemon</string>
  <key>ProgramArguments</key>
  <array>
    <string>/Applications/Emacs.app/Contents/MacOS/Emacs</string>
    <string>--daemon</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/> <!-- run the program again if it terminates -->
</dict>
</plist>
```

This will launch the daemon when you log in, or you can force it to launch now by running: `launchctl load ~/Library/LaunchAgents/my.emacsdaemon.plist`

Test that your client works by running: `/Applications/Emacs.app/Contents/MacOS/bin/emacsclient -c -a`

If that works, you can set up an Launch file to start the client.  I prefer using the Apple "Automator" application.  

Create a new script

Search for "Run Shell Script"

Set the shell if you have changed it and set the "Pass Input" to "as arguments"

In the command paste your executable path.  I use: 

```text
/Applications/Emacs.app/Contents/MacOS/bin/emacsclient --no-wait -c -a emacs "$@" >/dev/null 2>&1 &1
```

Save this as an Application in your `/Applications` Folder so that you can run it like a normal application

The icon will look silly, so if you want to use the normal Emacs logo you need to do the following:

Using the Finder, navigate to your Application folder and right click on your Emacs application \(not the client or daemon\), and click on `Show Package Contents.` 

Do the same for the Emacs Client application

Copy the `Emacs.icns` file from `Contents/Resources` of the Emacs application to the `Contents/Resources` of the Emacs Client application. 

Notice the sname of  the existing `.icns` file, then delete it.  

Rename `Emacs.icns` to the name of the previous `icns` file

