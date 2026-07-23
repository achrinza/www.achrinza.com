---
layout: post
title: "Running BOSH Lite on Windows 11 the easy way"
categories: cloudfoundry-bosh
author: Rifa Achrinza
excerpt: >-
  Locally deploy BOSH on Windows with WSL 2 mirrored mode and
  with minimal tinkering.
---

- Table of Contents
{:toc}


## A Primer on BOSH Lite

Cloud Foundry [BOSH](https://bosh.io/docs/) is a powerful tool that provides a
standard for packaging software, deploying it, and especially for managing
day-2 operations (including health checks, gradual rollouts, self-healing, and
metrics!).

Getting started with BOSH can seem quite daunting as production setups assume
you're using it to manage a bunch of virtual machines on a platform like AWS
or vSphere. But don't fret! For local development, there's *BOSH Lite*, a
lightweight setup crammed into a single VM.

```text
                                                                   
                                                                   
                      *** BOSH (Typical) ***                       
                                                                   
                      ┌───────────────────────────────────────────┐
       ┌──────────────┼────────┐  ┌───────────┐                   │
┌──────┼─────┐        │┌───────┼──┼────┐┌─────┼──────────────────┐│
│ ┌────┴───┐ │        ││┌──────▼──▼───┐││┌────▼─────┐┌──────────┐││
│ │BOSH CLI├─┼─┐  ┌───┼┼┤BOSH Director││││BOSH Agent││Deployment│││
│ └────────┘ │ │  │   ││└─────────────┘││└──────────┘└──────────┘││
│Dev Machine │ │  │   ││Virtual Machine││    Virtual Machine     ││
└────────────┘ │  │   │└───────────────┘└────────────────────────┘│
               │  └───►                                           │
               └──────►                 Platform                  │
                      └───────────────────────────────────────────┘
                                                                   
                                                                   
                         *** BOSH Lite ***                         
                                                                   
                     ┌────────────────────────────────────────────┐
       ┌─────────────┼────────┐                                   │
       │             │┌───────┼──────────────────────────────────┐│
       │             ││       │  ┌───────────┐                   ││
┌──────┼─────┐       ││       │  │     ┌─────┼──────────────────┐││
│ ┌────┴───┐ │       ││┌──────▼──▼───┐ │┌────▼─────┐┌──────────┐│││
│ │BOSH CLI├─┼──┐    │││BOSH Director│ ││BOSH Agent││Deployment││││
│ └────────┘ │  │    ││└──────┬──────┘ │└──────────┘└──────────┘│││
│Dev Machine │  │    ││       └────────►    Garden Container    │││
└────────────┘  │    ││                └────────────────────────┘││
                │    ││             Virtual Machine              ││
                │    │└──────────────────────────────────────────┘│
                └────►                  Platform                  │
                     └────────────────────────────────────────────┘

```

Compared to a typical setup, BOSH Lite swaps the BOSH Director's Cloud
Provider Interface's (CPI) Provider - the abstraction code that enables BOSH
to interact with platforms - for Warden (used to interact with Cloud Foundry's
own container runtime, ~~Warden~~ ahem,
[Guardian](https://github.com/cloudfoundry/garden-runc-release)) while the
ephemeral Bootstrap BOSH (that's `bosh create-env`) retains the original
platform's CPI Provider. This creates a split-CPI Provider setup which can
seem confusing at first. But as the diagram above shows, it's actually not too
different!

BOSH Lite can used on top of any BOSH-supported platforms, but more likely
than not it's used on either VirtualBox or Docker. Although Docker is the
[now-preferred](https://github.com/cloudfoundry/docs-bosh/issues/907#issuecomment-4867328298)
platform for local BOSH development, the VirtualBox support is more
[feature-complete](https://github.com/cloudfoundry/bosh-docker-cpi-release/tree/65582382f6c9ae25c3ff6868bd2ab0279924d234#todo),
better-documented, and mature.

This guide will provide steps for spinning up a Windows 11 VirtualBox BOSH
environment with the BOSH CLI via Windows Subsystem for Linux 2 (WSL 2).

Unlike [prior
guides](https://medium.com/pubsubplus/yes-you-can-run-bosh-lite-v2-on-windows-10-b55f679640b9),
this also works with the newer WSL 2 [mirrored networking
mode](https://learn.microsoft.com/en-us/windows/wsl/networking#mirrored-mode-networking)
and limits changes to within WSL 2 and one Hyper-V vSwitch and vEthernet
adapter pair.


## The problem

Most BOSH CPIs (including VirtualBox), and by extension the `bosh create-env`
 command, [do not support
 Windows](https://github.com/cloudfoundry/bosh-cli/issues/278). Therefore,
 it's recommended to run the command under WSL 2. However:

1. The VirtualBox CPI relies on the `VBoxManage` CLI and passes untransformed
    Linux paths for stemcell uploads
2. VirtualBox on WSL 2 is not supported
3. There is [no route](https://github.com/microsoft/WSL/issues/14381) between
   VirtualBox's host-only network and WSL 2 in mirrored mode.


## Prerequisites

1. Install [WSL 2](https://learn.microsoft.com/en-us/windows/wsl/install)
2. Install the [`bosh` CLI](./cli-v2-install) on the WSL 2 environment
3. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) on the
   Windows host
4. Clone [bosh-deployment](https://github.com/cloudfoundry/bosh-deployment)


## Setup

> **A word of caution**
>
> Following this guide will necessarily alter the security model such that
> the BOSH Director becomes routable to and from other virtual machines on
> the host machine rather than just the host operating system. For typical
> development machines where running VMs are trusted, this should not be an
> issue.


### Enabling the Hyper-V Windows feature

This is needed to leverage Hyper-V's vSwitch to expose a network adapter with
a routable IP address. Our VirtualBox VMs will attach to this network adapter
instead of VirtualBox's host-only network.

1. Open "Turn Windows Features on or off" from the Start Menu.
2. Tick `Hyper-V` and its sub-features.
    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/windows-feature-on-off-hyper-v.png)
3. Press OK.
4. Restart the computer when instructed.


### Create a internal Hyper-V vSwitch network

1. Open "Hyper-V Manager" from the Start Menu
2. Go to "Action" > "Virtual Switch Manager"

    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/hyper-v-open-virtual-switch-manager.png)

3. Under "Virtual Switches", click on "New virtual network switch"
4. Select "Internal" type
5. Click "Create Virtual Switch"
6. Set a name for the new vSwitch
7. Click OK

    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/hyper-v-internal-network-config.png)


### Making the network adapter routable

> **Tip**
>
> When creating a vSwitch in the previous section, a network adapter managed
> by Hyper-V (called "vEthernet") is also created. Here, we'll configure
> this adapter to be routable.

1. Open "View network connections" from the Start Menu.
2. Right-click on the newly-created network adapter and click "Properties". It
   can be identified by the vSwitch name from the previous section:
   `vEthernet (<vSwitch Name>)`.

    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/windows-network-connection.png)

3. Note down the network adapter's name listed under "Connect using".

    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/windows-network-connection-properties.png)

    > **Note**
    >
    > Your network adapter name may differ from the example.

4. Double-click "Internet Protocol Version 4 (TCP/IPv4)" in the items list.
5. Select "Use the following IP address:".
6. Set the IP address and subnet mask.

    > **Note**
    >
    > This example uses the subnet `10.150.250.0/24`, though it can be
    > configured to any unused subnet. `192.168.56.0/24` is not used to avoid
    > conflict with the default VirtualBox host-only network.

    ![](https://static.achrinza.com/media/articles/running-bosh-lite-on-windows-11/windows-network-connection-ipv4-properties.png)

7. Click OK on both dialog boxes to save changes.

### Adding `VBoxManage` CLI to WSL 2

In the WSL 2 environment, add the [VBoxManage WSL 2
wrapper](https://git.sr.ht/~achrinza/vboxmanage-wsl2-wrapper):

```shell
curl -LO  https://git.sr.ht/~achrinza/vboxmanage-wsl2-wrapper/blob/main/VBoxManage.sh
chmod +x VBoxManage.sh
sudo mv VBoxManage.sh /usr/local/bin/VBoxManage
```


### Deploying the BOSH Director

Because the networking stack differs from what `bosh-deployment` covers,
we'll need to create a new Ops File:

```yaml
# ./virtualbox-wsl2.yml
- name: /networks/name=default/subnets/0/cloud_properties?
  type: replace
  value:
    type: bridged
    # Replace this with your network adapter name:
    name: "Hyper-V Virtual Ethernet Adapter #4"
```

> **Note**
>
> Do not confuse this with the *vSwitch* name. The network adapter name is
> autogenerated and should start with `Hyper-V Virtual Ethernet Adapter #`.

From there, we can follow the [standard VirtualBox
deployment](https://bosh.io/docs/bosh-lite/#install), appending
our new Ops File and altering the `internal` network variables:

```diff
 bosh create-env ~/workspace/bosh-deployment/bosh.yml \
   --state ./state.json \
   -o ~/workspace/bosh-deployment/virtualbox/cpi.yml \
   -o ~/workspace/bosh-deployment/virtualbox/outbound-network.yml \
   -o ~/workspace/bosh-deployment/bosh-lite.yml \
   -o ~/workspace/bosh-deployment/bosh-lite-runc.yml \
   -o ~/workspace/bosh-deployment/uaa.yml \
   -o ~/workspace/bosh-deployment/credhub.yml \
   -o ~/workspace/bosh-deployment/jumpbox-user.yml \
+  -o ./virtualbox-wsl2.yml \
   --vars-store ./creds.yml \
   -v director_name=bosh-lite \
-  -v internal_ip=192.168.56.6 \
-  -v internal_gw=192.168.56.1 \
-  -v internal_cidr=192.168.56.0/24 \
+  -v internal_ip=10.150.250.6 \
+  -v internal_gw=10.150.250.1 \
+  -v internal_cidr=10.150.250.0/24 \
   -v outbound_network_name=NatNetwork
```

Then, we can alias the BOSH environment:

```shell
export BOSH_CLIENT=admin
export BOSH_CLIENT_SECRET="$(bosh int ./creds.yml --path /admin_password)"
bosh alias-env vbox -e 10.150.250.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)
```

## Next steps

Now we have a functioning BOSH environment! To start testing out BOSH, we can
try the classic ZooKeeper deployment (with [some
modifications](https://github.com/cppforlife/zookeeper-release/pull/16)):

```sh
bosh -e vbox update-cloud-config ~/workspace/bosh-deployment/warden/cloud-config.yml
# Get the latest `upload-stemcell` command from: https://bosh.io/stemcells/bosh-warden-boshlite-ubuntu-noble
bosh upload-stemcell --sha1 cda735071b3d91349b90ee3309ef8d16cfe2dc74 \
  "https://bosh.io/d/stemcells/bosh-warden-boshlite-ubuntu-noble?v=1.460"
bosh -e vbox -d zookeeper deploy <(curl -L https://github.com/achrinzafork/zookeeper-release/raw/b9a3df6abefb2a853738c99c200eaa93892f8f55/manifests/zookeeper.yml)
bosh -e vbox -d zookeeper run-errand smoke-tests
```


## Final thoughts

BOSH is an underrated, mature tool for managing VMs, but getting started on
Windows is more confusing than it needs to be. For me, this meant more time
spent trying to set up BOSH before being able to actually learn it. The setup
described above is what I use today for local BOSH development on Windows - A
quick way to get started with minimal fuss. That's why I wrote this article
with a BOSH Lite explainer and a networking guide; It's what I wish I knew
when first starting with BOSH, and I hope it comes in handy for those who want
to get started quickly.
