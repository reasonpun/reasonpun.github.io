---
layout: post
title: 使用Vagrant部署Rails项目
excerpt: Vagrant 是一个自动化工具，可以在你的电脑的虚拟机里自动搭建一个开发环境。这就意味着你本地开发环境可以完全与生产服务器上保持一致，你的合作小伙伴也可以和你保持高度一致的运行环境。
tag: [Vagrant, Rails, Ruby]
---

## 简述

Vagrant 是一个自动化工具，可以在你的电脑的虚拟机里自动搭建一个开发环境。这就意味着你本地开发环境可以完全与生产服务器上保持一致，你的合作小伙伴也可以和你保持高度一致的运行环境。

自个儿的Rails的开发环境也不会随着你开发机的变化而变化。另外，你可以在分分钟内启动一个可能需要在12个月以后重新访问的项目，那时你会感谢自己在项目初期使用了这个工具。

我们会使用[Chef](https://www.chef.io/chef/)完成虚拟机开发环境的自动部署。chef会负责为我们的系统配置Ruby和所有依赖的包。非常适合快速开发（It's pretty RAD）。

BB的够多的了，让我们开始吧！

## 配置Vagrant

首先，开始执行之前确认下本机至少有1G的空闲内存，因为Vagrant会在你的虚拟机中启动一个完整的操作系统，用来执行Rails程序。

第一步，在本机安装Vagrant和VirtualBox。

  * [移步这里下载安装Vagrant](https://www.vagrantup.com/downloads.html)
  * [移步这里下载安装VirtualBox](https://www.virtualbox.org/wiki/Downloads)

虚拟机是运行在VirtualBox里的，这些都是后台运行的程序，只能通过SSH进行交互。

其次，我们需要安装Vagrant的两个插件。

  * vagrant-vbguest 自动安装各种Guest扩展程序
  * vagrant-librarian-chef 让我们可以在机器启动的时候自动执行chef

```
vagrant plugin install vagrant-vbguest
vagrant plugin install vagrant-librarian-chef-nochef
```

这个可能需要些时间才能执行完毕。

```
➜  ds git:(dev) vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Installed the plugin 'vagrant-vbguest (0.13.0)'!
➜  ds git:(dev) vagrant plugin install vagrant-librarian-chef-nochef
Installing the 'vagrant-librarian-chef-nochef' plugin. This can take a few minutes...
Installed the plugin 'vagrant-librarian-chef-nochef (0.2.0)'!
➜  ds git:(dev)

```

## 创建Vagrant配置文件

首要的，进入需要配置chef的Rails项目目录，并执行如下命令。

```
➜  ds git:(dev) vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
➜  ds git:(dev) ✗ touch Cheffile
➜  ds git:(dev) ✗ ls
Cheffile     README.md    app          config.ru    log          tmp
Gemfile      Rakefile     bin          db           public       vendor
Gemfile.lock Vagrantfile  config       lib          test
```

由此会生成Vagrantfile和Cheffile两个文件。

## Cheffile

现在，我们可以配置Chef文件了。这个文件和我们的Rails的gem 文件类似，只不过是用于Chef的。这个文件中会定义很多Chef cookbooks用来在稍后的Vagrantfile中指挥Vagrant配置我们的环境的具体内容。

我们仅仅只需要黏贴如下代码到Cheffile中：

```

site "http://community.opscode.com/api/v1"

cookbook 'apt'
cookbook 'build-essential'
cookbook 'mysql', '5.5.3'
cookbook 'ruby_build'
cookbook 'nodejs'
cookbook 'rbenv', git: 'https://github.com/aminin/chef-rbenv'
cookbook 'vim'

```

## Vagrantfile

Vagrantfile 定义了虚拟机中的操作系统和Chef的配置项。

我们会使用Ubuntu 16.04 xenial 64-bit和4G的内存这个版本配置（如果你用的32位的系统，只需要改成xenial32即可）。还需要打开虚拟机的3000端口，这样，我们通过本地浏览器就可以访问虚拟机上的Rails服务了。不过至少到现在为止，我们已经在虚拟机中通过Chef配置好了Ruby2.3.1和MySQL。

Vagrantfile文件的内容如下：

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use Ubuntu 16.04 Xenial Xerus 64-bit as our operating system
  config.vm.box = "ubuntu/xenial64"

  # Configurate the virtual machine to use 4GB of RAM
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "4048"]
  end

  # Forward the Rails server default port to the host
  config.vm.network :forwarded_port, guest: 3000, host: 3000

  # Use Chef Solo to provision our virtual machine
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = ["cookbooks", "site-cookbooks"]

    chef.add_recipe "apt"
    chef.add_recipe "nodejs"
    chef.add_recipe "ruby_build"
    chef.add_recipe "rbenv::user"
    chef.add_recipe "rbenv::vagrant"
    chef.add_recipe "vim"
    chef.add_recipe "mysql::server"
    chef.add_recipe "mysql::client"

    # Install Ruby 2.3.1 and Bundler
    # Set an empty root password for MySQL to make things simple
    chef.json = {
      rbenv: {
        user_installs: [{
          user: 'vagrant',
          rubies: ["2.3.1"],
          global: "2.3.1",
          gems: {
            "2.3.1" => [
              { name: "bundler" }
            ]
          }
        }]
      },
      mysql: {
        server_root_password: ''
      }
    }
  end
end
```

## 执行Vagrant

至此，我们已经配置好了Vagrant和Chef。我们将启动Vagrant虚拟主机，并通过ssh登陆进去！

```
# The commented lines are the output you should see when you run these commands

vagrant up
#==> default: Checking if box 'ubuntu/xenial64' is up to date...
#==> default: Clearing any previously set forwarded ports...
#==> default: Installing Chef cookbooks with Librarian-Chef...
#==> default: The cookbook path '/Users/chris/code/test_app/site-cookbooks' doesn't exist. Ignoring...
#==> default: Clearing any previously set network interfaces...
#==> default: Preparing network interfaces based on configuration...
#    default: Adapter 1: nat
#==> default: Forwarding ports...
#    default: 3000 => 3000 (adapter 1)
#    default: 22 => 2222 (adapter 1)
#==> default: Running 'pre-boot' VM customizations...
#==> default: Booting VM...
#==> default: Waiting for machine to boot. This may take a few minutes...
#    default: SSH address: 127.0.0.1:2222
#    default: SSH username: vagrant
#    default: SSH auth method: private key
#    default: Warning: Connection timeout. Retrying...
#==> default: Machine booted and ready!
#==> default: Checking for guest additions in VM...
#==> default: Mounting shared folders...
#    default: /vagrant => /Users/chris/code/test_app
#   default: /tmp/vagrant-chef-1/chef-solo-1/cookbooks => /Users/chris/code/test_app/cookbooks
#==> default: VM already provisioned. Run `vagrant provision` or use `--provision` to force it

vagrant ssh
#Welcome to Ubuntu 16.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)
#
# * Documentation:  https://help.ubuntu.com/
#
# System information disabled due to load higher than 1.0
#
#  Get cloud support with Ubuntu Advantage Cloud Guest:
#    http://www.ubuntu.com/business/services/cloud
#
#
#vagrant@vagrant-ubuntu-xenial-64:~$

```

首次启动vagrant需要很长很长很长时间，因为需要准备chef文件中设置的诸多配置项，之后再次执行的话，不再执行Chef就会快很多了。

如果修改了Vagrantfile和Cheffile文件，你需要使用如下命令重新使机器配置生效

```
vagrant provision
```

## 在Vagrant中使用Rails

Vagrant会创建一个名字为/vagrant的共享文件夹，可以是虚拟机和本机之间做文件共享。如果进入/vagrant目录下并执行ls命令，会看到Rails项目的所有文件。

```
bundle
```
在虚拟机中安装所有需要的gem。

```
rbenv rehash
```
确保我们刚刚安装的gem是可用的。

```
rake db:create && rake db:migrate
```
创建和迁移数据库。

该目录下的Rails服务器通过3000端口对外提供服务。可以通过localhost:3000访问Rails。

## 结语

Vagrant是一个非常Diao的工具，非常方便，通过见得的Chef配置文件，我们可以随时随地使用。
如果你需要再次配置一个Vagrant机器，或者是你的合作小伙伴需要配置一个的话，你们只需要执行

```
vagrant up
```
即可。

Q&A

  * 执行vagrant up命令失败
    A：请检查 VirtualBox的版本是否与Vagrant下载扩展的版本一致，我本地使用的是5.0.24 。

    ```
    ==> default: Machine booted and ready!
    [default] GuestAdditions versions on your host (4.3.28) and guest (5.0.24) do not match.
    mesg: ttyname failed: Inappropriate ioctl for device
    mesg: ttyname failed: Inappropriate ioctl for device
    ```

    另外Vagrant 1.8.4与 VirtualBox5.1.x版本不兼容。

    ```
    ➜  ds git:(dev) ✗ vagrant up --provider=virtualbox
    No usable default provider could be found for your system.

    Vagrant relies on interactions with 3rd party systems, known as
    "providers", to provide Vagrant with resources to run development
    environments. Examples are VirtualBox, VMware, Hyper-V.

    The easiest solution to this message is to install VirtualBox, which
    is available for free on all major platforms.

    If you believe you already have a provider available, make sure it
    is properly installed and configured. You can see more details about
    why a particular provider isn't working by forcing usage with
    `vagrant up --provider=PROVIDER`, which should give you a more specific
    error message for that particular provider.
    ➜  ds git:(dev) ✗ cat /etc/profile
    ```

  * 需要安装ruby-dev

    ```
    sudo apt-get install ruby-dev
    ```

  * PostgresSQL 头文件

    ```
    sudo apt-get install libpq-dev
    ```
