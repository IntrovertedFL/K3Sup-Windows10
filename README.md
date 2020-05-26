# K3Sup Windows10/Server 2019

### Pre-requisite :

* #### [Visual Studio Code](https://code.visualstudio.com/download)
* #### [openssh for windows ](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
* #### [k3sup.exe](https://github.com/alexellis/k3sup/releases)
* #### [Chocolatey](https://chocolatey.org/install) (Windows Package Manager)
* #### [kubectl for windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
* #### 3 Virtual Machines

# 1. Install/Configure 3 VM's (I have created 3 Ubuntu 18.04 Machines Using [Proxmox](https://www.proxmox.com/en/))

* #### Configure hosts

```
nano /etc/hosts

Sample

127.0.0.1       localhost
192.168.254.88  master.tribestudios.io  master

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes ff02::2 ip6-allrouters
192.168.254.89 - node01.tribestudios.io node01
192.168.254.91 - node02.tribestudios.io node02
192.168.254.92 - node03.tribestudios.io node03
```

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
reboot and make sure it's off.
do "free -h" and make sure swap says "0"
```



# 2. Install/Configure OpenSSH on Windows Server 2019/Windows 10 1809

### A. Install OpenSSH

* #### Via Windows GUI


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse 

* #### Via Powershell


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse#installing-openssh-with-powershell 

### B. Generate SSH Key

* #### Open Visual Studio Code/Set default shell GitBash

```
aaron@Shed-Lappy MINGW64 ~
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/aaron/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /c/Users/aaron/.ssh/id_rsa
Your public key has been saved in /c/Users/aaron/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:J3cCEe+05wZ5c0LF+BWCZ2NSnV7NI+2dMNvyzH9q4EY aaron@Shed-Lappy
The key's randomart image is:
+---[RSA 3072]----+
|        o.  o=+o+|
|         o o.Xo==|
|        . o *.X.=|
|         + + o.=.|
|        S O * *  |
|         + BE+ + |
|           oo.  .|
|           .o . o|
|           . ....|
+----[SHA256]-----+
```

### C. Copy key to all 3 Machines/Also in GitBash

```
ssh-copy-id aaron@192.168.254.88
```
Example Output

```
aaron@Shed-Lappy MINGW64 ~
$ ssh-copy-id aaron@192.168.254.88
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/aaron/.ssh/id_rsa.pub"
The authenticity of host '192.168.254.88 (192.168.254.88)' can't be established.
ECDSA key fingerprint is SHA256:GE2bt679dcjb0X+Qo0Z33I54QDE8BVf8crSkhefcRTQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? Yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
aaron@192.168.254.88's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'aaron@192.168.254.88'"
and check to make sure that only the key(s) you wanted were added.
```



### D. Disable Password Authentication on your Server (Optional)

```
sudo nano /etc/ssh/sshd_config

Change line #PasswordAuthentication Yes to

PasswordAuthentication no

sudo service ssh restart
```





