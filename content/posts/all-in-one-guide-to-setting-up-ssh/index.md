---
title: "All in One Guide to Setting Up Ssh"
description: "How to set up SSH clients and servers for MAXIMUM PRODUCTIVITY (and security)"
date: 2023-05-23T15:59:26+10:00
draft: false
categories:
  - Computing
tags:
  - tips
  - linux
  - future-reference
  - security
  - ssl
  - open-source
  - fedora
images:
  - "posts/all-in-one-guide-to-setting-up-ssh/LockAndKey.jpg"
---
This guide is mostly a reference for myself as it's a workflow that I find myself following whenever I set up a new Linux machine. I'm sharing it here as I'm sure many will find this useful, and it will save me from looking up each distinct step in the future.

I also took the opportunity to add some bonus network configuration tips to get the most out of the setup.

<!--more-->

## SSH 101
SSH at its core is a means to connect to another machine remotely. One of the most fundamental uses of this is to log into another machine's terminal, but it also provides functionality to open tunnelling ports and run GUI apps over a secure connection.

SSH connections require two components; a server running on the machine you want to connect to, and a client running on the machine you want to connect from.

Secure, passwordless connections are facilitated by way of private/public key pairs. Think of a public key as a lock that you can replicate and give to other machines and services, and the private key is the only way to unlock those locks. So it should go without saying that you never give out your private key.

Some of my favourite things to do with SSH are:
- The obvious one, administering a machine remotely through the terminal
    - With an SSH client such as [ConnectBot](https://f-droid.org/packages/org.connectbot/), you can even manage a machine from your phone.
- SFTP - using SSH you can expose the filesystem of a machine using the SFTP protocol.
- VS Code Remote Development - With the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit) extension for VS Code, you can run VS Code locally while developing on a remote machine (I'm doing this right now). But why? Well, in my case I'm editing the files directly on my web server (bit naughty), but in other cases you may want to offload the hard work of building, testing etc to a remote machine, freeing up resources on your working machine, or to centralise your working environment so that you can use any machine to do the work without losing your place.

## Initial server setup
### Install the SSH Server
Ensure the server is installed on the machine you intend to connect to. The package is typically called `openssh-server`, don't be surprised if it's already installed. You can check if it's running with:

``` sh
$ systemctl status sshd
```

### Enable the SSH Server
If it isn't running automatically, just enable it.

``` sh
$ sudo systemctl enable sshd
```

## Client setup
### Install the SSH Client
Again, the client will probably already be installed on the machine you want to connect from. If not, the package is typically called `openssh-client`.

### Create Private/Public Key Pair
For secure, passwordless connections, you'll want to create a public/private key pair.

``` sh
$ ssh-keygen
```

The defaults are usually fine and random enough. This will create two files in your home's `.ssh` directory, `id_rsa` (your private key, keep it private) and `id_rsa.pub` (your public key, for sharing).

### Set Up Connection Profiles
Create a new file in `~/.ssh` called `config`. Below is an example file with two connection profiles.

``` config
Host compy
	HostName compy-386
	User strongbad
    Port 1234
    X11Forwarding yes

Host tandy-400
    HostName 192.168.0.11
    User strongbad
```

With this in place, I could connect to the computer `compy-386` on a custom port with the specified user and X11 forwarding support by simply entering

``` sh
$ ssh compy
```

### Copy Public Key to Server
Assuming the config has been set up, you can copy the generated public key to the server with:

``` sh
$ ssh-copy-id compy
```

## Improving Server Security
### Change the SSH Port
You probably noticed in that first config profile that the port was specified. By default, SSH listens on port 22, but you can make it a bit more secure by making it harder to guess.

``` sh
$ sudo nano /etc/ssh/sshd_config
```

Not far from the top you will see a line with

``` config
# Port 22
```

That hash (`#`) symbol is a comment, remove it and change the `22` to something else, taking care not to use a port number reserved for a service you will be using.

On Fedora there are some [extra steps](https://www.ctrl.blog/entry/how-to-alternate-ssh-port-fedora.html), run the following:

``` sh
$ sudo systemctl edit sshd.socket
```

Add the following between the lines the file tells you to edit, substituting `1234` with the port you're using for SSH (the empty value entry is necessary to clear all previous settings):

``` config
[Socket]
ListenStream=
ListenStream=1234
```

Save and close the file. Next, add the port to SELinux's SSH policy, substituting `1234` to reflect your chosen port:

``` sh
$ sudo semanage port -a -t ssh_port_t -p tcp 1234
```

Then open the firewall port, substituting `1234` to reflect your chosen port:

``` sh
$ sudo firewall-cmd --permanent --service="ssh" --add-port "1234/tcp"
$ sudo firewall-cmd --reload
```

Finally, no matter what distro you're on, reload the SSHD service's config for the changes to take effect.

``` sh
$ sudo systemctl reload sshd
```

#### Tidying up
Don't forget to update your local `config` file if the port wasn't specified before. Once you've confirmed a connection can be made with the port changes, remove the old port from the firewall config:

``` sh
$ sudo firewall-cmd --permanent --service="ssh" --remove-port "22/tcp"
$ sudo firewall-cmd --reload
```

And remove the old port from SELinux's SSH policy:

``` sh
$ sudo semanage port -d -p tcp 22
```

### Remove password access
Once you've confirmed that you can login without a password using a public key, you can turn off SSH password authentication altogether. This method requires a valid private key on the client machine to connect, you will no longer be prompted to enter a password. I would only recommend this if you have a way to access the machine either physically or through some kind of always available terminal (e.g. QEMU terminal in a web page) so that you can still access it if for some reason you don't have any valid private keys to connect with.

Simply edit the SSHD config:

``` sh
$ sudo nano /etc/ssh/sshd_config
```

Find the following line:

``` config
#PasswordAuthentication yes
```

Remove the comment character `#` and change the value from `yes` to `no`. Save and exit, then reload the configuration:

``` sh
$ sudo systemctl reload sshd
```

## Router Setup
If your machine is hosted on your home network, you can make it available publicly (hopefully secure by this point) for when you want to, y'know, run [btop](https://github.com/aristocratos/btop) from the bus. I won't go into too much detail about this as every router is different, but the principles are the same.

First, configure **DHCP** settings on your router, give your machine a static IP. This is necessary so that port forwarding rules will always point to the correct machine.

Next you need to configure **Port Forwarding**. This allows you to open up a public port and have it relay through to a certain network address (that'll be the static IP you created earlier) and port (it doesn't have to be the same port as the one exposed publicly).

With this in place you should be able to access your machine using the specified public port without being on the network, as long as you know the IP address of your home network.

But you can take this to the next level with Dynamic DNS. For this you will need your own domain and the ability to set your DNS settings. I use [DNS-O-Matic](https://www.dnsomatic.com/) for this, it updates the **A** record for a subdomain to point to my home network.

There are two parts to it; first you need to set up a DNS-O-Matic account and give it access to your domain service, and then set up **Dynamic DNS** in your router to update the service every time it establishes a new internet connection, ensuring your domain or subdomain always points to the correct IP address.