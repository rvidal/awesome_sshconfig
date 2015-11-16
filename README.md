# [Awesome *SSHConfig*](https://github.com/mgarces/awesome_sshconfig)
A curated list of awesome hacks for [SSH_Config](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/ssh_config.5?query=ssh_config); for several years I've improved the use of my ssh_config file, and here I share everything I have obtained so far; I use this on Mac OS X, but should be straighforward to addapt this to Linux.

So far, here is what I have obtained:
* [Awesome *SSHConfig*](https://github.com/mgarces/awesome_sshconfig)
  * [Separated SSH config files](#separated-ssh-config-files)
  * [A sane *default* config file](#a-sane-default-config-file)
  * [ControMaster / ControlPath](#contromaster--controlpath)
  * [PKI Authentication](#pki-authentication)
  * [Tunnels](#tunnels) 
  * [SOCKS Proxy](#socks-proxy)
  * [Jump Hosts](#jump-hosts)
  * [Remote copy](#remote-copy)
  * [Forward Agent](#forward-agent)
  * [Keep Alive](#keep-alive)
  * [Default user defined by context](#default_user)

## Separated SSH config files
OpenSSH does not have an option to read several config files, or
including everything inside a given directory; it only reads options,
either from the command line, from */etc/ssh/ssh_config* or *~/.ssh/config*.

This is fine, if you have a couple of hosts on your *SSHConfig* file, but
if that grows, and you are like me, and have hundreds of hosts, there is
a need to separate config files, to differentiate for example,
**Production** from **Development**.

To obtain this, first create a folder inside your ~/.ssh, lets call it
_configs_; after this, throw your config files inside, using the
*.config* extension, and also add a file with the name _default_.

Now here is the fun part! Add the next aliases to your *.bashrc* (or *.zshrc*, if you are with the cool kids):

```
alias sshconfig="vim ~/.ssh/configs"

alias sshcompile="echo -n >! ~/.ssh/config && cat
~/.ssh/configs/*.config >> ~/.ssh/config && cat ~/.ssh/configs/default
>> ~/.ssh/config"

alias ssh="sshcompile && ssh"
```
The last line is optional, you can comment it if you want to "compile"
*SSHConfig* manually; leave it like this, if you want to "compile"
SSConfig each time you connect to a host (recommended). _sshcompile_
cleans the ~/.ssh/config, concatenates all the *.config* files inside
the directory, and lastly, concatenates the _default_ file to the end of the
config; this way, you get your *SSHConfig* file all organized.

## A sane *default* config file
A good place to start with your **SSHConfig** is to setup a global
default file, which will be added at the end of the final config file. I
usually set all the sane defaults there, and also give some examples in
the form of commentaries, for future generations :). I have provided a
good base in the *configs* directory, so you can start using it right
away.

```
###################################################
############ GLOBAL SETTINGS ######################
###################################################
# default config from 
# https://github.com/mgarces/awesome_sshconfig
###################################################

Host *
  ControlMaster auto
  ControlPath ~/.ssh/tmp/%r@%h:%p
  IdentityFile ~/.ssh/keys/id_rsa
  User myuser
  ForwardAgent yes
  Port 22
  ServerAliveInterval 60
  ServerAliveCountMax 3
  RemoteForward 55555 127.0.0.1:55555
###################################################
#This will force "User root"
Host r.*
  User root
###################################################
# EXAMPLE FORWARDS
# Put remote service, on local port
# EX: bring remote 8080 to local 8080
#
#      LocalForward localhost:8080 localhost:8080
#
# Put local service, on remote port
# EX: push local port 22, to remote 2222
#
#
#     RemoteForward localhost:2222 localhost:22
#
# Create a dynamic tunnel, can be used has
# SOCKS5 proxy
#
#     DynamicForward localhost:7777
#
#
# Now you can configure your browser to use
# localhost:7777 SOCKS5, and your HTTP gateway is the
# remote server
```
I will explain what most of this stuff does in the next sections, for
now, just notice the __Host *__ declaration; this affects all hosts,
and its overwritten if you declare it inside a given host or on the
command line. For example, imagine if you are connecting to a server,
and the user it's not _myuser_... you either can specify that on the
command line using:
```
ssh anotheruser@remotehost
```
or you can declare it in a host entry on the _SSHConfig_ file:
```
Host remotehost
  HostName remotehost.example.com
  User anotherUser
```

The default it's just that, something to fall back if you don't specify
it.
## ControMaster / ControlPath
During a normal work day, you might connect to a lot of systems, and use
public/private key authentication; this is fine, but most of the times,
even though we have tools like *screen* that allow us to have multiple
terminals on the remote server, it's way faster to just open a new
connection to the server. Each time you connect to a remote server, you
end up creating a new connection and socket, and depending on the size
and security of your keys, or if you are using password authentication
(shame on you), it get's a little anoying to wait those seconds for the
connection, or having to type, over and over again a password. Also, as
we explain further down this document, when using Jump Hosts, if you do
not have *ControlMaster* on, you will have to authenticate each time on
the _server-in-the-middle_ for your remote server; I can not emphasize
how much boring this is.
_SSHConfig_ manual pages tell us this:
```
ControlMaster

             Enables the sharing of multiple sessions over a single
network connection.  When set to ``yes'', ssh(1) will listen for
connections on a control socket specified using the ControlPath
argument.  Additional sessions can connect to
             this socket using the same ControlPath with ControlMaster
set to ``no'' (the default).  These sessions will try to reuse the
master instance's network connection rather than initiating new ones,
but will fall back to connecting
             normally if the control socket does not exist, or is not
listening.
```
I recommend creating a _tmp_ directory inside _~/.ssh/_ and store
everything related to _ControlMaster/ControlPath_ inside, like is noted
on the _default_ config file.
```
  ControlMaster auto
  ControlPath ~/.ssh/tmp/%r@%h:%p
```

From now on, each time you reconnect to a host, that you are already
connected, *ControlMaster* will share the same connection, no matter how
many times you use this (whick can be a lot, when you are using
[JumpHosts](#jump-hosts)).
## PKI Authentication
authentication without passwords, using strong public/private keys
## Tunnels
exposing remote and local ports
## SOCKS Proxy
VPN Browser navigation 
## Jump Hosts
Imagine this scenario: I use a IPSec VPN connection to my work, and the
security guys are not very fond for SSH, but I ended up convincing them
to allow me, at least, to connect to one host, which has access to all
the other hosts in the network, or, at least some of them; I have
disallowed passwords, and I'm using a combination of PKI with One Time
Password (using [Google
Auth](https://github.com/google/google-authenticator)).

I could access my servers the usual (newbie) way, in where I access
the **JUMPHOST** and then I access the required server, example:
```
ssh jumphost
    [myuser@JUMPHOST01 ~]$
ssh insidehost
    [myuser@INSIDEHOST01~]$
```
This is fun and all, and it works, but it gets boring quick! At least we
are already using ControlMaster, so we only authenticate the first time
(remember, here I am using PKI+OTP).

But there is a better, cleaner solution! First, lets create a entry, for
the **INSIDEHOST01** and for a second host, let's call it
**INSIDEHOST02**:
```
Host insidehost1 j.insidehost1
  HostName insidehost01.example.com

Host insidehost2 j.insidehost2
  HostName insidehost02.example.com
```
I could setup four different entries for these two hosts, one for the
normal connection, another for using the jump host, but I use a trick,
where I can specifie multiple alias in the **Host** parameter; this way,
I can create a default entry for the **j.hosts** entry, like we did on the
__Host *__.
Just add this inside _default_ config file, or create a _JUMP.config_
inside the _configs_ directory:
```
  Host jumphost
    HostName jumphost01.example.com
   
  Host j.*
  ProxyCommand ssh jumphost nc -w 120 %h %p
```
You can create several different jump hosts, depending on context, and
you can also jump more then one server, for example, if you need to get
to _insidehost1_ to reach _insidehost2_, just declare:
```
Host insidehost2
  HostName insidehost02.example.com
  ProxyCommand ssh j.insidehost1 nc -w 120 %h %p
```
This way, you first SSH to JUMPHOST01, then to INSIDEHOST01, and finally
to INSIDEHOST02, everything automatically. If you are using public key
authentication, and you already are connected to the first jumphost, you
will login directly on the final destination.

Note that any option you give to a jumphost, will be set on the final
destination, for example, a port forwarding; it will no be forwarded
inside the first jumphost.
## Remote copy
using pbcopy on remote host
## Forward Agent
holding your keys to reuse
## Keep Alive
..so your conection is not dropped!
## Default user defined by context
