# 在自己的服务器上部署Git服务  
为了让自己的代码使用git来进行版本控制，并且不希望放在GitHub的公开仓库，我决定在自己的云服务器上部署git服务。  

在服务器上部署git服务，方式有多种，我选择了一种比较简单并且适合小团队使用的方式：创建一个git用户。  

## Step 1:添加git用户  
```bash  
ubuntu@VM-252-55-ubuntu:~$ sudo adduser git
[sudo] password for ubuntu:
Adding user `git' ...
Adding new group `git' (1000) ...
Adding new user `git' (1000) with group `git' ...
The home directory `/home/git' already exists.  Not copying from `/etc/skel'.
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for git
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
ubuntu@VM-252-55-ubuntu:~$  
```  

## Step 2:添加用户的shh公钥到 /home/git/.ssh/authorized_keys  
我本地的机器使用的是WSL，所以我先把WSL的ssh公钥复制到云服务器：  
```bash  
scp /home/stupidl/.ssh/id_rsa.pub ubuntu@123.206.221.113  
```  
上面 **ubuntu@123.206.221.113** 中的 **Ubuntu** 是云服务器上的一个用户，我先把本地的ssh public key发送到云服务器，再将该公钥放到 **/home/git/.ssh/authorized_keys** 中去。  
```bash  
ubuntu@VM-252-55-ubuntu:~$ sudo su
root@VM-252-55-ubuntu:/home/ubuntu# cd
root@VM-252-55-ubuntu:~# cat /home/ubuntu/id_rsa.pub > /home/git/.ssh/authorized_keys
root@VM-252-55-ubuntu:~# cat /home/git/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC0B+XGeDivsGpv0ZMHUWyuP/LryFmZWmq8AW5UnglVGceUcNkVUs3xvsyz29PbFTNRnoi+LCALrQEMIZ0R0iNR1ls/CxFbXKzsesxfaMgwTAoMYuBd6EgXeCp8D3JSxN4afVJxkwu38LqJXlU2dYhTtXySGrYjMal4BYPqKCC7Cxxs6WdlxwlwnSl+BQ+EWcWaxjoE2SNBEvo22t4gjbRB7neCqvMOkBHJzxh1PtEX40LG6sRm1gCuY6Nkl3jOm1ljUiMlZl024iXpsqU2qNgUVMnFnDBBOzFBu/T1di/KTUIRHsxaJVBwNb3TM7A9dpea5K3M0ZhJZO2Pe8N3FWP/pjxe4y+J6sCqmQ4hbNF4s2P9RjdERdMiSsq4Kf6jega/81BJg2ucKPUWornEoeswf8BAqmWweNdZ3efgHus2S6AEKSU7TLkWoNDAZkiKMKrDZLvjeCrV+ZdWA8rA7soNTqr0XhWC415CTxs8nCclf7i94yx0yr1Khz71iM0xw0jGXnmzELXjmpkbOJQZA80XD4Sss8cq6+iKY0vUUIZTwPJkKR421j3iex8ci7Zd5QRaH6XomXBwPFCtefMHRdpGAAjfATVpkc6CN6uaPMM7sto+8GeuB9N7wniqYHR8KaOAX0lTT9+aM5FNZzIQ8Jw== 562117676@qq.com
root@VM-252-55-ubuntu:~#  
```  
上述ssh public key我肯定是改过的，哈哈。  
你会发现这个过程，和在云主机的控制台，为你的云服务器添加ssh登录方式时候，添加ssh pub key的过程是一样的。也就是说，我在上面的过程完成之后，我再WSL里面完全可以通过ssh登录到云服务器的git用户：  

```bash  
stupidl@StupidL:~$ ssh git@123.206.221.113
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

