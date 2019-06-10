# SSH

## Basic

### Breaking down the SSH Command Line

The following `ssh` example command uses common parameters often seen when connecting to a remote `SSH` server.

```sh
localhost:~$ ssh -v -p 22 -C neo@remoteserver
```

- `-v` : Print debug information, particularly helpful when debugging an authentication problem. Can be used multiple times to print additional information.
- `-p 22` : Specify which **port to connect to** on the remote SSH server. **22** is not required as this is the default, but if any other port is listening connect to it using the `-p` parameter. The listening port is configured in the `sshd_config` file using the `Port 2222` format.
- `-C` : Compression is enabled on the connection using this parameter. If you are using the terminal over a slow link or viewing lots of text this can speed up the connection as it will compress the data transferred on the fly.
- `neo@` : The string before the **@** symbol denotes the username to authenticate with against the remote server. Leaving out the **user@** will default to using the username of the account you are currently logged in to (`~$ whoami`). User can also be specified with the `-l` parameter.
- `remoteserver` : The hostname that the `ssh` is connecting to, this can be a fully qualified domain name, an IP address or any host in your local machines hosts file. To connect to a host that resolves to both **IPv4** and **IPv6** you can specify add a parameter `-4` or `-6` to the command line so it resolves correctly.

Each of the above a parameters are optional apart from the **remoteserver**.

### Using a Configuration File

While many users are familiar with the `sshd_config` file, there is also a client configuration file for the `ssh` command. This defaults to `~/.ssh/config` but can also be specified as a parameter with the `-F` option.

```config
Host remoteserver
    HostName remoteserver.thematrix.io
    User neo
    Port 2112
    IdentityFile /home/test/.ssh/remoteserver.private_key

Host *
    Port 2222
```

n the above **example ssh configuration** file you can see that there are two Host entries. The first is a specific host entry with **Port 2112** configured, as well as a custom IdentifyFile and username. The second is a wildcard value of **\*** that will match all hosts. Note that the first configuration option found will be used, so the most specific should be at the top of the configuration. More information is found in the man page (`man ssh_config`).

The configuration file can save a lot of typing by including advanced configuration shortcuts any time a connection is made to particular hosts.

### Copy Files over SSH with SCP

The `ssh` client comes with two other very handy tools for moving files around over an **encrypted ssh connection**. The commands are `scp` and `sftp`, see the examples below for basic usage. Note that many parameters for the `ssh` can be applied to these commands also.

```sh
localhost:~$ scp mypic.png neo@remoteserver:/media/data/mypic_2.png
```

In this example the file **mypic.png** was copied to the **remoteserver** to file system location **/media/data** and was renamed to **mypic_2.png**.

Don't forget **the difference in the port parameter**. This is a gotcha that hits everyone using `scp` on the command line. The port parameter is `-P` not `-p` as it is in the `ssh` client! You will forget, but don't worry everyone does.

For those familiar with command line `ftp`, many of the commands are similar when using `sftp`. You can **push**, **put** and **ls** to your hearts desire.

```sh
sftp neo@remoteserver
```

## [Practical Examples](examples.md)
