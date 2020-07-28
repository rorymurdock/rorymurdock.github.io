---
layout: post
title:  "Remote Development"
author: "Rory Murdock"
tags: DevOps Development
---

How to create a better development environment with VS Code

# Remote Development

If you're like me then you install a bunch of python packages and you might have python 2.7 and 3.8 both installed and then maybe Ruby or Powershell or any number of other languages. It's hard to keep everything in sync and to make sure you remember to put all the packages your package needs in the requirements.txt because half of them are already installed on your machine. What's the answer? A virtual machine for each project seems tedious and annoying, uninstalling all your dependencies before you start a new project? It's simple, containers.

VS Code has a few options that you can use for remote development.

[https://code.visualstudio.com/docs/remote/remote-overview]([https://code.visualstudio.com/docs/remote/remote-overview])

There's three different options that we'll look at here:

1. Remote via SSH
2. Develop in Containers
3. Visual Studio Codespaces

## Remote via SSH
First let's look at Remote via SSH

For this demo I've setup a VM on my laptop but you could have a compute instance in the cloud or on your [Home Lab]({% post_url 2020-07-20-Home-lab-setup %}) just make sure you [secure it]({% post_url 2020-07-16-Securing-ssh-hosts %})

Google's Cloud Platform offers a lifetime free [F1-micro](https://cloud.google.com/free/docs/gcp-free-tier) instance that you could use if you wanted an always on remote host, although it would be [fairly resource limited](https://www.opsdash.com/blog/google-cloud-f1-micro.html).

All I've done is install Ubuntu 20.04, VS Code, Python, PIP, and secured SSH with keypair based auth and 2FA.

First install the [Remote Development extension pack](vscode:extension/ms-vscode-remote.vscode-remote-extensionpack) on the Server.
![Screenshot]({{ site.url }}/assets/img/Remote-Development/extension_install.png)

Then the install the [Remote - SSH](vscode:extension/ms-vscode-remote.remote-ssh) extension on the client
![Screenshot]({{ site.url }}/assets/img/Remote-Development/connect_ssh_1.png)

![Screenshot]({{ site.url }}/assets/img/Remote-Development/connect_ssh_2.png)


Once connected make your Git folder, and install `git`, clone a repo. From there it's the same as using VS Code locally. In the bottom left corner the Orange label shows which environment you're working in.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/install_git.png)

You can work as normal, install extensions, terminal even opens on the remote host. Pretty cool.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/working.png)

## Develop in Containers

[What is a container?](https://www.docker.com/resources/what-container) Containers are an abstraction at the app layer that you can use to package code and dependencies together. Basically it lets you virtualise an environment but without the overhead and complexity of a virtual machine.

Docker is a popular container engine so we'll use that for this demo.

First, on our machine, lets install [Docker-CE](https://docs.docker.com/engine/install/)

* [Mac](https://docs.docker.com/docker-for-mac/install/)
* [Windows](https://docs.docker.com/docker-for-windows/install/)
* [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

Next install the [Remote - Containers](vscode:extension/ms-vscode-remote.remote-containers) extension

Open your project and from the orange icon and select reopen in Container and select the Python3 container.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/local.png)

![Screenshot]({{ site.url }}/assets/img/Remote-Development/reopen.png)

![Screenshot]({{ site.url }}/assets/img/Remote-Development/py3-container.png)

VS Code will download the container, mount your local directory, and expose the UI ports.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/architecture-containers.png)

You can view the progress and watch what's doing

![Screenshot]({{ site.url }}/assets/img/Remote-Development/installing_container.png)

Easy, you now have a containerised development environment that you can easily and safely do anything you want in. Have a look at the `.devcontainer/Dockerfile` to see what it does. You can add lots more, an easy one is `python3 -m pip install -r requirements.txt` which will install all your requirements at build time.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/debug_container.png)

I build this blog using a container, [check it out](https://github.com/rorymurdock/rorymurdock.github.io/.devcontainer), makes it very easy to have a consistent environment and run Jekyll.

You can combine Remote via SSH and Containers and connect to a Dockers host over SSH by setting a [remote host](https://code.visualstudio.com/docs/remote/containers-advanced#_developing-inside-a-container-on-a-remote-docker-host)

More on containers [here](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/)

## Visual Studio Codespaces

VS Codespaces is an Azure backed instance of the above with integration with Azure to spin down your instances when they're not active and provide a cloud based development environment. You can read more about it [here](https://docs.microsoft.com/en-au/visualstudio/codespaces/quickstarts/vscode).

There is also a self hosted VS Codespace option that will let you use your own machine for free which is a really cool option and lets you run a CloudBased IDE with your own VM as a worker.

![Screenshot]({{ site.url }}/assets/img/Remote-Development/codespace_register.png)

![Screenshot]({{ site.url }}/assets/img/Remote-Development/codespace_browser.png)

![Screenshot]({{ site.url }}/assets/img/Remote-Development/codespace_browser_debug.png)

https://devblogs.microsoft.com/visualstudio/bring-your-own-machine-to-visual-studio-online/

