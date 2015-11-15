# [Awesome SSHConfig](https://github.com/mgarces/awesome_sshconfig)
A curated list of awesome hacks for [SSH_Config](http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/ssh_config.5?query=ssh_config); for several years I've improved the use of my ssh_config file, and here I share everything I have obtained so far; I use this on Mac OS X, but should be straighforward to addapt this to Linux.

So far, here is what I have obtained:
* [Awesome SSHConfig](https://github.com/mgarces/awesome_sshconfig)
  * [Separated SSH config files](#separated-ssh-config-files)
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
either from the command line, from /etc/ssh/ssh_config or ~/.ssh/config.

This is fine, if you have a couple of hosts on your SSHConfig file, but
if that grows, and you are like me, and have hundreds of hosts, there is
a need to separate config files, to differentiate for example,
*Production* from *Development*.

To obtain this, first create a folder inside your ~/.ssh, lets call it
_configs_; after this, throw your config files inside, using the
.config extension, and also add a file with the name _default_.

Now here is the fun part! Add the next aliases to your .bashrc (or to
your .zshrc, if you are with the cool kids):

```
alias sshconfig="vim ~/.ssh/configs"

alias sshcompile="echo -n >! ~/.ssh/config && cat
~/.ssh/configs/*.config >> ~/.ssh/config && cat ~/.ssh/configs/default
>> ~/.ssh/config"

alias ssh="sshcompile && ssh"
```
The last line is optional, you can comment it if you want to "compile"
SSHConfig manually; leave it like this, if you want to "compile"
SSConfig each time you connect to a host (recommended). _sshconpile_
alias only cleans the ~/.ssh/config, cats all the .config files inside
the directory, and lastly it cats the _default_ file to the end of the
config; this way, you get your SSHConfig file all organized.
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
