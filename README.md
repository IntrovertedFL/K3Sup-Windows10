# K3Sup Windows10/Server 2019

### Pre-requisite :

* #### [openssh for windows ](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
* #### [k3sup.exe](https://github.com/alexellis/k3sup/releases)
* #### [kubectl for windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)

# 1. Install/Configure 3 VM's (I have created 3 Ubuntu 18.04 Machines Using [Proxmox](https://www.proxmox.com/en/))

* #### Enable Passwordless Sudo

```
sudo nano /etc/sudoers
Find the line which contains #includedir /etc/sudoers.d
Below that line add: username ALL=(ALL) NOPASSWD: ALL
```

* #### Disable Swap

```
swapoff -a
nano /etc/fstab
Comment out line "#/dev/mapper/master--vg-swap_1 none            swap    sw              0       0"
reboot and make sure it's off!! It doesn't play well with kubernetes.
do "free -h" and make sure swap says "0"
```



# 2. Install/Configure OpenSSH on Windows Server 2019/Windows 10 1809

### A. Install OpenSSH

* #### Via Windows GUI


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse 

* #### Via Powershell


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse#installing-openssh-with-powershell 

### B. Generate SSH Key

* #### Open Elevated Powershell (Note the key's location C:\Users\me/.ssh/)

```
PS cd C:\
PS C:\ mkdir K3Sup
PS cd C:\K3Sup
PS C:\K3Sup> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\me/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\me/.ssh/id_rsa.
Your public key has been saved in C:\Users\me/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:JzBE+x5mKPpflttEQYTtrno6ZOACheVlXKEVpvB8SHM me@my-Lappy
The key's randomart image is:
+---[RSA 3072]----+
| oo.=oEo +o      |
|...B.X ....      |
| .. * =  ..      |
|.   .. =  ..     |
| . .... S.o      |
|  ....o+ *.      |
|  .. o  =..      |
|   .  .oo+       |
|    ..+=. .      |
+----[SHA256]-----+
PS C:\K3Sup>
```

### C. Copy key to all 3 Machines

* #### Option 1. Use SCP (Didn't work for me)

```
scp ~/.ssh/id_rsa.pub me@192.168.254.69:~/.ssh/authorized_keys
```
* #### Option 2. Manually Copy 

```
SSH into the first machine
Make sure you are not root

sudo mkdir ~/.ssh
sudo chmod 700 ~/.ssh
sudo nano ~/.ssh/authorized_keys

Copy and paste key from C:\Users\me/.ssh/id_rsa.pub (Can Open With Notepad)

Paste key into "~/.ssh/authorized_keys"

Do "Ctrl o" 
Press enter & "Ctrl x"

sudo chmod 600 ~/.ssh/authorized_keys
sudo chown -R $(whoami):$(whoami) ~/.ssh/
logout

Should be able to ssh username@192.168.254.69 from Powershell now
```
### D. Disable Password Authentication on your Server (Optional)

```
sudo nano /etc/ssh/sshd_config

Change line #PasswordAuthentication Yes to

PasswordAuthentication no

sudo service ssh restart
```

# 3. Install and Configure K3Sup.exe

* #### Download K3Sup.exe

[Get Latest Here](https://github.com/alexellis/k3sup/releases)

```
Download K3Sup.exe to the folder we created C:\K3Sup

Open the Search, type in “env”, Click on "Edit Eviroment Variables for **Your Account**"

At Top in "User Varibles" Click on the "Path" Variable line and select Edit.

On right Select "Browse" and select the folder we created that you added K3Sup.exe too.
```
```
Now Open Powershell and * Note may need to close and reopen Powershell to take effect.

PS cd C:\K3Sup
PS C:\Users\me> k3sup version
←[31m _    _____
| | _|___ / ___ _   _ _ __
| |/ / |_ \/ __| | | | '_ \
|   < ___) \__ \ |_| | |_) |
|_|\_\____/|___/\__,_| .__/
                     |_|
←[0mVersion: 0.9.2
Git Commit: 5a636dba10e1f8e6bb4bb5982c6e04fc21c34534
```

# 4. Install/configure kubectl

* #### Option 1. Requires Chocolatey - [Install Here](https://chocolatey.org/install) 

```
choco install kubernetes-cli
kubectl version --client
```

* #### Option 2. Install manually

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe
kubectl version --client
```


