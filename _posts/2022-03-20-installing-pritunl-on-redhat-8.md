---
title: "Installing Pritunl on Redhat 8"
date: 2022-03-20T09:59:42+01:00
categories: [homelab, linux]
tags: [pritunl,ssl vpn, linux, redhat 8, selinux, homelab]
---

A quick write-up on installing Pritunl on RedHat 8.

<!--more-->
## Intro

Pritunl is a distributed OpenVPN, IPsec and WireGuard Server that I've been using for a number of years as my goto solution for:

- accessing my lab when working remote
- tunneling my webtraffic whenever I'm forced to use a unknown wifi network

My  Pritunl server was still running on a older version of CentOS so it was time for a rebuild.
For the new build I chose to use one of my free RedHat licenses you can get [here](https://developers.redhat.com/articles/faqs-no-cost-red-hat-enterprise-linux#)
This post will cover the installation not the configuration of Pritunl since it's so simple to set up once you get it installed.

## Installation

So let's get this thing up and running.

Enable the mongodb and pritunl repos:

```bash
sudo tee /etc/yum.repos.d/mongodb-org-5.0.repo << EOF
[mongodb-org-5.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/8/mongodb-org/5.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-5.0.asc
EOF
```

```bash
sudo tee /etc/yum.repos.d/pritunl.repo << EOF
[pritunl]
name=Pritunl Repository
baseurl=https://repo.pritunl.com/stable/yum/oraclelinux/8/
gpgcheck=1
enabled=1
EOF
```

```bash
gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 7568D9BB55FF9E5287D586017AE645C0CF8E292A
gpg --armor --export 7568D9BB55FF9E5287D586017AE645C0CF8E292A > key.tmp; sudo rpm --import key.tmp; rm -f key.tmp
```

Activate the [epel](https://www.redhat.com/en/blog/whats-epel-and-how-do-i-use-it) repo:

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

I'm running the server behind a firewall so I'm disabling the firewall on the host.

```bash
sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
```

Update the system

```bash
sudo dnf -y update
```

Install [wireguard](https://www.wireguard.com/) if you need it

```bash
sudo dnf -y install wireguard-tools
```

Install pritunl and mongodb

```bash
sudo dnf -y install pritunl mongodb-org
```

The pritunl documentation mentions using the newer pritunl openvpn package, so we comply

```bash
sudo dnf --allowerasing install pritunl-openvpn
```

Start and enable the mongodb and pritunl services

```bash
sudo systemctl enable mongod pritunl
sudo systemctl start mongod pritunl
```

So if things went according to plan, you should be able to see the pritunl website up and running.
Go back to your ssh session and run the following

```bash
sudo pritunl setup-key
```

Paste the setup key in the webinterface and continue to the next step, which is creating the initial pritunl login

```bash
sudo pritunl default-password
```

Login using those credentials and finish the pritunl setup which consists of the following:
-Enter a new password for the pritunl user
-Creating your first VPN user
-Creating an organization
-Creating the VPN server
-Attaching the organization to the VPN server
-Starting the VPN server
Done!

## Bonus I: running the pritunl webinterface on a different port

I've kept selinux active on the box even though multiple threads on the pritunl forum advise people to turn if off.
The Pritunl webinterface runs on port 443 by default. It is possible to change this port but we need some additional tools
to configure selinux:

```bash
sudo dnf install policycoreutils-python-utils
```

Now let's say you want to change the webinterface port from 443 to 1234:

```bash
sudo semanage port -a -t http_port_t -p tcp 1234
```

Then go into the webinterface and change the port by clicking on settings in the right top corner. You will be logged out and back in after it has finished changing its port.

## Bonus II: Update the server automatically

Having a machine that updates itself helps me sleep well at night.

```bash
sudo dnf install dnf-automatic
```

Edit the dnf-automatic config file to your liking

```bash
sudo vi /etc/dnf/automatic.conf
```

Unhash/enable (at least) the following settings

```bash
[commands]
#  What kind of upgrade to perform:
default                            = all available upgrades

download_updates = yes

apply_updates = yes
```

After you are satisfied with the configuration you put in place, enable the service

```bash
sudo systemctl enable dnf-automatic.timer
sudo systemctl start dnf-automatic.timer
```

Alternatively, if you want the updates to be **downloaded** at a specific time, edit the following file

```bash
sudo vi /usr/lib/systemd/system/dnf-automatic.timer
```

If you want to **install** the updates at a specific time, edit this file

```bash
sudo vi /usr/lib/systemd/system/dnf-automatic-install.timer
```

and enable the install timer

```bash
sudo systemctl start dnf-automatic-install.timer
sudo systemctl enable dnf-automatic-install.timer
```

To test the dnf config

```bash
sudo dnf-automatic
```

The logging of dnf-automatic is limited. You can view the installed updates in /var/log/dnf.rpm.log file. This will show what packages are upgraded and installed

To check which services need to be restarted after an update, enter the following command

```bash
sudo dnf needs-restarting
```

Get an overview of the timers

```bash
sudo systemctl list-timers *dnf-*
```

For more info on dnf-automatic, check [here](https://docs.oracle.com/en/operating-systems/oracle-linux/software-management/sfw-mgmt-UpdateSoftwareonOracleLinux.html#update-software) and [here](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/managing-software-packages_configuring-basic-system-settings)
## Final Thoughts
There you have it! Pritunl should be up and running now. The system should also update itself whenever updates are available.
Another thing I might look at is rebooting the server each week during the night.
As always, if you have any suggestions feel free to contact me. Until next time!

