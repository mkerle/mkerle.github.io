---
layout: single
title:  "SSH Public Key Authentication"
date:   2022-08-17 19:00:00 +1000
toc: true
categories: linux ssh publickey
---

Public key authentication offers many benifits over password based authentication.  This guide describes how to add public key authentication between servers.

## Generate Public Key

On the client machine generate a key pair.  An example of this command is shown below.  It is safe to overwrite the id file (in the first case).  A password can be added to the key.

{% highlight bash %}
$ ssh-keygen -t Ed25519
{% endhighlight %}

The `pub` file that is generated needs to be copied to the server.  This can be done using `ssh-copy-id`.

{% highlight bash %}
$ ssh-copy-id user@server01
{% endhighlight %}

On the server the copy command will add the key to `~/.ssh/authorized_keys`.

## Configuration

To enesure that public key authentication is enabled on the server check the config in `/etc/ssh/sshd_config` for the `PubkeyAuthentication yes` setting.

# Security Logs

# Password Based Logins

Linux servers accecpting SSH connections will display "Accepted password for <user> from <ip> port <src port> ssh.