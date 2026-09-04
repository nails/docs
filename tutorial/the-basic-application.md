---
description: >-
  In this section we set up a new Nails application using the Docker environment
  and browse to the welcome page in a browser.
---

# The basic application

## Before you start

We'll be setting up a new Nails project using the [Nails Docker environment](http://docker.nailsapp.co.uk). Make sure you have Docker installed and running for your system. Configuring and using Docker is outside the scope of this guide.

{% embed url="https://www.docker.com/get-started" %}

{% hint style="warning" %}
If you already have Docker containers running make sure any containers bound to ports 80, 443 (HTTP/S), and 3306 (MySQL) are stopped.
{% endhint %}

## Install the command line tool

If you haven't already, install the [Nails Command Line Tool](../command-line-tool.md), it does most of the heavy lifting for you and is the easiest way to get started.

{% content-ref url="../command-line-tool.md" %}
[command-line-tool.md](../command-line-tool.md)
{% endcontent-ref %}

## Set up the project

The command line tool will install the [Docker Environment](http://docker.nailsapp.co.uk) to an empty directory of your choosing, from there you can spin up the project and begin working.

```bash
# Create a new working directory and move into it
mkdir ~/nails-tutorial
cd ~/nails-tutorial

# Create a new nails project
nails new:project
```

The tool will download and extract the [Docker Environment](http://docker.nailsapp.co.uk) and prompt you to spin up the environment:

```bash
# Build and run the project containers
make up
```

First you will see Docker creating the containers, `make up` might take some time the first time you run it.

Because we have not yet created our `./www` directory (where our code will live) `make up` automatically executes `make build` once it has finished bringing the containers online. `make build` will install all the app's dependencies and compile any assets (i.e CSS and JS).

{% hint style="info" %}
`make build` executes `./www/scripts/build.sh` - inspect that file to understand what is happening.
{% endhint %}

## Hello World!

Once built, you should be able to navigate to [https://localhost](https://localhost) where you will see the Nails welcome page.

![](<../.gitbook/assets/Screenshot 2020-03-01 20.33.24.png>)

Congratulations, you have set up a brand new Nails project! Next, we will understand [routes and modules](routes-controllers-and-views.md).

{% hint style="warning" %}
You will likely encounter SSL errors when you visit the URL. This is due to the Docker environment using a self-signed SSL certificate, which is considered insecure.&#x20;

[Read more about this behaviour, and how to trust the certificate on the Nails Docker Environment docs.](https://docker.nailsapp.co.uk/containers/web-server#ssl)
{% endhint %}
