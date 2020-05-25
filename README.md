# K3Sup-Windows10

Environment
OS: Windows 10
Pre-requisite :

kubectl for windows (download from kubernetes.io)

openssh for windows (see Microsoft documentation)

k3sup (download k3sup.exe from Release Page)

Infrastructure setup:
Provision (1 or 2) new virtual machine running a compatible operating system such as Ubuntu, Debian, Raspbian, or something else.
With 1 VM you can just run k3s.
With 2 you can join the second VM to the first as a worker node.

The rest of this document assumes you have 2 VM's with ubuntu as the user and IP's as 192.168.83.128, 192.168.83.129

Note : ssh-copy-id is not available on openssh for windows. You generate your private/public key and then copy the public key over manually to the linux device.

To generate a new key on windows
ssh-keygen

Now lets copy it over from windows to the remote device (will ask for password this time)

scp c:\user\<your_name>\.ssh\id_rsa.pub ubuntu@192.168.83.129:/home/ubuntu/key.pub

On the remote linux device - logged in as ubuntu

ubuntu@vm1:~$ cat key.pub >>~/.ssh/authorized_keys
ubuntu@vm1:~$ rm key.pub
Repeat on second VM (if required)

ubuntu@vm2:~$ cat key.pub >>~/.ssh/authorized_keys
ubuntu@vm2:~$ rm key.pub
Test from windows by
ssh ubuntu@192.168.83.128

Aim is that you can login without requiring a password.

Finally you need to ensure you can use sudo on the linux device without requiring a password.
This requires running the following command on the linux device(s)
sudo visudo

and adding the following text to the end (which enables the ubuntu user to use sudo without password)
ubuntu ALL=(ALL) NOPASSWD: ALL

Run k3sup :
set VM1=192.168.83.128
set USER=ubuntu 
k3sup install --ip=%VM1% --user=%USER%
All this going well - you should now get a lot of text followed by

Saving file to: C:\Users\<your_name>\k3sup\upstream\kubeconfig

Set your environment variable using path from previous command

set KUBECONFIG=C:\Users\<your_name>\k3sup\upstream\kubeconfig

Now test with kubectl and you should get remote access to k3s on that remote VM