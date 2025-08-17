# n8n-project

## setup termux for pubkey auth

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
   chmod 700 ~/.ssh
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

scp -P 8022 "file.txt" phone:~