*** System restart required ***
Last login: Mon Apr 17 12:59:17 2017 from 124.76.13.127
git@VM-252-55-ubuntu:~$
```  

## Step 3:创建Git仓库目录  

```bash  
ubuntu@VM-252-55-ubuntu:/home/git$ sudo mkdir server
ubuntu@VM-252-55-ubuntu:/home/git$ cd server/
ubuntu@VM-252-55-ubuntu:/home/git/server$ sudo git init --bare Hello.git
Initialized empty Git repository in /home/git/server/Hello.git/
ubuntu@VM-252-55-ubuntu:/home/git/server$ ls
Hello.git
ubuntu@VM-252-55-ubuntu:/home/git/server$ cd Hello.git/
ubuntu@VM-252-55-ubuntu:/home/git/server/Hello.git$ ls
HEAD  branches  config  description  hooks  info  objects  refs
```  

将文件的所有者改成git用户：  
```bash  
ubuntu@VM-252-55-ubuntu:/home/git$ sudo chown -R git:git server
ubuntu@VM-252-55-ubuntu:/home/git$ cd server
ubuntu@VM-252-55-ubuntu:/home/git$ sudo chown -R git:git Hello.git
```  

## Step 4:clone仓库  

接下来，就可以在WSL里clone项目了：  
```bash  
stupidl@StupidL:~$ git clone git@123.206.221.113:/home/git/server/Hello.git
Cloning into 'Hello'...
warning: You appear to have cloned an empty repository.
Checking connectivity... done.
stupidl@StupidL:~$ cd Hello/
stupidl@StupidL:~/Hello$ ls
stupidl@StupidL:~/Hello$ vim Hello.java
stupidl@StupidL:~/Hello$
stupidl@StupidL:~/Hello$
stupidl@StupidL:~/Hello$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        Hello.java

nothing added to commit but untracked files present (use "git add" to track)
stupidl@StupidL:~/Hello$ git add .
stupidl@StupidL:~/Hello$ git commit -m "create Hello.java"
[master (root-commit) c016083] create Hello.java
 1 file changed, 6 insertions(+)
 create mode 100644 Hello.java
 stupidl@StupidL:~/Hello$ git push -u origin master
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 306 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@123.206.221.113:/home/git/server/Hello.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
stupidl@StupidL:~/Hello$  
```  
上述过程就是：clone仓库，创建了一个文件Hello.java，并且提交、推送到了服务器。  

接下来，我们把本地的仓库删除，再重新clone试试：  

```bash  
stupidl@StupidL:~$ rm -rf Hello/
stupidl@StupidL:~$ git clone git@123.206.221.113:/home/git/server/Hello.git
Cloning into 'Hello'...
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.
Checking connectivity... done.
stupidl@StupidL:~$ cd Hello/
stupidl@StupidL:~/Hello$ ls
Hello.java
stupidl@StupidL:~/Hello$ git log
commit c0160832eb9a982c1a760ca0d125ccc993ffa037
Author: StupidL <562117676@qq.com>
Date:   Mon Apr 17 13:28:20 2017 +0800

    create Hello.java
stupidl@StupidL:~/Hello$  
```  

至此，功能已经实现了。  

最后我们看看服务器端的项目结构吧：  

```bash  
ubuntu@VM-252-55-ubuntu:/home/git$ tree server/
server/
`-- Hello.git
    |-- HEAD
    |-- branches
    |-- config
    |-- description
    |-- hooks
    |   |-- applypatch-msg.sample
    |   |-- commit-msg.sample
    |   |-- post-update.sample
    |   |-- pre-applypatch.sample
    |   |-- pre-commit.sample
    |   |-- pre-push.sample
    |   |-- pre-rebase.sample
    |   |-- prepare-commit-msg.sample
    |   `-- update.sample
    |-- info
    |   `-- exclude
    |-- objects
    |   |-- c0
    |   |   `-- 160832eb9a982c1a760ca0d125ccc993ffa037
    |   |-- ef
    |   |   `-- 3da929dd3b06dfbaedcfe29b3e5600b5723511
    |   |-- f1
    |   |   `-- 6e19d7f2c3b9e9472954c269821b87d3f77ba9
    |   |-- info
    |   `-- pack
    `-- refs
        |-- heads
        |   `-- master
        `-- tags

13 directories, 17 files
ubuntu@VM-252-55-ubuntu:/home/git$  
```  
希望对你有参考帮助。谢谢！  
