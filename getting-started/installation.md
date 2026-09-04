---
description: This page covers how to install Nails and get up and running.
---

# Installation

To set up a new Nails project use the [Command Line Tool](../command-line-tool.md) to install the app skeleton and dependencies into an empty directory

```bash
# Create an empty working directory
make ~/my-project
cd ~/my-project

# Install Nails
nails new:project
```

{% hint style="info" %}
By default the [Docker Environment](http://docker.nailsapp.co.uk) will be used. If you do not wish to use this then pass the `--no-docker` flag.
{% endhint %}

Once installed, executing `make up` will build the project and install dependencies. Once up and running you can navigate to [https://localhost](https://localhost) in your browser where you will see the Nails welcome screen.

![](<../.gitbook/assets/Screenshot 2020-03-01 20.33.24.png>)

{% hint style="warning" %}
You will likely encounter SSL errors when you visit the URL. This is due to the Docker environment using a self-signed SSL certificate, which is considered insecure.&#x20;

[Read more about this behaviour, and how to trust the certificate on the Nails Docker Environment docs.](https://docker.nailsapp.co.uk/containers/web-server#ssl)
{% endhint %}
