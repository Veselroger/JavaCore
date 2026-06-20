# [←](../README.md) <a id="home"></a> Linux commands

## Table of content:
- [WSL sandbox](#sandbox)
- [Process Status](#ps)
- [Disc Status](#disc)
- [Network](#network)

----

## [↑](#Home) <a name="sandbox"></a> WSL sandbox
To make things simple for Windows, we can use WSL as a sandbox.\
Please check the **"[How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)"** guide.

We need to open the WSL terminal to configure SSH server.\
Such SSH server should be configured. For cloud environment SSH server is usually preinstalled already.

Let's connect to the WSL:
> wsl -d Ubuntu

Now we can install openssh server:
> sudo apt update
sudo apt install openssh-server

Then, we should start it:
> sudo service ssh start
ps aux | grep ssh

Now we need to check the WSL IP address that is available from the host machine:
> hostname -I

This IP address should be used to connect with SSH client from Windows HOST machine to Linux WSL machine.\
⚠️ WSL terminal should be opened while we are working through the SSH client with WSL machine. In other case it will be stopped.

----

## [↑](#Home) <a name="ps"></a> Process Status

![](../img/tools/psss.png)

Linux has a tool to get the **P**rocess **S**tatus (``ps``).\
There are a lot of different options. To get more information about them:
> ps --help simple

This tool allows to list running processes.

Each process has its own **Process ID (PID)**.\
We can use the ``kill`` command to send messages to the process.\
The utility's name is misleading. Process termination is only one of the possible signals (used by default).\
For example:
> kill -3 PID

This signal #3 for Java process just prints the thread dump without real killing the process.

Linux is an interesting system. It has the procfs.\
**proc filesystem (procfs)** is a special filesystem in Unix-like operating systems that presents information about processes and other system information.\
For example:
> cat /proc/<PID>/status

There can be a lot of differet information. We can use grep to grab some specific information:
> cat /proc/2344/status | grep Name

Ther are differet aspects that we can get. Because it's behave as a file system, we can search for such thigs like for files:
> ls /proc/2344

When we know the process ID we can get the information from the **Table Of Processes (top)**:
> top -p <PID>

----

## [↑](#Home) <a name="disc"></a> Disc Status
The disc information can be analyzed by different commands.

The **disc free** information can be obtained by ``df`` command:
> df -h

``-h`` means ``human-readable``.

The **disc usages** information can be obtained by ``du`` command:
>  du -h /var/log | sort -h

The current directory file list can be seen by ``ls`` command:
> ls -l

``-l`` means Long format.

For each file there is a permissions information. For example:
```
drwx------
```

The first character define item type: (d)irectory, (-)file, symbolic (l)ink.\
Then we have a permissions info: ```[owner][group][others]```.\
Available permissions are: r - read, w - write, x - execute.

This information can be changed by ``chmod`` tool:
> chmod 755 file.txt

There is a simple mask: 1 for execute, 2 for write, 4 for read.\
7 means: execute + write + read.\
5 means: read + execute (BUT NOT write).

⚠️ Only file owner or root can change permissions for the file.

There is a tool to change the owner: ``chown`` (**ch**ange **ow**ner).\
But it can be launched ONLY from the root.\
For this purposes it should be called from the ``sudo`` (**s**witch **u**ser and **do**):
> sudo chown -R java:java /opt/app

There is a specific file ``/etc/sudoers`` that controls which users and which actions can do through the ``sudo`` tool.\
Also, sudo permissions can be provided to the users group.

There is a possibility to add user to the group:
> sudo usermod -aG wheel alice

Usually, wheel (as "big wheel" - like important person) group is a system group that is mentioned in the sudoers.\
So we can add user to this group to be able to use sudo from this user.

By default it's impossible to use several groups for file.\
There is more flexible way to control permissions: **Access Control List (ACL)**:
> getfacl file

Or we can set several groups:
> setfacl -m g:dev:rwx /data/shared
setfacl -m g:qa:rx /data/shared

The ``cat`` command can be used to read files.\
But it reads the whole content. For long files the ``less`` tool should be preferred.

There is a ``more`` tool, but it's less flexible and convenient way to work with files.\
It sounds like: ``less is more`` =) If it's possible it's recommended to use ``less`` tool instead.

``tail`` is a powerful tool for reading data from file:
> tail -f app.log

``-f`` means **follow**. In that case we read data and waiting for new lines.\
By default it reads 10 last rows. But this behavior can be changed by ``-n`` flag.

Also, we can create a pipeline:
> tail -f app.log | grep --line-buffered ERROR

It's important to use ``--line-buffered`` because grep uses buffers to handle data.

----

## [↑](#Home) <a name="network"></a> Network
One of the most useful tool is ``ip`` to work with **Internet Protocol**:
> ip a

``ip a`` shows **ALL** known network interfaces that are registered.

Also, we can check defined **routes** (i.e. how to send packages, through which interfaces):
> ip route

We can check analyze specific routes:
> ip route get 1.1.1.1

Also, ``traceroute`` and ``tracepath`` tools can be used to trace packages path.

To analyze which application are listening ports the ``ss`` tool can be used:
> ss -tulpn

``-t`` means tcp, ``-u`` means udp, ``-l`` means listening, ``-p`` means process, ``-n`` do not resolve names.

The hostname with specific flag ``-I`` can be used to check currently assigned addresses for this host:
> hostname -I
 
To check a specific port for listening applications we can use the ``lsof`` tool:
> sudo lsof -i :9000

Or the ``lsof -p 12345 | grep TCP`` can be used to check information for a specific process.

For example, UDP port 53 can be used for DNS names resolving.\
Someone (like web browser) can send a request to it and it gets the answer to its temporal UDP port.\
It will work fast because UDP doesn't create and hold a connection (that is expensive).

----
