---
description: >-
  The Nails command line tool is a useful utility to make working with Nails
  applications and the console module easy and intuitive.
---

# Command Line Tool

The Nails Command Line Tool is the easiest way to manage basic, repetitive aspects of a Nails Application.

## Installation

Installation of the tool is officially supported via the following methods.

### [Homebrew](https://brew.sh)

```bash
brew tap nails/utilities
brew install nails
```

### [Composer](http://getcomposer.org/)

```bash
composer global require nails/command-line-tool
```

{% hint style="warning" %}
In order to function properly your `$PATH` must be set correctly to include the relevant Composer binary directories.

If you are getting errors such saying `nails` cannot be found make sure your `$PATH` contains `~/.composer/vendor/bin` by executing the following:

```bash
echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc
```

You should then logout and login again or execute:

```bash
source ~/.bashrc
```

Substitute `.bashrc` with your shell's appropriate config file if it is different.
{% endhint %}

### Manual

If you wish to install the tool manually then you may do so by installing the binaries from the `./dist` directory of the [GitHub repository](https://github.com/nails/command-line-tool/tree/master/dist).

## Usage

The `nails` tool acts both as a standalone application to make creating Nails sites easier, as well as a proxy for the bundled [console module](modules/console.md) (if installed).

### Standalone

#### **Creating a new Nails project**

```
nails new:project
```

Creates a new Nails project in the current directory using the [Docker skeleton](getting-started/docker.md).

**Creating a new Nails Module**

```bash
nails new:module
```

This creates a new Nails module for distribution via Composer. [Read more about building modules for Nails](contributing/modules.md).

#### **Further reading**

The tool is built using the [Symfony Console](https://symfony.com/doc/current/components/console.html) component, execute `nails --help` for further information, or to see additional commands available.

### Proxy

If `nails` is called in a directory which contains a Nails application then it will proxy the app's [console module](modules/console.md) (if installed).
