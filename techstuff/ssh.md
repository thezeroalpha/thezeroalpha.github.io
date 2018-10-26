---
title: SSH stuff
---

# Connect to `<targetserver>` via `<stepstone>`
Set it up in ~/.ssh/config like this:

```sshconfig
Host stepstone
    User <user>
    Hostname <hostname>
Host targetserver
    User <user>
    Hostname <hostname>
    ProxyCommand ssh -W %h:%p stepstone
```

`<user>` is your username, `<hostname>` is the hostname or IP address of the server.
ProxyCommand is the command that SSH will use as a transport channel -- write to stdin of the command, read from stdout of the command. 

This is essentially running the command

```bash
ssh -o ProxyCommand="ssh -l <username> -W %h:%p <stepstone>" -l <username> <targetserver>
```

`%h:%p` gets translated to `targetserver:22` (port 22 by default, you can also set up a `Port` option).

If the `-W` option is not available in SSH, you can use the old style, with netcat:

```bash
ssh -o ProxyCommand="ssh -l <username> <stepstone> 'nc %h %p'" -l <username> <targetserver>
```

Where `nc` basically passes packets to the target server. You connect to the stepstone, and then execute netcat on it to tunnel to the next server. Here's a diagram:

![Diagram of SSH tunneling](https://backreference.org/wp-content/uploads/2010/01/proxycommand.png)

[Here's even more explanation.](https://backreference.org/2010/02/26/jump-in-with-ssh-and-netcat/)
