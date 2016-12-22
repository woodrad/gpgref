# gpgref
My personal GnuPG configuration.

It should work on GNU+Linux systems running [GnuPG](https://gnupg.org/) and Microsoft Windows running [Gpg4win](https://www.gpg4win.org).

It will also work alongside hardware security tokens and smartcards presuming you have the required software installed for your smartcard or token. I've tested using the Yubico NEO and [OpenPGP smartcards](http://www.g10code.com/p-card.html) available from [KernelConcepts.de](http://shop.kernelconcepts.de/#openpgp).

### Installation

Setup involves copying the configuration files from here into the home directory of your system.
Configuration is the same whether you are using Windows or GNU+Linux, except where noted below.

#### For Microsoft Windows

Copy [gpg.conf](https://github.com/Kajisav/gpgref/raw/master/gpg.conf) to the %APPDATA%\gnupg directory.

Typically, this is located at C:\Users\YourName\AppData\Roaming\gnupg. You can press WindowsKey + R and type %APPDATA%\gnupg and click OK or press Enter to open a file explorer window to the correct directory.

If you plan to use PuTTY or OpenSSH to connect to remote servers using a PGP/GPG authentication key or a smartcard, then also copy [gpg-agent.conf](https://github.com/Kajisav/gpgref/raw/master/gpg-agent.conf) to the %APPDATA%\gnupg directory right along with gpg.conf.

If you are using a [Yubico Yubikey](https://www.yubico.com/) (USB hardware token) you should read the [Windows Yubikey Caveats](https://github.com/Kajisav/gpgref#caveats-using-yubikey-on-windows) section below.

#### For GNU+Linux

Copy my gpg.conf to your GnuPG home directory:

    git clone https://github.com/Kajisav/gpgref.git
    cd gpgref/
    cp gpg.conf ~/.gnupg/.

If you plan to use OpenSSH along with your PGP/GPG key or with a smartcard, you should also copy gpg-agent.conf your GnuPG home:
    
    cp gpg-agent.conf ~/.gnupg/.
    
If you are using the Gnome 3 desktop window manager, you probably also need to disable Gnome Keyring from intercepting signals to gpg-agent. [See below](https://github.com/Kajisav/gpgref#disabling-gnome-keyrings-ssh-and-pkcs-11-support).

### GNU+Linux Environment Setup

Copy .pam_environment or .bashrc from the gpgref repository to your user's home directory.

Using pam_env should cause any process using SSH_AUTH_SOCK when talking to ssh-agent, to talk to the gpg-agent instead. The bash solution is perhaps less elegant and may be more error prone when it comes to changing TTY and X sessions, but may be preferred if you don't want gpg-agent overriding ssh-agent system-wide, for instance.

#### 1st Time Environment Setup

If you used the pam_env method above:
    
    export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/gnupg/S.gpg-agent.ssh" 

If you used the Bash method:
    
    source ~/.bashrc

References above to $XDG_RUNTIME_DIR points to /run/user/1000/ on my system, set by X. Your GNU+Linux distribution may use a different default location for the gpg-agent socket, such as your home .gnupg directory ~/.gnupg/S.gpg-agent.ssh for instance. If you run ssh-add -L and see something like this:

    me@debian:~$ ssh-add -L
    Error connecting to agent: No such file or directory

That is because the location set for the SSH_AUTH_SOCK variable is incorrect. You can find out where gpg-agent is keeping its socket by starting the agent from the command line. You should check if it's already running first:

    me@debian:~$ ps -A | grep gpg-agent
     1561 ?        00:00:00 gpg-agent
    me@debian:~$ sudo kill `pidof gpg-agent`
    me@debian:~$ gpg-agent --daemon
    SSH_AUTH_SOCK=/home/me/.gnupg/S.gpg-agent.ssh; export SSH_AUTH_SOCK;

So now you know to run 'export SSH_AUTH_SOCK=/home/me/.gnupg/S.gpg-agent.ssh' and remember to also update ~/.pam_environment so this is handled upon startup. This process can differ depending on how your particular distribution is configured. I've tested using Debian Stretch and Ubuntu 16.04 LTS.
    
#### Disabling Gnome Keyring's SSH and PKCS #11 support

Gnome Keyring will set SSH_AUTH_SOCK on startup to $XDG_RUNTIME_DIR/keyring/ssh and intercept traffic intended for ssh-agent (or in our case, to gpg-agent). To disable this:

    mkdir ~/.config/autostart
    cp /etc/xdg/autostart/gnome-keyring-*.desktop ~/.config/autostart/.
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-ssh.desktop
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-pkcs11.desktop

As far as I know, this is just a Gnome Keyring thing and the interception by it is [by design](https://wiki.gnome.org/Projects/GnomeKeyring/Ssh); it just doesn't play well when using smartcards and hardware tokens for OpenSSH authentication and can cause issues when using gpg commands from a remote OpenSSH shell. I use OpenPGP cards and a Yubikey, so I also disable PKCS #11 support in Gnome Keyring's to make sure smartcard middleware/drivers talk directly to gpg-agent.

### Caveats using Yubikey on Windows

If you are using a Yubikey alongside Gpg4win, there is an issue where after unplugging the Yubikey from the system, when you go to plug it back in, the gpg-agent is not able to talk to the Yubikey and trying to do:

    gpg --card-status
    
from a command line results in a card error being returned. The best solution to this that I have tested is to create a simple BAT file with instructions to stop and restart some services after unplugging the Yubikey and making a shortcut on the desktop to this BAT file:

    gpg-connect-agent killagent /bye
    net stop scardsvr
    gpg-connect-agent /bye

You can download the file [from here](https://raw.githubusercontent.com/Kajisav/gpgref/master/CardRemove.bat) or create it yourself using the above commands. Place it somewhere, I used C:\Users\Me\CardRemove.bat and created a shortcut on the desktop by right-clicking the BAT file and selecting "Send to Desktop (Create Shortcut)" from the menu.

Once you have the CardRemove.bat shortcut on your desktop, you can right-click the shortcut and go to "Properties" then click the "Advanced..." button and check the "Run as administrator" checkbox, hit OK, then OK again. Then you have the following procedure when you remove the Yubikey:

    1. Remove Yubikey
    2. Run CardRemove.bat shortcut
    3. gpg-agent is restarted and the Windows Smart Card service is stopped.
    4. Re-insert the Yubikey when you are ready to use it again.
    5. Run 'gpg --card-status' from a command line or check in Kleopatra to confirm the key is working.
