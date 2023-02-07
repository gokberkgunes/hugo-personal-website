---
title: "Python User Package Management"
date: 2023-02-07T17:10:39+03:00
authors: ["Gokberk Gunes", ]
tags: [python]
draft: false
---
In this post, we will delve into how to manage  packages in a computer
properly. We will not deal with some fancy package manager like `anaconda`; the
post will directly use `pip` and package manager of the operating system.
## Quick Overlook on Pip
Here we will very briefly talk about usage of `pip`. There are three commands
we should know; how to install a package, how to remove a package, list all the
installed packages.

### System-wide (Root User)
Beware that we am running all of these as root user by calling `doas`.
```sh
umask 022 # Allow non-root access to installed files.
doas pip install $PACKAGE_NAME # Install a python package.
doas pip uninstall $PACKAGE_NAME # Unnstall a python program.
doas pip list # Show all system-wide installed packages.
```

### User-site (Normal User)
```sh
pip install $PACKAGE_NAME # Install a program.
pip uninstall $PACKAGE_NAME # Uninstall a program.
pip list --user # Show all user-wise installed packages.
```

## Clash Between Pip and Package Manager
Generally, it is preferable toto install packages through the package managers.
If we have installed the package through the package manager before, the
package will be reinstalled. However, if we have used system-wide `pip` to
install this package; then, `pacman` will try to install the package and throw
an error. So far so good, we know that we already installed this python package
then. So there is nothing to worry about, right? Unfortunately, there is still
a problem hence this post. Now, assume that we are trying to install completely
seperate program to the computer through `pacman`. This package wants a python
package we have already installed through `pip`. Now we are unable to install
this new program. Now we need to find the required package, remove it with
`pip` as a root user before the installation.

## Normal User Installations: Avoiding Pollution at ~/.local
At this point, if we try to install packages in the user-site we will find
bunch of files in user configuration folder under the home directory. The folder which holds
binary files will get a lot of new executables that we did not intend to store.
Similarly, in the library folder under the `.local` directory, automatically
created files will be waiting us.

Now, let's check where does `python` install folder as in user-site manner. For
this, we may check the directory  using below command.
```sh
python -m site
```
This command will return `USER_BASE` and `USER_SITE` variables to us. For me,
the default location points to `~/.local`, where we have all the other
configuration files.

### Change user-site
In order to avoid the littering, we need to pick a folder to install all user
files. Generally, selecting `/opt` folder for user installed files is a good choice.
```sh
mkdir -p /opt/pythonenv
```
Add this folder to environmental variables, in `profile` folder.

```txtt
$XDG_CONFIG_HOME/zsh/.zprofile
```
```txtb
export PYTHONUSERBASE=/opt/pythonenv
```

Now, if we install something with `pip install --user` we will find all the
files are in `/opt/pythonenv`. Yet, `pip` will warn us that
`/opt/pythonenv/bin` is not on our `PATH`. This means we cannot run the
installed binaries under this folder.

Now, we need to include the `/opt/pythonenv/bin` folder to the `PATH` to run
binaries. For this reason, we need to change `profile` file as shown below.
```txtt
$XDG_CONFIG_HOME/zsh/.zprofile
```
```txtb
export PATH="$PATH:/opt/pythonenv/bin"
```



