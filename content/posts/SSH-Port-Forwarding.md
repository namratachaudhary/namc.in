---
title: "SSH Tunneling - Local, Remote & Dynamic"
tags: [Networking]
date: 2018-06-26 14:02:18 -0700
keywords: ["networking", "ssh"]
---

Most of us are familiar with SSH (Secure Shell) - a protocol which allows us to securely log onto remote systems. SSH Tunneling (also known as SSH Port Forwarding) is a feature of SSH which forwards encrypted connections between a local and remote system. SSH tunneling works by using the already established SSH connection for sending additional network traffic.

We're going to look at the three types of port forwarding - local, remote & dynamic.

## Local port forwarding

Let's see what the man page for SSH tells us about `local`

```
local: -L
Specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side.
```

So generically, the command would look something like

```bash
$ ssh -L sourcePort:forwardToHost:destPort connectToHost
```

This translates to - connect with ssh to `connectToHost`, and forward all connection attempts to the **local** `sourcePort` to port `destPort` on the machine called `forwardToHost`, which can be reached from the `connectToHost` machine. Forwarding can also be done using Unix sockets.

Say YouTube is blocked on your office network, and you reaaaalllyy wanna watch some kitty videos. So, you can get around it by creating a tunnel through a server which isn't on your network and access YouTube. The above command can be translated to :

```bash
$ ssh -L 9000:youtube.com:80 user@example.com
```

The `-L` flag here signfies that we're doing local port forwarding. We connect with the `user@example.com` machine. We then forward any connection to port 9000 on the local machine to port 80 (which is the default port for HTTP) on `youtube.com`. Now, if you open your browser and go to `http://localhost:9000` , a request is made to HTTP server listening on `youtube.com`. However, you, on your local machine, have no webserver running.

You still won't be able to the see homepage - so don't worry about that.

The requests in the browser for `localhost:9000` are built with a Host Destination header of localhost value. This request reaches `youtube.com` machine. But this request is ignored with an error message - `Invalid virtual host ..` because localhost cannot be a domain name on the server which is running `youtube`.

To fix this, we change the Host HTTP header enabling remote web server to identify the corresponding destination.

```bash
$ curl -H "Host: youtube.com" -L localhost:9000
```

`-L` parameter in curl is for following redirects. You should now be able to see the contents of homepage from `youtube.com` on `localhost:9000` as long as you are connected to `user@example.com` machine.

Also! The good things about SSH tunnels is that they are encrypted, so nobody can see what sites you are visiting - only an SSH connection to your server. (Take that, Office admins!! 😎)

By default, anyone (even on different machines) can connect to the specified port on the local machine. This can be restricted to programs on the same host by supplying a bind address:

```bash
$ ssh -L 127.0.0.1:9000:youtube.com:80 user@example.com
```

SSH binds to port 9000 on the local machine. Any traffic that comes to this port is sent to the SSH server that listens on `user@example.com` - the remote machine. Once received by remote-machine, the traffic is then sent to port 80 of 127.0.0.1, which is `user@example.com` itself.

### Connecting to a database behind a firewall

`forwardToHost` host may also refer to the remote machine through which the ssh connection is made (i.e. `connectToHost`) - in which case, the value for `forwardToHost` becomes 127.0.0.1 or localhost, as its local in the context of already established connection with `connectToHost`.

```bash
$ ssh -L 9000:127.0.0.1:80 user@example.com
```

An example for this is when you need to connect to a database console, which only allows local connection for security reasons, running PostgreSQL on your server, which by default listens on the port 5432.

```bash
$ ssh -L 9000:localhost:5432 user@example.com
```

This command forwards the local port 9000 to the port 5432 on the remote machine. You can connect to that remote PostgreSQL server through the local machine using psql on localhost:9000, simply like :

```bash
$ psql -h localhost -p 9000
```

Let's take a moment here and understand what is actually going on.

In the YouTube example, `9000:youtube.com:80` says - forward my local port 9000 to `youtube.com` at port 80. So SSH on your server actually makes a tunnel (connection) between those two ports - one of which lies on your local machine, and another on target machine.

In the example of database connection, `9000:localhost:5432` means localhost from server's perspective, not localhost on your machine. In other words - forward my local port 9000 to port 5432 on the server - because when you're on the server, localhost means server itself.

Port numbers less than 1024 or greater than 49151 are reserved for the system, and can only be forwarded by root. If you're using port forwarding of any kind, you need to specifiy the destination server, i.e. `connectToHost`.

Port forwarding is enabled by default. If not, check `AllowTcpForwarding` in `/etc/ssh/sshd_config`.

## Remote port forwarding

Going back to the man-page again to see the definition of `remote`

```
remote: -R
Specifies that the given port on the remote (server) host is to be forwarded to the given host and port on the local side.
```

The command would look much like local tunneling's but with an `-R` flag.

```bash
$ ssh -R sourcePort:forwardToHost:destPort connectToHost
```

