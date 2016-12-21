# gpgref
My personal GnuPG configuration.

It should work on GNU+Linux systems running gnupg v1 or v2 and Windows running Gpg4win available at https://www.gpg4win.org.

### Installation
Copy gpg.conf to ~/.gnupg/ or %APPDATA%\gnupg\ on Windows with Gpg4Win.

### OpenSSH Support
Copy gpg-agent.conf to ~/.gnupg/ or %APPDATA%\gnupg\ on Windows with Gpg4Win.

####Linux Only Config:

Copy .pam_environment or .bashrc to your home directory ~/

Using pam_env should cause any process using SSH_AUTH_SOCK when talking to ssh-agent, to talk to the gpg-agent instead. The bash solution is perhaps less elegant and may be more error prone when it comes to changing TTY and X sessions, but may be preferred if you don't want system-wide gpg-agent use instead of ssh-agent, for instance.

**1st Time Environment Setup**

If you used the pam_env method above:
    
    export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/gnupg/S.gpg-agent.ssh" 

If you used the Bash method:
    
    source ~/.bashrc

Gnome3's built-in keyring will set SSH_AUTH_SOCK on startup to $XDG_RUNTIME_DIR/keyring/ssh and intercept traffic intended for ssh-agent (or in our case, to gpg-agent). To disable this:

    cp /etc/xdg/autostart/gnome-keyring-*.desktop ~/.config/autostart/.
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-ssh.desktop
    echo 'X-GNOME-Autostart-enabled=false' >> ~/.config/autostart/gnome-keyring-pkcs11.desktop
