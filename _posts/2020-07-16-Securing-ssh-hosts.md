---
layout: post
title:  "Securing SSH hosts"
author: "Rory Murdock"
tags: Security DevOps SSH
---

How to secure your SSH server

## Securing SSH hosts

<p align="center"><img src="https://media.giphy.com/media/UWsZzJTs5KBgg4K8FE/giphy.gif"></p>

If you have a server that you want to access via SSH then you should definitely secure it, the internet is full of malicious attackers that will scan for SSH running on port 22 and try to bruteforce in, [Torrey Betts ran an ssh honey pot](https://www.infragistics.com/community/blogs/b/torrey-betts/posts/what-i-learned-after-using-an-ssh-honeypot-for-7-days) that gives an example of just how many attacks you could get in just 7 days. You can setup your own if you want [here](https://blog.ruanbekker.com/blog/2018/10/11/capturing-54-million-passwords-with-a-docker-ssh-honeypot/).

There's four ways the help prevent this

* Setup your firewall to allow SSH from only your IP
* Use a random TCP port
* Disable password based authentication
* Enable 2FA

## IP based allow list

Setting the firewall allow list really depends on the router or where you have your server hosted. [AWS EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/authorizing-access-to-an-instance.html) for example you can edit your default security group to allow your IP/32

Or alternatively you could just not expose SSH at all and use a [Bastion Host](https://cloud.ibm.com/docs/solution-tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) or VPN into your DMZ network.

## Install SSH

If you don't already have SSH installed on your server

```shell
sudo apt-get update
sudo apt-get install openssh-server -y
```

## Change your default port

You can change the port either on your router or via sshd config

```shell
nano /etc/ssh/sshd_config
```

find `Port 22` and change it to a random unused port

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_port.png)

So you really shouldn't open port 22, what else?

## Disable logging in with passwords, use only SSH key pairs

What this does is let you use a key pair to authenticate to the server. There's a few different algorithms you can use for your key pair:

* 🚨DSA - Don't use it, ever. It's been cracked long ago.
* ⚠️ RSA - Only use with a 4096bit key or higher
* ⚠️ ECDSA - There's various opinions on this one but the gist is that it could be compromised by a [bad random number generator](http://www.hyperelliptic.org/tanja/vortraege/20130531.pdf)
* ✅ Ed25519 - This overcomes the potential time based attacks

So, let's create a keypair, open terminal and run

```shell
ssh-keygen -a 128 -t ed25519
```

to create your keypair

`-a 128` will do 128 rounds of encryption on the passphrase to increase the [resistance against brute forcing](https://man.openbsd.org/ssh-keygen.1#:~:text=When%20saving%20a%20private%20key,The%20default%20is%2016%20rounds.)

`-t ed25519` will use the ed25519 as the key type

enter your private key passphrase

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_keygen.png)

This will create your keypair in `~/.ssh`, you'll have `id_ed25519` and `id_ed25519.pub`

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_keys.png)

`id_ed25519.pub` is your public key which will be uploaded to the server. When you connect to the server it will encrypt a random message using the public key and send it, your private key will be used to decrypt this message to verify that you have the private key and are authorised. This is why it's important to keep your private keys (`id_ed25519`) safe and why the permissions are read only 400 to you vs the public key which is 644.

```shell
bash-3.2$ ls -la .ssh/*
-rw-------  1 rorymurdock  staff  419 19 Jul 20:40 .ssh/id_ed25519
-rw-r--r--  1 rorymurdock  staff  105 19 Jul 20:40 .ssh/id_ed25519.pub
```

So we have our keypair, it's recommended to use one keypair per server so that if any one key is compromised you don't need to change them all.  Let's copy this key over to our server

```shell
ssh-copy-id rorymurdock@10.0.0.71
```

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_copy_id.png)

You should verify that the key fingerprint matches your server

```shell
ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub
```

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_fingerprint.png)

Once the keys are copied over you can test it

```shell
ssh rorymurdock@10.0.0.71
```

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_auth.png)

You will need to disable password based authentication by editing  `/etc/ssh/sshd_config` and changing `PasswordAuthentication yes` to `PasswordAuthentication no`

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/ssh_password_auth.png)

## Enable 2FA

I use [Duo](https://duo.com) for my 2FA and they have [an excellent guide](https://duo.com/docs/duounix) on protecting unix with 2FA.

On your server

```shell
sudo apt-get install gcc make libssl-dev libpam-dev -y
wget https://dl.duosecurity.com/duo_unix-latest.tar.gz
tar zxf duo_unix-latest.tar.gz
cd duo_unix-1.11.4
./configure --with-pam --prefix=/usr && make && sudo make install
```

Fill out your Duo details from the add [application section](https://admin-d67b23c1.duosecurity.com/applications/protect/types) of your Duo account

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/duo_site.png)

Note that `failsafe = secure` means if your server cannot connect to DUO it will not let you in.

```shell
sudo nano /etc/duo/pam_duo.conf
```

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/duo_config.png)

Update your SSH settings

```shell
sudo nano /etc/ssh/sshd_config

PubkeyAuthentication yes
PasswordAuthentication no
AuthenticationMethods publickey,keyboard-interactive
UsePAM yes
ChallengeResponseAuthentication yes
UseDNS no
```

Update PAM to use DUO

```shell
sudo nano /etc/pam.d/sshd

auth  [success=1 default=ignore] /lib64/security/pam_duo.so
auth  requisite pam_deny.so
auth  required pam_permit.so
```

Restart SSH

```shell
systemctl restart sshd
```

Now when you ssh into your server you will get a prompt from DUO which you'll have to allow on your phone.

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/duo_term_2FA.png)

![Screenshot]({{ site.url }}/assets/img/Securing-SSH-Hosts/duo_phone.png)

Amazing! We're in and it's secure. You can set Duo to automatically send a push notification without prompt by setting `autopush: yes` in `/etc/duo/pam_duo.conf`

Further reading:

Public key authentication still has it's problems, arguably with 2FA you're better protected but here's a piece on certificate based authentication by CloudFlare
[https://blog.cloudflare.com/public-keys-are-not-enough-for-ssh-security/](https://blog.cloudflare.com/public-keys-are-not-enough-for-ssh-security/)

<p align="center"><img src="https://media.giphy.com/media/9r8DZBBfzACgxkBRh7/giphy.gif"></p>
