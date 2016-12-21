# gpgref
My personal GnuPG configuration

### Installation
Copy gpg.conf to ~/.gnupg/ or %APPDATA%/gnupg/ on Windows with Gpg4Win.

### OpenSSH Support
Copy gpg-agent.conf to ~/.gnupg/ or %APPDATA%/gnupg on Windows with Gpg4Win available at https://www.gpg4win.org/.

####Linux Only: Add .pam_environment or .bashrc to your home directory ~/

Using pam_env should cause any process, using SSH_AUTH_SOCK to talk to ssh-agent, to talk to gpg-agent instead. The bash solution is perhaps less elegant and may be more error prone when it comes to changing TTY and X sessions, but may be preferred if you don't want system-wide gpg-agent use instead of ssh-agent, for instance.

You may need to logout and back in to update pam_env or run 'source ~/.bashrc' if using the Bash method.
