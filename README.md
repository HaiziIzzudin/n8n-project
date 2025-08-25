# n8n-project

## setup termux for pubkey auth

Looking for debian setup instead? [click here](#for-debian-setup-assuming-you-have-clean-slate)

Here’s a straightforward way to set up **passwordless SSH login from Windows → Termux** so you won’t need to enter your password each time. I’ll summarize the process using the [Termux Wiki](https://wiki.termux.com/wiki/Remote_Access) and a practical [GitHub guide](https://gist.github.com/evandrocoan/f503188587587d7b1d1ba8746c9c6107):

### Steps

1. **Install SSH support in Termux (Android side)**  
   Open Termux and install OpenSSH:  
   ```bash
   pkg upgrade
   pkg install openssh
   ```
   Start the SSH server (listens on port `8022` by default):  
   ```bash
   sshd
   ```

2. **Generate an SSH key on your Windows machine**  
   On Windows 10/11 you can use PowerShell:  
   ```powershell
   ssh-keygen -t rsa -b 2048
   ```
   Press enter to save it as the default (`C:\Users\YourName\.ssh\id_rsa`). Leave passphrase empty if you truly want no prompts.

3. **Copy your public key to Termux**  
   From Windows, run (replace `PHONE_IP` with the LAN IP of your Android):  
   ```powershell
   scp -P 8022 C:\Users\YourName\.ssh\id_rsa.pub u0_aXXX@PHONE_IP:~/.ssh/authorized_keys
   ```
   Notes:  
   - Create the `.ssh` folder in Termux first if needed: `mkdir -p ~/.ssh`  
   - Replace `u0_aXXX` with the actual Termux user (Termux internally uses these, but usually you can just put any user and it will default to Termux’s one).  

   Alternatively, you can copy-paste the contents of `id_rsa.pub` into `~/.ssh/authorized_keys` inside Termux.

4. **Fix permissions in Termux**  
   In Termux, run:  
   ```bash
   chmod 700 ~/.ssh && \
   chmod 600 ~/.ssh/authorized_keys
   ```

5. **(Optional but recommended) Disable password authentication**  
   Edit Termux’s sshd config:  
   ```bash
   nano $PREFIX/etc/ssh/sshd_config
   ```
   Set:
   ```
   PasswordAuthentication no
   PubkeyAuthentication yes
   ```
   Restart the ssh server:  
   ```bash
   pkill sshd && sshd
   ```

6. **Connect without password from Windows**  
   Now you should be able to connect directly:  
   ```powershell
   ssh -p 8022 PHONE_IP
   ```

   It will log in using the private key (`id_rsa`) and won’t prompt for a password anymore.

---

### References
- [Termux Wiki: Remote Access / Public Key Auth](https://wiki.termux.com/wiki/Remote_Access)  
- [SSH Termux Desktop connectivity guide (GitHub)](https://gist.github.com/evandrocoan/f503188587587d7b1d1ba8746c9c6107)


### When you try to login, it will error for a bit, something like: `Permission denied`. Fix:

### 1. Use your **`~/.ssh/config`** file
Create or edit the config file at:

```
C:\Users\<YourName>\.ssh\config
```

Add an entry like this ([Stack Overflow suggestion](https://stackoverflow.com/questions/84096/setting-the-default-ssh-key-location)):

```
Host phone
    HostName 192.168.0.50
    Port 8022
    User u0_aXXX
    IdentityFile C:/Users/<YourName>/.ssh/id_rsa (pls locate the right id_rsa file)
```

Now you can just run:
```powershell
ssh phone
```
It’ll automatically pick the right port, user, and key.


## transfer/ copy file to the remote machine

```
scp -P 8022 "file.txt" phone:~
```




## for debian setup (assuming you have clean slate)

follow generate key on windows host like above, but don't transfer yet. Your remote debian most likely don't have .ssh folder. Create first

```bash
ssh user@10.147.XX.XXX -p 8022
```

then

```bash
mkdir ~/.ssh && \
chmod 700 ~/.ssh
```

then

```bash
exit
```

then, use command below to transfer pubkey to your remote debian system
```
scp -P 8022 C:\Users\<user>\.ssh\id_rsa.pub user@10.147.XX.XXX:~/.ssh/authorized_keys
```

Try login. Should be there is no prompt for password anymore and you will log in.





## Changing the SSH Port on Fedora

To enhance security, changing the default SSH port from **22** to a different port is a common practice. Here’s how to do it on a Fedora system.

### Step-by-Step Instructions

1. **Connect to Your Server**: Use SSH to log in to your server as the root user or a user with sudo privileges.

2. **Backup the Configuration File**: Before making changes, it's wise to back up the SSH configuration file.
   ```bash
   sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
   ```

3. **Edit the SSH Configuration File**: Open the SSH daemon configuration file in a text editor.
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Locate the line that says `#Port 22` and change it to your desired port number (e.g., `Port 8022`). Remove the `#` to uncomment the line.

4. **Edit the SSH Socket Configuration**: You also need to modify the SSH socket configuration.
   ```bash
   sudo systemctl edit sshd.socket
   ```
   In the editor, add or modify the following lines in between where it says add in this area (be aware to not just uncomment since settings below certain comment line will be discarded):
   ```
   ListenStream=
   ListenStream=8022
   ```
   This clears the old port and sets the new one.

5. **Adjust Firewall Rules**: If you have a firewall enabled (like `firewalld`), you need to allow the new port and remove the old one.
   ```bash
   sudo firewall-cmd --permanent --add-port=8022/tcp
   sudo firewall-cmd --permanent --remove-port=22/tcp
   sudo firewall-cmd --reload
   ```

6. **Update SELinux Configuration**: If SELinux is enabled, you need to allow the new port.
   ```bash
   sudo semanage port -a -t ssh_port_t -p tcp 8022
   ```

7. **Restart the SSH Service**: Apply the changes by restarting the SSH daemon.
   ```bash
   sudo systemctl restart sshd
   ```

8. **Test the New SSH Port**: Before logging out of your current session, open a new terminal and try connecting to the server using the new port:
   ```bash
   ssh username@your_server_ip -p 8022
   ```

### Important Notes
- Ensure that the new port you choose is not already in use by another service.
- Always test the new configuration before closing your current SSH session to avoid being locked out.
- If you encounter issues, you can revert to the backup configuration file you created earlier.

By following these steps, you can successfully change the SSH port on your Fedora server, enhancing its security against unauthorized access.
