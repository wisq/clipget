What is this?
-------------

clipget and clipput are a pair of simple scripts that allow you to copy your clipboard to a Windows machine running Cygwin with SSH.

The problem
-----------

I have a Windows machine that I only use for games.  I don't like Windows, I don't trust Windows, but my games run on Windows, so I use Windows.

Some of my games require passwords.  Some of those games are written by misguided developers who think that making people type their password in _every time the game launches_ somehow increases security, instead of just making people choose crappy passwords.

I normally generate massive long passwords using the excellent [1Password](https://agilebits.com/onepassword) program.  (Get 1Password.  It's awesome.)  And I refuse to compromise on that just because of some annoying game code.  But, transcribing a ridiculously long password by hand gets pretty tiring after a couple of times.

(Yes, I know, 1Password does have a Windows version.  But given all the keylogging spyware out there, I don't even trust Windows with my master password, much less my entire password database.)

Cygwin to the rescue … sort of.  I can SSH from my Mac into my Windows machine.  And Cygwin does have a lovely ```/dev/clipboard``` that I can write to.  But the two don't play well together: If you SSH in and write to ```/dev/clipboard``` directly, it won't alter the clipboard for the user logged in on the physical console.  (They're separate clipboards.)

The solution
------------

The solution is pretty simple.

1. Run a program on the desktop that listens on a FIFO and writes to the clipboard.
2. SSH in and run a program that writes the password to the FIFO.

So, that's what this is.

clipget
-------

clipget is a Ruby program that waits for input from a FIFO.  It writes the data it receives to the clipboard.  As such, it __must be run directly on the desktop__ in order to access the correct clipboard.

clipget will loop and continue to receive data for as long as it's running.  As such, you can just run it once, leave it running (or minimised), and continually copy clipboard data over as needed.

clipget will also clear the clipboard 30 seconds after receiving data.  This prevents passwords from sitting in the clipboard buffer forever.  (This is similar to what 1Password itself does.)

clipput
-------

clipput is a simple shell script that writes to the FIFO.  You can SSH in and run it, and either type (or paste) the data you want to send, or pipe it in directly from another program.  (If you're typing or pasting, don't forget to hit control-D to send an EOF.)

Automation
----------

Want to send your current clipboard to your Windows machine, without having to actually paste or type it in yourself?  Easy!

From Mac OSX:

```shell
pbpaste | ssh windowsbox /path/to/clipput
```

From another Windows machine running Cygwin:

```shell
ssh windowsbox /path/to/clipput < /dev/clipboard
```

I don't know a Linux version offhand, but I'm sure you Linux users can figure out something. :)

Limitations
-----------

Due to the way FIFOs and file I/O work on Windows/Cygwin, to terminate clipget, you'll need to hit control-C to interrupt the program, __and also__ issue a (dummy) clipput to wake up clipget.

clipput is silent, and if clipget is not currently running, it'll hang silently and wait for you to run it.  Again, that's just how FIFOs work.

Using a local Unix socket would probably solve all this, but this works for now.
