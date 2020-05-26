# Multi-master HA Kubernetes using K3Sup on Windows10/Server 2019 (video tutorial in the works)

### Software used:

* #### [Visual Studio Code](https://code.visualstudio.com/download)
* #### [openssh for windows ](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
* #### [k3sup.exe](https://github.com/alexellis/k3sup/releases)
* #### [Chocolatey](https://chocolatey.org/install) (Windows Package Manager)
* #### [kubectl for windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
* #### 3 Virtual Machines

# 1. Install K3Sup.exe

```
In this tutorial I created a folder K3sup @ C:\Users\username\k3sup\

With Powershell (change username to you & take notice of version of k3sup 0.9.2)

Invoke-WebRequest https://github.com/alexellis/k3sup/releases/download/0.9.2/k3sup.exe -OutFile C:\Users\username\k3sup\K3sup.exe
```

```
Open the Search, type in “env”, Click on "Edit Enviroment Variables for **Your Account**"

At Top in "User Varibles" Click on the "Path" Variable line and select Edit.

On right Select "Browse" and select the folder we created that you added K3Sup.exe too.
```
```
PS C:\Users\username> k3sup
 _    _____
| | _|___ / ___ _   _ _ __  
| |/ / |_ \/ __| | | | '_ \ 
|   < ___) \__ \ |_| | |_) |
|_|\_\____/|___/\__,_| .__/ 
                     |_|    
Usage:
  k3sup [flags]
  k3sup [command]

Available Commands:
  app         Install Kubernetes apps from helm charts or YAML files
  help        Help about any command
  install     Install k3s on a server via SSH
  join        Install the k3s agent on a remote host and join it to an existing server
  update      Print update instructions
  version     Print the version

Flags:
  -h, --help   help for k3sup
```

# 2. Install/Configure 3 VM's (I have created 3 Ubuntu 18.04 Machines)

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

# 3. Install/Configure OpenSSH on Windows Server 2019/Windows 10 1809

### A. Install OpenSSH

* #### Via Windows GUI


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse 

* #### Via Powershell


> https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse#installing-openssh-with-powershell 

### B. Install Git, vscode & Kubectl - (install choco here - https://chocolatey.org/install)

```
Make sure to run powershell as administrator

choco install git -y
choco install kubernetes-cli -y
choco install vscode -y
```
**Might need to reboot after all that.**

### C. Generate SSH Key

* #### Open Visual Studio Code/Set default shell GitBash

```
username@Shed-Lappy MINGW64 ~
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/username/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /c/Users/username/.ssh/id_rsa
Your public key has been saved in /c/Users/username/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:J3cCEe+05wZ5c0LF+BWCZ2NSnV7NI+2dMNvyzH9q4EY username@Shed-Lappy
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
ssh-copy-id username@192.168.254.88
```
**Example Output**

```
username@Shed-Lappy MINGW64 ~
$ ssh-copy-id username@192.168.254.88
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/username/.ssh/id_rsa.pub"
The authenticity of host '192.168.254.88 (192.168.254.88)' can't be established.
ECDSA key fingerprint is SHA256:GE2bt679dcjb0X+Qo0Z33I54QDE8BVf8crSkhefcRTQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? Yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
username@192.168.254.88's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'username@192.168.254.88'"
and check to make sure that only the key(s) you wanted were added.
```
### D. Disable Password Authentication on your Server (Optional)

```
sudo nano /etc/ssh/sshd_config

Change line #PasswordAuthentication Yes to

PasswordAuthentication no

sudo service ssh restart
```
# 4. Now all the fun we worked so hard for.

* #### Create bootstrap.sh

**Again I created a folder for k3sup for simplicity**

```
username@Shed-Lappy MINGW64 ~
$ cd k3sup

username@Shed-Lappy MINGW64 ~/k3sup
$ touch bootstrap.sh
```
**Paste into bootstrap.sh** (make sure to change ip's)

```
#!/bin/bash
set -e

export NODE_1="192.168.254.88"
export NODE_2="192.168.254.89"
export NODE_3="192.168.254.91"
export USER=root

# The first server starts the cluster
k3sup install \
  --cluster \
  --user $USER \
  --ip $NODE_1

# The second node joins
k3sup join \
  --server \
  --ip $NODE_2 \
  --user $USER \
  --server-user $USER \
  --server-ip $NODE_1

# The third node joins
k3sup join \
  --server \
  --ip $NODE_3 \
  --user $USER \
  --server-user $USER \
  --server-ip $NODE_1
  ```
  **Make sure we are remaining in gitbash**
  
  ```
  username@Shed-Lappy MINGW64 ~/k3sup
$ ./bootstrap.sh
  ```
  
  Once that is complete.
  
  ```
  username@Shed-Lappy MINGW64 ~/k3sup
$ export KUBECONFIG=`pwd`/kubeconfig

username@Shed-Lappy MINGW64 ~/k3sup
$ kubectl get node
NAME     STATUS   ROLES    AGE   VERSION
node01   Ready    master   54s   v1.17.2+k3s1
master   Ready    master   81s   v1.17.2+k3s1
node02   Ready    master   35s   v1.17.2+k3s1
```

**Have Fun :)**