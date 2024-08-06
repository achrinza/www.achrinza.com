---
title: Protecting against malicious dependencies with Flatpak
author: Rifa Achrinza
date: 2024-08-05
layout: post
---

[Flatpak](https://flatpak.org/) is a great tool for installing user applications in a self contained environment without polluting the host machine. This mitigates a range of issues, notably [depenency hell](https://en.wikipedia.org/wiki/Dependency_hell). There's even  packages for popular code editors such as VSCode/VSCodium and Emacs.

In fact, I code in Emacs purely through Flatpak. Not for the above reasons per-se, but for something more valuable - _to contain the blast radius_.

## The problem with modern programming

An issue that plagues vitually every software development project nowadays is the abundant use of third-party dependencies. This issue is more prevalent in some ecosystems such as Node.js where many small packages tend to be preferred over larger, batteries-included packages. This means more code written by many different people.

It's impossible to vet every single dependency and the intentions of their maintainers. Hence it becomes an exercise of trust that the maintainers protect their publishing credentials from compromise or to not go rogue. But every now and then, [it happens](https://www.bleepingcomputer.com/news/security/big-sabotage-famous-npm-package-deletes-files-to-protest-ukraine-war/).

These malicious packages can be quickly authored and easily compromise a developer's machine and privileged credentials. Hence, we need a broad, defensive approach towards mitigating the impact of such a compromise.

## Flatpak sandboxing

This is where Flatpak sandboxing comes in. It restricts an application's ability to access certain resources such as network, dbus, devices, and filesystem - the last one's what we'll talk about.

In a perfect world, Flatpak apps would be configured to use [Flatpak Portals](https://docs.flatpak.org/en/latest/portal-api-reference.html) - A mechanism to give granular, just-in-time access to a certain directory or file in the host file system with explicit user consent. However, anecdotally most apps don't support Portals. Furthermore, the subtle incompatibilities with certain tools and the inability to auto-revoke the access at the end of each session makes the experience feel especially clunky.

As a compromise, it is by convention, that most Flatpak developer-centric apps are pre-configured with broad filesystem access. This provides out-of-the-box usability without the need for Portals. This does mean that the app environment can interact with  most of the host filesystem.

We can see the permissions with `flatpak info --show-permissions <app-id>`:

```sh
$ flatpak info --user --show-permissions org.gnu.emacs
[Context]
shared=network;ipc;
sockets=x11;pulseaudio;
filesystems=/var/tmp;/tmp;host;

[Session Bus Policy]
org.freedesktop.Flatpak=talk
```

As we can see, my Flatpak-installed Emacs has access to the entire `host` filesystem. If you'd like to learn more about these permissions, the [Flatpak docs](https://docs.flatpak.org/en/latest/sandbox-permissions.html#filesystem-access) is a great reference.

Also note that I'm using the `--user` flag as I did not install Emacs system-wide. If your app is installed system-wide, omit this flag.

## Clamping down with Flatpak overrides

As the name suggests, Flatpak overrides allow us to override the default permissions. If we want to block `host` filesystem access, we simply counter it with its negative:

```sh
$ flatpak override --user --nofilesystem=host org.gnu.emacs
```

Running the same `flatpak info` command as above, we can see the `host` filesystem is no longer there:

```
$ flatpak info --user --show-permissions org.gnu.emacs
[Context]
shared=network;ipc;
sockets=x11;pulseaudio;
filesystems=/var/tmp;/tmp;

[Session Bus Policy]
org.freedesktop.Flatpak=talk
```

> Note: When performing `flatpak override`, double-check that the app name is spelled correctly. Flatpak does not verify if the app actually exists! To fix this, simply re-run the command with the correct app name. Though there's still a stray override config that we might want to delete - I'll explain how to find and delete it below.

If we now create a new file in Flatpak Emacs, the file does not appear in the host filesystem. Instead, it's saved in an ephemeral filesystem that gets wiped when this instance of Emacs is closed.

## Giving granular access

To give granular access to the filesystem, we need to decide what that granularity is. For me, access is given on a per-project basis (i.e. per-Git repository). This is a good compromise as it prevent unwanted access and alteration of other projects whilst allowing the tools in the active project to function correctly.

With this setup, whenever we want to work on a project we can simply run these commands:

```sh
$ cd <project directory>
$ flatpak run --user --filesystem="$(pwd)" org.gnu.emacs
```

The main difference is the addition of `--filesystem="$(pwd)"` which gets the current working directory (via `pwd`), and tells Flatpak to perform a one-time override to allow filesystem access. This override only applies to that instance of the app, and we can open multiple projects on separate instances with access to different parts of filesystem - no cross-mingling of permissions.

We can even package this into a wrapper called `emc` (my shorthand for Emacs):

```sh
$ sudo su -c 'cat <<EOF >>/usr/local/bin/emc
#!/bin/sh
flatpak run --user --filesystem="$(pwd)" org.gnu.emacs @
EOF'

$ sudo chmod +x /usr/local/bin/emc
```

Now we can give just-in-time, granular access just as easily as starting any other programme:

```shpp
$ cd <project directory>
$ emc
```

## Where are my override files?

`flatpak override` is quite rudementary, and it's easy to make mistakes like typos in the app name. This can lead to stray or cluttered override files. Their location depends if the override was done with the `--user` or `--system` flag (no flag defaults to `--system`).

- For apps installed system-wide: `/var/lib/flatpak/overrides/<app-id>`
- For apps installed per-user: `~/.local/share/flatpak/overrides/<app-id>`

If we inspect the file, we can see that it contains our overrides, and has the same syntax as the output of `flatpak info --show-permissions`:

```
$ cat ~/.local/share/flatpak/overrides/org.gnu.emacs
[Context]
filesystems=!host;
```

We can modify this file by hand, or even delete it entirely if we want to get back the default permissions.

## Limitations

There are some limitations that I've personally stumbled into:

1. With certain apps, the overrides is unable to handle multiple instances. I've previously observed this with VSCodium, but I'm not sure if this issues still persists. It does not affect Emacs.
2. For apps that use Flatpak Portals, [it is not possible to disable Portals](https://github.com/flatpak/flatpak/issues/3977), so you'll need to manaully revoke the portals each session with `flatpak document-unexport`. Automating this is left as an exercise for the reader.

## In conclusion

Flatpak can be a simple but powerful tool to protect yourself from the impact of running malicious code. It's not a full replacement for a true-sandboxed virtual machine environment nor for a proper antivirus solution, but it can signficantly mitigate the reach of the malware. When you get compromised, killing the Flatpak instance, then re-cloning the project should be enough for trivial cases.

This just-in-time approach to filesystem sandboxing is something I use daily, and its ease of use has made it an easy habit to adopt with little downsides. If you're just starting out on your Flatpak journey, or have already used it for a while, I hope this guide help you learn how you can control Flatpak to improve your security posture.
