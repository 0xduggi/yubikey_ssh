# Setup
## Intro
Setting up SSH authentication can be confusing for anyone new to remote management with Linux. Adding FIDO2 security keys on top is a great way to improve security, especially when managing your server outside the local network.

## Steps
These steps assume you have purchased a compatible Yubikey and initialized a PIN using [Yubico Manager](https://www.yubico.com/support/download/yubikey-manager/).

### Client Setup
1. Insert your Yubikey into your client device (the one you will be using to manage the server)
2. Open Terminal and generate a new SSH key pair using `ssh-keygen`
Example:
```
ssh-keygen -t ed25519-sk -O resident -O application=ssh:SERVERNAME -O verify-required
```
where `SERVERNAME` is the name of the server you want to SSH into with your new keys.

3. Enter the PIN set for your Yubikey when prompted.
4. Optional: Add a password to protect the private key.

> **Note**: This is optional because we are going to set the key to require verification already using the Yubikey. My *opinion* is that an additional password on top of the Yubikey hardware, PIN, and physical presence confirmation is unnecessary.

5. Choose a location to save the file. E.g. `/home/USERNAME/.ssh/SERVERNAME-sk` 
6. Open the SSH config file `sudo nano ~/.ssh/config`
7. Enter details for the server, following the below example:
```
HOST <IP of Server>
User <username>
PubKeyAuthentication yes
IdentityFile ~/.ssh/USERNAME/SERVERNAME-sk
```
8. Type `ctrl + o` to write the changes.
9. Exit the file with `ctrl + x`.

### Transfer the Public Key to the server
1. Still on the client, open the Public Key we just created using `nano ~/.ssh/SERVERNAME-sk.pub`.
2. We need to copy the contents from this file to the server.
    a. If you already have a valid SSH key for the server on the client device, open a new terminal session, using the existing key to connect to the server.
    b. If you can only connect to your server from a *different* device, we need to get the public key for the newly generated key to that device to copy to the server. This can be done by copying a text file to a USB, file share, or cloud vault (e.g. [OneDrive personal vault](https://www.microsoft.com/en-us/microsoft-365/onedrive/personal-vault)). 

### Server Setup
1. SSH into the server with existing key
2. Run `sudo nano ~/.ssh/authorized_keys` to open the authorized_keys file.
3. Paste the contents of `SERVERNAME-sk.pub` at the end of the `authorized_keys` file, appending `verify-required`.

The entire line should look something like:
```
sk-ssh-ed25519@openssh.com AAAAGnNrLXNzaC1lZDI1NTE5QG9wZW5zc2guY29tAAAAILhX/sFQ8asadflihasdfkljaJDAUDFNkdjfDpakjdfjdWwwMQ== USER@CLIENT verify-required
```
(not a real public key)

4. Type `ctrl + o` to write the changes.
5. Exit the file with `ctrl + x`.

### Test functionality
1. Move back to the client and open a new terminal session.
2. SSH to the server: `ssh USER@SERVERIP -i SERVERNAME-sk`
3. Follow the prompts for PIN (optionally password) and confirm presence by tapping the Yubikey.
4. You should now be in an SSH session using the newly generated FIDO2 protected SSH key.

### Remove old SSH keys
Once you are 100% confident SSH with the new FIDO2-protected key is working, you can remove key files.

```
rm ~/.ssh/OLDKEY
rm ~/.ssh/OLD.pub
```

### References
- [Securing SSH with the Yubikey](https://developers.yubico.com/SSH/)
