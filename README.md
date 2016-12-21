# gpgref
My personal GnuPG configuration.

It should work on GNU+Linux systems running gnupg v1 or v2 and Microsoft Windows running [Gpg4win](https://www.gpg4win.org).

### Installation

Setup involves copying the configuration files from here into the home directory of your system.
Configuration is the same whether you are using Windows or GNU+Linux, except where noted below.

**For Microsoft Windows**

Copy [gpg.conf](https://github.com/Kajisav/gpgref/raw/master/gpg.conf) to the %APPDATA%\gnupg directory.

Typically, this is located at C:\Users\YourName\AppData\Roaming\gnupg. You can press WindowsKey + R and type %APPDATA%\gnupg and click OK or press Enter to open a file explorer window to the correct directory.

If you plan to use PuTTY or OpenSSH to connect to remote servers using a PGP/GPG authentication key or a smartcard, then also copy [gpg-agent.conf](https://github.com/Kajisav/gpgref/raw/master/gpg-agent.conf) to the %APPDATA%/gnupg directory right along with gpg.conf.

**For GNU+Linux**

Copy my gpg.conf to your GnuPG home directory:

    git clone https://github.com/Kajisav/gpgref.git
    cd gpgref/
    cp gpg.conf ~/.gnupg/.

If you plan to use OpenSSH along with your PGP/GPG key or with a smartcard, you should also copy gpg-agent.conf your GnuPG home:
    
    cp gpg-agent.conf ~/.gnupg/.
    
If you are using the Gnome 3 desktop window manager, you probably also need to disable Gnome Keyring from intercepting signals to gpg-agent. See below.

### GNU+Linux Environment Setup

Copy .pam_environment or .bashrc from the gpgref repository to your user's home directory.

Using pam_env should cause any process using SSH_AUTH_SOCK when talking to ssh-agent, to talk to the gpg-agent instead. The bash solution is perhaps less elegant and may be more error prone when it comes to changing TTY and X sessions, but may be preferred if you don't want gpg-agent overriding ssh-agent system-wide, for instance.

**1st Time Environment Setup**

If you used the pam_env method above:
    
    export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/gnupg/S.gpg-agent.ssh" 

If you used the Bash method:
    
    source ~/.bashrc

References above to $XDG_RUNTIME_DIR points to /run/user/1000/ on my system, set by X. You can do echo $XDG_RUNTIME_DIR to confirm where your gnupg socket directory is located. You can also run gpg-agent from the command line along with --daemon to see the directory gpg-agent is using for its socket and export that to SSH_AUTH_SOCK.

**Disabling Gnome Keyring's SSH and PKI support**

Gnome3's built-in keyring will set SSH_AUTH_SOCK on startup to $XDG_RUNTIME_DIR/keyring/ssh and intercept traffic intended for ssh-agent (or in our case, to gpg-agent). To disable this:

    cp /etc/xdg/autostart/gnome-keyring-*.desktop ~/.config/autostart/.
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-ssh.desktop
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-pkcs11.desktop

As far as I know, this is a Gnome-specific circumstance and the interception by Gnome Keyring is by design; it just doesn't play well when using smartcards for OpenSSH authentication, which I am doing, so I disable that part of Gnome Keyring's functionality.
