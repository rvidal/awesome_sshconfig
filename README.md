# [Awesome *SSHConfig*](https://github.com/mgarces/awesome_sshconfig)
A curated list of awesome hacks for [SSH_Config](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/ssh_config.5?query=ssh_config); for several years I've improved the use of my ssh_config file, and here I share everything I have obtained so far; I use this on Mac OS X, but should be straighforward to addapt this to Linux.

So far, here is what I have obtained:
* [Awesome *SSHConfig*](https://github.com/mgarces/awesome_sshconfig)
  * [Separated SSH config files](#separated-ssh-config-files)
  * [A sane *default* config file](#a-sane-default-config-file)
  * [PKI Authentication](#pki-authentication)
  * [ControMaster / ControlPath](#contromaster--controlpath)
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


## PKI Authentication
authentication without passwords, using strong public/private keys
## ControMaster / ControlPath
reusing connections
## Tunnels
exposing remote and local ports
## SOCKS Proxy
VPN Browser navigation 
## Jump Hosts
getting straight to some host, behind firewall
## Remote copy
using pbcopy on remote host
## Forward Agent
holding your keys to reuse
## Keep Alive
..so your conection is not dropped!
## Default user defined by context
