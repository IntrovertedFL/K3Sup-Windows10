# K3Sup Windows10/Server 2019

### Pre-requisite :

* #### [openssh for windows ](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
* #### [k3sup.exe](https://github.com/alexellis/k3sup/releases)
* #### [kubectl for windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
* #### [Chocolatey](https://chocolatey.org/install) (Windows Package Manager)
* #### 3 Virtual Machines

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
SSH into the first machine as "User"

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

Don't forget to do same for all the other nodes
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

Open the Search, type in “env”, Click on "Edit Enviroment Variables for **Your Account**"

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

# 4. Install/configure kubectl - [chocolatey required](https://chocolatey.org/install) 

```
choco install kubernetes-cli
cd c:/Users/username
mkdir .kube
cd .kube
New-Item config -type file
```

# 5. Now all the fun we worked so hard for. 

Sample hosts

```
192.168.254.69 - master (server ip)
192.168.254.70 - node1
192.168.254.71 - node2
```


* #### Boostrap Master

``` 
k3sup install --ip=192.168.254.69 --user=username
```
* #### Copy kubeconfig 

```
Can click search, type notepad, right click notepad and run as administrator
Click "File" and open C:\Users\me\.kube\config
Paste in Kube config, save and open powershell
kubectl get node -o wide
```

* #### Add Worker Nodes

```
k3sup join --ip=192.168.254.70 --server-ip=192.168.254.69 --user=username

k3sup join --ip=192.168.254.71 --server-ip=192.168.254.69 --user=username
```

* #### Check Again

```
PS C:\Users\me> kubectl get node -o wide
NAME     STATUS   ROLES    AGE   VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master   Ready    master   54m   v1.17.2+k3s1   192.168.254.69   <none>        Ubuntu 18.04.4 LTS   4.15.0-101-generic   containerd://1.3.3-k3s1
node01   Ready    <none>   12m   v1.17.2+k3s1   192.168.254.70   <none>        Ubuntu 18.04.4 LTS   4.15.0-101-generic   containerd://1.3.3-k3s1
node02   Ready    <none>   11m   v1.17.2+k3s1   192.168.254.71   <none>        Ubuntu 18.04.4 LTS   4.15.0-101-generic   containerd://1.3.3-k3s1
````

# 6. Download/Configure Arkade

```
No need to configure.
Download Arkade.exe to the folder we created C:\K3Sup
Now go back to Powershell and

PS C:\Users\me> arkade
            _             _      
  __ _ _ __| | ____ _  __| | ___
 / _` | '__| |/ / _` |/ _` |/ _ \
| (_| | |  |   < (_| | (_| |  __/
 \__,_|_|  |_|\_\__,_|\__,_|\___|

Get Kubernetes apps the easy way

Usage:
  arkade [flags]
  arkade [command]

Available Commands:
  help        Help about any command
  info        Find info about a Kubernetes app
  install     Install Kubernetes apps from helm charts or YAML files
  update      Print update instructions
  version     Print the version

Flags:
  -h, --help   help for arkade

Use "arkade [command] --help" for more information about a command.


PS C:\Users\me>arkade install -h

Install Kubernetes apps from helm charts or YAML files using the "install" 
command. Helm 3 is used by default unless you pass --helm3=false, then helm 2
will be used to generate YAML files which are applied without tiller.

You can also find the post-install message for each app with the "info"
command.

Usage:
  arkade install [flags]
  arkade install [command]

Examples:
  arkade install
  arkade install openfaas --helm3 --gateways=2
  arkade install inlets-operator --token-file $HOME/do-token

Available Commands:
  argocd                  Install argocd
  cert-manager            Install cert-manager
  chart                   Install the specified helm chart
  cron-connector          Install cron-connector for OpenFaaS
  crossplane              Install Crossplane
  docker-registry         Install a Docker registry
  docker-registry-ingress Install registry ingress with TLS
  grafana                 Install grafana
  info                    Find info about a Kubernetes app
  ingress-nginx           Install ingress-nginx
  inlets-operator         Install inlets-operator
  istio                   Install istio
  jenkins                 Install jenkins
  kafka-connector         Install kafka-connector for OpenFaaS
  kube-state-metrics      Install kube-state-metrics
  kubernetes-dashboard    Install kubernetes-dashboard
  linkerd                 Install linkerd
  metrics-server          Install metrics-server
  minio                   Install minio
  mongodb                 Install mongodb
  openfaas                Install openfaas
  openfaas-ingress        Install openfaas ingress with TLS
  portainer               Install portainer to visualise and manage containers
  postgresql              Install postgresql
  tekton                  Install Tekton pipelines and dashboard
  traefik2                Install traefik2

Flags:
  -h, --help                help for install
      --kubeconfig string   Local path for your kubeconfig file (default "kubeconfig")
      --wait                If we should wait for the resource to be ready before returning (helm3 only, default false)

Use "arkade install [command] --help" for more information about a command.
```


# Download/Configure Inlets Pro

```
No need to configure.
Download inlets-pro.exe to the folder we created C:\K3Sup
Now go back to Powershell and

PS C:\Users\me> inlets-pro
 _       _      _            _
(_)_ __ | | ___| |_ ___   __| | _____   __
| | '_ \| |/ _ \ __/ __| / _` |/ _ \ \ / /
| | | | | |  __/ |_\__ \| (_| |  __/\ V /
|_|_| |_|_|\___|\__|___(_)__,_|\___| \_/

PRO edition


        Tunnel TCP traffic at L4

Usage:
  inlets-pro [flags]
  inlets-pro [command]

Available Commands:
  client      Start the tunnel client.
  help        Help about any command
  server      Start the tunnel server.
  version     Display the clients version information.

Flags:
  -h, --help   help for inlets-pro

Use "inlets-pro [command] --help" for more information about a command.

PS C:\Users\me> inlets-pro server -h
Start the tunnel server.

Usage:
  inlets-pro server [flags]

Examples:
  inlets server \
    --auto-tls \
    --common-name PUBLIC_IP \
    --listen :8123 \
    --token AUTH_TOKEN

Flags:
      --auto-tls               Generate a TLS CA and cert (default true)
      --auto-tls-path string   Path for self-signed TLS CA
      --common-name string     Common Name for TLS certificate
      --debug                  Debug logging
  -h, --help                   help for server
      --id string              Peer ID - leave as default
      --listen string          Listen address for the control-port (control-plane) (default ":8123")
      --peers string           Peers format id:token:url,id:token:url  - leave as default
      --remote-tcp string      Remote host to forward to on client's network (default "192.168.0.35")
      --token string           Inlets authentication token for control-plane


PS C:\Users\me> inlets-pro client -h
Start the tunnel client.

Note: You can pass the --token argument followed by a token value to both the server and client to prevent unauthorized connections to the tunnel.

Usage:
  inlets-pro client [flags]

Examples:

  inlets client \
    --connect wss://REMOTE_IP:8123 \
    --token AUTH_TOKEN \
    --tcp-ports 80,443 \
    --license LICENSE

Flags:
      --auto-tls              Toggle use of self-signed TLS CA, disable if providing own TLS (default true)
      --connect string        Address of control-plane for server i.e. wss://localhost:8123/connect (default "ws://localhost:8123/connect")
      --debug                 Debug logging
      --generate string       Generate a service file for one of the following: systemd
  -h, --help                  help for client
      --id string             Peer ID for client - leave as default (default "inlets-client")
      --license string        Your JWT license for inlets pro
      --license-file string   JWT license for inlets pro from a file
      --tcp-ports string      TCP ports to open on server, comma separated (default "80,443")
      --token string          Token for authentication
```