This translates to - connect with ssh to `connectToHost`, and forward all connection attempts to the **remote** `sourcePort` to port `destPort` on the machine called `forwardToHost`, which can be reached from the `connectToHost` machine. Forwarding can also be done using Unix sockets.

Okay! Let's see an example.

Say you're developing an application on your local machine and you'd like to show the prototype to your boss.

In most cases, the ISP doesn't provide you with a public IP address, so you cannot connect your machine directly via the internet. While this problem can be solved by configuring NAT (Network Address Translation) on your router - this might not always work, there's a technical overhead of changing the configuration of your router, and you would need the admin access on your network.

In such a scenario, you can setup a server on internet which is publicly accessible and has SSH access. Then we tell SSH to make a tunnel that opens up a new port on server, and you connect to it via local port on your machine.

```bash
$ ssh -R 9000:localhost:3000 user@example.com
```

The syntax here is very similar to local port forwarding, with a single change of **-L** for **-R**.

SSH will connect to the remote machine - in this case `user@example.com`. The flag **-R** makes ssh listen on the port 9000 of that machine. Once there's a process on the machine connecting to 9000, the ssh server listening on the same machine will transfer that connection to local machine - the machine that initiated the ssh communication - and forward it to localhost on the port 3000.

Remote port forwarding allows to map a port of the local machine onto the remote server via SSH.

Another thing which you need to do is to set `GatewayPorts`. In your `/etc/ssh/sshd_config` , set

```bash
GatewayPorts yes
```

And restart SSH

```bash
$ sudo service ssh restart
```

This allows the SSH server to bind port 3000 to the wildcard address - such that the port becomes available to the public address of the `connectToHost` remote machine.

You can also set `GatewayPorts` to `clientspecified` - in which case, the remote port will not bind on the wildcard address. You might need to explicitly specify an empty bind address for binding the wildcard address - which can be done by prefixing remote port with `:` sign.

```bash
$ ssh -R :9000:localhost:3000 user@example.com
```

You can also specify an IP address from which connections to the port are allowed, such that only connections from the given IP address to the given port are allowed.

```bash
$ ssh -R 1.2.3.4:9000:localhost:3000 user@example.com
```

Now your boss will be able to access your application on port 3000 by pointing their browser to `connectToHost` IP address on the port 9000.

### Double forwarding

If the remote server has `GatewayPorts` set to `no`, with no possibility of changing it - you can execute the remote forwarding above followed by a local forwarding using `-g` option, but from the remote server.

```bash
$ ssh -g -L 9000:localhost:3000 user@example.com
```

**-g** allows remote hosts to connect to local forwarded ports and this will make loopback port 3000 on the server accessible on all interfaces on port 9000.

## Dynamic port forwarding - SOCKS

Dynamic port forwarding allows communication across a range of ports. This port forwarding is created using a **-D** parameter. This makes SSH acts as a [SOCKS proxy server](https://en.wikipedia.org/wiki/SOCKS).

There's two kinds of SOCKS protocols - SOCKS4 and SOCKS5. These are basically internet protocols which route packets between a server and a client using proxy server. SOCKS5 uses both TCP and UDP, whereas SOCKS4 uses only TCP.

A SOCKS proxy is a simple SSH tunnel in which specific applications forward their traffic through the tunnel to the remote server, then the proxy forwards the traffic out to the general internet. Unlike a VPN, SOCKS proxy has to be configured for each application separately on the client machine.

Dynamic port forwarding can handle connections from multiple ports. It analyses the traffic to determine the destination for given connection. However, you might need to configure programs to use a SOCKS proxy server.

This can be done by :

```bash
$ ssh -D 9000 -f -C -q -N connectToHost
```

**-D** tells SSH to create a SOCKS tunnel on the the port 9000.

**-f** forks the process to the background.

**-C** compresses the data before sending it.

**-q** enables quiet mode.

**-N** tells SSH that no command will be sent once the tunnel is up.

A downside to using proxies is decreased performance and mislabelled errors as they rewrite data packet headers. With SOCKS5 proxy, the server does not rewrite data packet headers - hence being more performant and less prone to data routing errors. Because SOCK5 proxies they are low-level, they can work with any kind of data traffic - program, protocol etc.

## Closing Tip!

The `-nNT` flags will cause SSH to not allocate a tty and only do the port forwarding. This will prevent the creation of shell everytime you create a tunnel.

```bash
$ ssh -nNT -L 9000:youtube.com:80 user@example.com
```

### More reading

* [SSH Tunneling](https://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html?utm_source=cronweekly.com)
* [UnixSE - how does ssh tunneling work](https://unix.stackexchange.com/questions/46235/how-does-reverse-ssh-tunneling-work)
* [UnixSE - ssh port forward to access my home machine from anywhere](https://unix.stackexchange.com/a/19624)
* [The Black Magic Of SSH / SSH Can Do That?](https://vimeo.com/54505525) - This is a video.

