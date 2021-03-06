# Practical SSH examples

- [Practical SSH examples](#practical-ssh-examples)
  - [1. Proxy Traffic over SSH using SOCKS](#1-proxy-traffic-over-ssh-using-socks)
  - [2. SSH Tunnel (port forward)](#2-ssh-tunnel-port-forward)
  - [3. SSH Tunnel Forward to Secondary Remote host](#3-ssh-tunnel-forward-to-secondary-remote-host)
  - [4. SSH Reverse Tunnel](#4-ssh-reverse-tunnel)
  - [5. SSH Reverse Proxy](#5-ssh-reverse-proxy)
  - [6. Establish a VPN over SSH](#6-establish-a-vpn-over-ssh)
  - [7. Copy your SSH key (ssh-copy-id)](#7-copy-your-ssh-key-ssh-copy-id)
  - [8. Run Command Remotely (non-interactive)](#8-run-command-remotely-non-interactive)
  - [9. Remote Packet Capture & View in Wireshark](#9-remote-packet-capture--view-in-wireshark)
  - [10. SSH Copy Folder from Local to Remote](#10-ssh-copy-folder-from-local-to-remote)
  - [11. Remote GUI Applications with SSH x11 Forwarding](#11-remote-gui-applications-with-ssh-x11-forwarding)
  - [12. Copy files remotely with rsync and SSH](#12-copy-files-remotely-with-rsync-and-ssh)
  - [13. SSH over Tor Network](#13-ssh-over-tor-network)
  - [14. SSH to EC2 instance](#14-ssh-to-ec2-instance)
  - [15. Edit text files with VIM over ssh/scp](#15-edit-text-files-with-vim-over-sshscp)
  - [16. Mount remote SSH location as local folder with SSHFS](#16-mount-remote-ssh-location-as-local-folder-with-sshfs)
  - [17. SSH Multiplex using ControlPath](#17-ssh-multiplex-using-controlpath)
  - [18. Stream Video over SSH using VLC + SFTP](#18-stream-video-over-ssh-using-vlc--sftp)
  - [19. Two Factor Authentication](#19-two-factor-authentication)
  - [20. Bouncing through jump hosts with ssh and -J](#20-bouncing-through-jump-hosts-with-ssh-and--j)
  - [21. Block SSH Brute Force Attempts with iptables](#21-block-ssh-brute-force-attempts-with-iptables)
  - [22. Modify Port Forwarding within a session with ~C](#22-modify-port-forwarding-within-a-session-with-c)

## 1. Proxy Traffic over SSH using SOCKS

The SSH Proxy feature has been placed at number 1 for good reason. It is more powerful than many users realise giving you access to any system that the remote server can reach, using almost any application. The `ssh` client can tunnel traffic over the connection using a **SOCKS** proxy server with a quick one liner. A key thing to understand is that traffic to the remote systems will have a source of the remote server. For example in a web server log file.

```sh
localhost:~$ ssh -D 8888 user@remoteserver

localhost:~$ netstat -pan | grep 8888
tcp        0      0 127.0.0.1:8888       0.0.0.0:*               LISTEN      23880/ssh
```

Here we start the socks proxy server running on TCP port 8888, the second command checks that the port is now listening. The `127.0.0.1` indicates the service is running on localhost only. We can use a slightly different command to listen on all interfaces including ethernet or wifi, this will allow other applications (browsers or other) on our network to connect to the **ssh socks proxy** service.

```sh
localhost:~$ ssh -D 0.0.0.0:8888 user@remoteserver
```

Now we can configure our browser to connect to the socks proxy. In Firefox select **preferences -> general -> network** settings. Add the IP address and the port for the browser to connect to.

![](img/ssh-socks-proxy.png)

Note the option at the bottom of the form to **force browser DNS requests to also go over the socks proxy**. If you are using the proxy to encrypt your web traffic on the local network you will definitely want to select this option so the DNS requests are also tunnelled over the SSH connection.

**Enable Socks Proxy on Chrome**

Using a command line parameter when launching Chrome will use the **socks proxy** and also **tunnel DNS** requests from the browser over the socks5 proxy. Trust but verify, use tcpdump (tcpdump not port 22) to confirm the DNS requests are no longer visible.

```sh
localhost:~$ google-chrome --proxy-server="socks5://192.168.1.10:8888"
```

**Using other applications with the Proxy**

Keep in mind that there are many other applications that can utilise a **socks proxy**. A web browser is simply the most popular. Some applications will have configuration options for use of the proxy. Others may need some help by using a helper program that talks the socks protocol. An example of this is proxychains. Using this tool we can for example use Microsoft RDP over the socks proxy.

```sh
localhost:~$ proxychains rdesktop $RemoteWindowsServer
```

The configuration options for the socks proxy are set in the proxychains configuration file.

> Hot Tip: Using remote desktop from Linux to Windows? Try the [FreeRDP](https://github.com/FreeRDP/FreeRDP) client. A more modern implementation than `rdesktop` with much smoother interaction.

**Use Case for the SSH Socks Proxy**

You are in a cafe or hotel having to use the somewhat sketchy WIFI. From our Laptop we run the `ssh` proxy locally and establish an `ssh` tunnel into our home network using a our local Rasberry Pi. Using the browser or other applications configured for the SOCKS proxy we can access any network services on our home network or browse to the Internet via our Home Network Connection. Everything between our Laptop and the Home Server (across the WIFI and Internet to home) is encrypted in the SSH tunnel.

## 2. SSH Tunnel (port forward)

In its simplest form an SSH tunnel opens a port on your local system that connects through to another port at the other end of the tunnel.

```sh
localhost:~$ ssh  -L 9999:127.0.0.1:80 user@remoteserver
```

Lets break down the **-L** parameter. Think of **-L** as the Local listening side. So in our example above the port **9999** is listening on localhost and port forwards through to port **80** on **remoteserver**, note that the **127.0.0.1** refers to localhost on the remote server!

Lets take it up a notch. In this following example the port that is listening can be connected to from other hosts on the local network.

```sh
localhost:~$ ssh  -L 0.0.0.0:9999:127.0.0.1:80 user@remoteserver
```

In these examples the port we are connecting is a listening web server. It could also be a proxy server or any other TCP service.

## 3. SSH Tunnel Forward to Secondary Remote host

We can use the same options seen above to have the tunnel connect to **another service running on a secondary system** from the remote server.

```sh
localhost:~$ ssh  -L 0.0.0.0:9999:10.10.10.10:80 user@remoteserver
```

In this example we are forwarding the tunnel from **remoteserver** to the web server running on 10.10.10.10. The traffic from **remoteserver** -> **10.10.10.10** is **no longer within the ssh tunnel**. The web server on **10.10.10.10** will see remoteserver as the source of the web requests.

## 4. SSH Reverse Tunnel

In this scenario we want to setup a listening port on the remote server that will connect back to a local port on our localhost (or other system).

```sh
localhost:~$ ssh -v -R 0.0.0.0:1999:127.0.0.1:902 192.168.1.100 user@remoteserver
```

With this ssh session established a connection to the **remoteserver** port **1999** will be forwarded to port **902** on our local client.

## 5. SSH Reverse Proxy

In this case we are establishing a SOCKS proxy with our ssh connection, however the proxy is listening at the remote server end. With connections to that remote socks proxy now emerging from the tunnel as traffic originating from our localhost. Requires **OpenSSH version 7.6+**.

```sh
localhost:~$ ssh -v -R 0.0.0.0:1999 192.168.1.100 user@remoteserver
```

**Troubleshooting Remote SSH Tunnels**

If you are having trouble getting the remote SSH options to work, check with `netstat` which interface the listening port is attached too. Even though we have specified **0.0.0.0** in the above examples, if **GatewayPorts** is set to no in the **sshd_config** then the listener will only bind to **localhost** (127.0.0.1).

> **Security Warning:** Note that when you are opening tunnels and socks proxies you may be exposing internal network resources to **untrusted networks** (like the Internet!). This can be a serious security risk so ensure you understand what is listening and what it has access too.

## 6. Establish a VPN over SSH

A common term amongst offensive security folks (pentesters / red teams / etc), is to pivot into a network. Once you have a connection established on one system that system becomes a gateway point for further access to the network. This is known as pivoting and enables lateral movement through the network.

We can use the SSH proxy for this and **proxychains**, however there are some limitations. For example we cannot use raw sockets, so **Nmap** `SYN` scans cannot be used to port scan the Internal network.

Using this more advanced VPN option we move the connectivity down to **layer 3**. We can then route traffic through the tunnel using standard network routing.

This technique uses `ssh`, `iptables`, `tun interfaces` and routing.

First we need these options set in the `sshd_config`. Since we are making interface changes on the remote system and the client system, we will **need root privileges on both sides**.

```config
PermitRootLogin yes
PermitTunnel yes
```

Then we will establish our `ssh` connection using the parameter that requests `tun` devices be initialised.

```sh
localhost:~$ ssh -v -w any root@remoteserver
```

Now you should have a `tun` device when you show interfaces (`# ip a`). Next step is to add IP addresses to the tunnel interfaces.

SSH Client Side:

```sh
localhost:~$ ip addr add 10.10.10.2/32 peer 10.10.10.10 dev tun0
localhost:~$ ip tun0 up
```

SSH Server Side:

```sh
remoteserver:~$ ip addr add 10.10.10.10/32 peer 10.10.10.2 dev tun0
remoteserver:~$ ip tun0 up
```

Now we should have a direct route to the other host (`route -n` and `ping 10.10.10.10`).

It is now possible to route any subnet through the other side host.

```sh
localhost:~$ route add -net 10.10.10.0 netmask 255.255.255.0 dev tun0
```

On the remote side we need to enable `ip_forward` and `iptables`.

```sh
remoteserver:~$ echo 1 > /proc/sys/net/ipv4/ip_forward
remoteserver:~$ iptables -t nat -A POSTROUTING -s 10.10.10.2 -o enp7s0 -j MASQUERADE
```

Boom! **Layer three VPN through an SSH tunnel**. Now that's winning.

Any trouble, try `tcpdump` and `ping` to see where its broken. Since we are playing at layer 3 our `icmp` packets should be jumping through that tunnel.

## 7. Copy your SSH key (ssh-copy-id)

There are multiple ways to achieve this however this command is a shortcut that saves time. What does it actually do? This command replicates what you can also do manually. Copying the `~/.ssh/id_rsa.pub` (or the default) key from your system and adds it to an `~/.ssh/authorized_keys` file on the remote server.

```sh
localhost:~$ ssh-copy-id user@remoteserver
```

## 8. Run Command Remotely (non-interactive)

The `ssh` command can be chained to other commands for the usual piping fun. Add the command you want to run on the remote host as a final parameter in quotes.

```sh
localhost:~$ ssh remoteserver "cat /var/log/nginx/access.log" | grep badstuff.php
```

In this example the `grep` is being performed on the local system after the log file has been pushed across the `ssh` session. If the file is large it would be more efficient to run the `grep` on the remote side by enclosing the pipe and `grep` in the double quotes.

Another example performs the same function as the `ssh-copy-id` short cut in Tip 7.

```sh
localhost:~$ cat ~/.ssh/id_rsa.pub | ssh remoteserver 'cat >> .ssh/authorized_keys'
```

## 9. Remote Packet Capture & View in Wireshark

Use it for a remote packet capture with the results feeding directly into your local Wireshark GUI.

```sh
localhost:~$ ssh root@remoteserver 'tcpdump -c 1000 -nn -w - not port 22' | wireshark -k -i -
```

## 10. SSH Copy Folder from Local to Remote

A neat trick that compresses a folder using `bzip2` (that's the -j in the `tar` command), then extracts the `bzip2` stream on the other side creating a duplicate of the folder on the remote server.

```sh
localhost:~$ tar -cvj /datafolder | ssh remoteserver "tar -xj -C /datafolder"
```

**Copy remote folder to local tar archive**

To go the other way, copying a remote folder to a local archive. Handy for quick backups of remote resources.

```sh
localhost:~$ ssh user@remoteserver "tar -jcf - /path/to/backup" > dir.tar.bz2
```

## 11. Remote GUI Applications with SSH x11 Forwarding

If the client and remote server both have X installed. It is possible to run a GUI command remotely, with the Window appearing on your local desktop. This feature has been around since the beginning of time, but can still be very useful. Run a remote web browser or even the VMWawre Workstation console as I do in this example.

```sh
localhost:~$ ssh -X remoteserver vmware
```

Requires `X11Forwarding yes` in the `sshd_config`.

## 12. Copy files remotely with rsync and SSH

Using the `rsync` has many advantages over `scp`, if periodically need to backup a directory, large numbers of files or very large files it should be used. It has the ability to recover from failed transfers and only copy differences between two locations saving bandwidth and time.

The example here uses `gzip` compression (-z) and archive mode (-a) that includes recursive copy.

```sh
localhost:~$ rsync -az /home/testuser/data remoteserver:backup/
```

## 13. SSH over Tor Network

The anonymised **Tor Network** can tunnel SSH traffic by using the `torsocks` command. The following command will proxy the `ssh` connection through the Tor network.

```sh
localhost:~$ torsocks ssh myuntracableuser@remoteserver
```

[Torsocks](https://github.com/dgoulet/torsocks) will use the **localhost** port **9050** to proxy traffic. As always when using `tor` serious consideration must be taken to understand what traffic is being tunnelled and other **operational security** (opsec) concerns. **Where are your DNS requests going**?

## 14. SSH to EC2 instance

When using SSH to connect to your EC2 instance within Amazon you will need to use a private key. Download the key (extension .pem) from your Amazon EC2 control panel and change the permissions (`chmod 400 my-ec2-ssh-key.pem`. Keep this key somewhere safe or put it in your `~/.ssh/` folder.

```sh
localhost:~$ ssh -i ~/.ssh/my-ec2-key.pem ubuntu@my-ec2-public
```

The **-i** parameter tells the `ssh` client to use this key. This would be an ideal example of where to use the `~/.ssh/config` to configure the use of the key automatically when connecting to the ec2 host.

```config
Host my-ec2-public
   Hostname ec2.compute-1.amazonaws.com
   User ubuntu
   IdentityFile ~/.ssh/my-ec2-key.pem
```

## 15. Edit text files with VIM over ssh/scp

For all those vim users out there, this one can save some time. Using `vim` we can edit files over `scp` with one command. Using this method creates a file in `/tmp` on the local system and then copies it back once we write the file in `vim`.

```sh
localhost:~$ vim scp://user@remoteserver//etc/hosts
```

Note the format is slightly different to regular `scp`. After the host we have a double `//`. This references the absolute path. A single slash will have a path that is relative to the users home directory.

```sh
**warning** (netrw) cannot determine method (format:
protocol://[user@]hostname[:port]/[path])
```

If you see this error, double check the format of your command. It usually means there is a syntax error.

## 16. Mount remote SSH location as local folder with SSHFS

Using `sshfs` - an `ssh` filesystem client, we can mount a local directory to a remote location with all file interaction taking place over the encrypted `ssh` session.

```sh
localhost:~$ apt install sshfs
```

On Ubuntu and Debian based system we install the `sshfs` package and then mount the remote location.

```sh
localhost:~$ sshfs user@remoteserver:/media/data ~/data/
```

## 17. SSH Multiplex using ControlPath

By default when you have an existing connection to a remote server with `ssh`, a second connection using `ssh` or `scp` will establish a new session with the overhead of authentication. Using the `ControlPath` options we can have the existing session be used for all subsequent connections. This will speed things up significantly. It is noticeable even on a local network but even more so when connecting to remote resources.

```config
Host remoteserver
    HostName remoteserver.example.org
    ControlMaster auto
    ControlPath ~/.ssh/control/%r@%h:%p
    ControlPersist 10m
```

**ControlPath** denotes a socket that is checked by new connections to see if there is an existing `ssh` session that can be used. The **ControlPersist** option above means even after you exit the terminal, the existing session will remain open for 10 minutes, so if you were to reconnect within that time you would use that existing socket. See the `ssh_config man` page for more information.

## 18. Stream Video over SSH using VLC + SFTP

Long time users of `ssh` and `vlc` (Video Lan Client) are not always of aware of this handy option for when you need to watch video over the network. Using the `vlc` option to File | Open Network Stream one can enter the location as a an `sftp://` location. A prompt will appear for authentication details if password is required.

```sh
sftp://remoteserver//media/uploads/myvideo.mkv
```

## 19. Two Factor Authentication

Most readers will understand the value in using **Two Factor Authentication**, the same benefits that apply to your banking or Google Account can be applied to your SSH service.

Of course `ssh` comes with a form of Two Factor capability included, that being a passphrase and an SSH key. An advantage of using a hardware based token or the **Google Authenticator App** is the fact that they are generally coming from a second physical device.

See our 8 minute guide to getting started with **Google Authenticator and SSH**.

## 20. Bouncing through jump hosts with ssh and -J

When network segmentation means you are jumping through multiple `ssh` hosts to get to a final destination network or host, this jump host shortcut might be just what you need. Requires **OpenSSH version 7.3+**.

```sh
localhost:~$ ssh -J host1,host2,host3 user@host4.internal
```

A key thing to understand here is that this is not the same as `ssh host1` then `user@host1:~$ ssh host2`, the `-J` jump parameter uses forwarding trickery so that the **localhost** is establishing the session with the next host in the chain. So our **localhost** is authenticating with **host4** in the above example; meaning our **localhost keys** are used and the session from **localhost** to **host4** is **encrypted end to end**.

To use this ability in the `ssh_config` use the **ProxyJump** configuration option. If you regularly have to jump through multiple hosts; use the config file and your alias to `host4` will save you a lot of time.

## 21. Block SSH Brute Force Attempts with iptables

Anyone who has managed an **SSH** service on the Internet, and viewed the logs will be aware of the amount of SSH **brute force attempts** that take place every hour of every day. An immediate way to reduce the noise in your logs is to move SSH to a port other than **22**. Make the change in the `sshd_config` file using the **Port ##** configuration option.

Using `iptables` we can also block attempts to connect to the port from sources that reach a certain threshold. A way to do this is to use `OSSEC`, as this not only blocks SSH but will also perform a bunch of other host based intrusion detection functions (HIDS).

## 22. Modify Port Forwarding within a session with ~C

And our final `ssh` example is for modifying port forwarding on the fly within an existing ssh session. Picture this example scenario. You are deep in a network; perhaps you have jumped through half a dozen jump hosts and need a local port on your workstation forwarded to Microsoft SMB on the old Windows 2003 system you spotted (ms08-67 anyone?).

After hitting `enter` try typing `~C` in your terminal. This a control escape sequence within the session that allows to make changes to the existing connection.

```sh
localhost:~$ ~C
ssh> -h
Commands:
      -L[bind_address:]port:host:hostport    Request local forward
      -R[bind_address:]port:host:hostport    Request remote forward
      -D[bind_address:]port                  Request dynamic forward
      -KL[bind_address:]port                 Cancel local forward
      -KR[bind_address:]port                 Cancel remote forward
      -KD[bind_address:]port                 Cancel dynamic forward
ssh> -L 1445:remote-win2k3:445
Forwarding port.
```

You can see here we have forwarded our local port **1445** to the **Windows 2003** host we found on the internal network. Now launch `msfconsole` and we are good to go (assuming you were planning on **exploiting that host**).
