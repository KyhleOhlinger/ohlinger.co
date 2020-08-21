---
layout: post
title: Automation Series Part 1&#58; Vagrant & Ansible
date: 2020-07-25 12:00:00 +0200
img: mystery.jpg # Add image post (optional)
description: Hi all, My name is Kyhle Öhlinger and this blog post forms part of my personal blog. If you enjoy any of the posts, feel free to reach out and let me know :) 
tags: [InfoSec, Technical, Automation_Series]
---

This blog post will be the first in the series where I try to automate a "mystery" process. While the initial blog posts will not provide any specific details, they will provide the building blocks used during the development process and technologies I learnt along the way.

<p class="imgRight">
<img src="/assets/img/Mystery/questionmark.png" width="120" style="border-radius: 50%;"/>
The Mystery icon will differentiate the Automation Series blog posts from the rest. The outcome of this series will be revealed in the final blog post with links to the (hopefully) completed Open Source project that I am currently working on - the actual icon will be revealed along with the Open Source project.
</p>

## Part 1: Vagrant & Ansible

If you are interested in learning about Vagrant and Ansible, this blog post will take you through the basics. This post assumes that you have the following installed:
* Vagrant
* Ansible
* VirtualBox

You can confirm that both Vagrant and Ansible are installed using the following commandline arguments:

### Vagrant: 
```bat
vagrant --version

Vagrant 2.2.9
```

### Ansible: 
```bat
ansible --version

  ansible 2.9.10
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/kyhle/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.7 (default, Jun  4 2020, 15:43:14) [GCC 9.3.1 20200408 (Red Hat 9.3.1-2)]
```
During testing, I used the above versions of both Ansible and Vagrant and I cannot guarantee that it will work with previous versions.

## What is Vagrant and how does it work?

Vagrant is a tool to manage virtual machine environments, and allows you to configure and use reproducible work environments on top of various platforms. It also has integration with Ansible as a provisioner for these virtual machines. Vagrant makes use of a `Vagrantfile` which you can configure to suit your needs. Even though this is described in detail in the Vagrant documentation, I am going to provide a basic `Vagrantfile` below which will spin up a Ubuntu 18.04 LTS virtual machine.

### Creating a Vagrant File:

Before diving in to the file, here are some notes on the configuration:
* **hashicorp/bionic64** --  This line will provide you with a fully fledged virtual machine in VirtualBox running Ubuntu 18.04 LTS 64-bit. 
* **Version "2"** -- This setting represents the configuration for Vagrant 1.1+ leading up to 2.0.x.
* **disksize** -- If you want to make use of the `disksize` variable, you need to run `dnf remove vagrant-libvirt` which will remove the `libvirt` library which comes with Vagrant by default.

The following command will install the required dependencies for `disksize`:

```bat
vagrant plugin install vagrant-disksize

Installing the 'vagrant-disksize' plugin. This can take a few minutes...
Fetching vagrant-disksize-0.1.3.gem
Installed the plugin 'vagrant-disksize (0.1.3)'!
```

Once you have the above installed, you will be able to make use of the `disksize` configuration settings below. This is not required and you can remove it if you want to. In order to create a Vagrant file, you need to do the following:
  1. Create a directory in which you will store all of your configuration files, e.g. VagrantProject.
  2. Create an empty file called `Vagrantfile` -- the file should not have a file extension in the name.
  3. Copy the following text into the file.
 
```ruby
# Base Vagrant File for UbuntuVM
# NOTE: If you want to speed this process up, you can download the vm and add it to the config instead.
## In commandline: vagrant box add downloaded_vm
## In config file: config.vm.box = "downloaded_vm" 
 
Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"
    config.disksize.size = "10GB"
    config.vm.hostname = “Some_Hostname”
    config.vm.provider :virtualbox do |v, override|
        v.name = "Some_VM_Name"
        v.gui = true
        v.cpus = 2
        v.memory = 2048
        v.customize ["modifyvm", :id, "--vram", 64]
    end
config.vm.provision "shell", reboot: true
end
```

Now that you have a Vagrant file, you can run it using the following commands:

1. In commandline, move into the directory (e.g. cd VagrantProject)
2. In commandline, run `vagrant up`. This will start the virtual machine (VM), and run the provisioning playbook (on the first VM startup).


If you don't have the VM already installed on the host, the output should look similar to this:
```bat
vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'hashicorp/bionic64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'hashicorp/bionic64'
    default: URL: https://vagrantcloud.com/hashicorp/bionic64
==> default: Adding box 'hashicorp/bionic64' (v1.0.282) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/hashicorp/boxes/bionic64/versions/1.0.282/providers/virtualbox.box
Download redirected to host: vagrantcloud-files-production.s3.amazonaws.com
==> default: Successfully added box 'hashicorp/bionic64' (v1.0.282) for 'virtualbox'!
```

If the VM is already on the host, you should see something similar to the following:
```bat
vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'hashicorp/bionic64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'hashicorp/bionic64' version '1.0.282' is up to date...
```

That's it, you have just created your first VM using Vagrant. The next part of this blog post will cover the integration of Ansible into the Vagrant configuration file.

## Creating an Ansible Playbook

Looking back at the basic Vagrant file above, you will see that the base configuration calls `config.vm.provision "shell"`. When adding an Ansible playbook to the configuration settings, we need to add in additional provision settings for the playbooks. The additional `config.vm.provision` sections will refer to the playbooks which will have a `.yml` extension, e.g. `my_playbook.yml`. These files need to be stored in the *same directory* as the `Vagrantfile`. 

Since Vagrant runs the provisioners once the virtual machine has already booted and is ready for access, it makes it very easy to test out the playbooks without needing to reboot the VM each time. There are a lot of Ansible options you can configure in your `Vagrantfile` which are described in detail within the Ansible documentation, but I am going to show you some basic guidelines on how to get started with Vagrant and Ansible.

### Creating your first playbook:

As stated above, the Ansible playbook (my_playbook.yml) will need to be created. A basic playbook is provided below:

```yaml
- hosts: all
  tasks:   
    - name: set FQDN
      lineinfile:
        path: /etc/hosts
        regexp: '^127.0.0.1.*$'
        line: 127.0.0.1 web.somevm.local
        firstmatch: yes

    - name: Display the config
      debug:
        msg: {% raw %}“The hostname is {{ ansible_net_hostname }}”{% endraw %}
```

This playbook will:
* Set the Fully Qualified Domain Name (FQDN) of the virtual machine, and 
* It will take in `ansible_net_hostname` as a Vagrant argument and output the result.

Now that you have a basic Ansible playbook, you need to ensure that the Vagrant file runs the playbook. You need to add a `config.vm.provision` section to the Vagrant file for the new playbook. An example of a new provisioner is provided below:

```ruby
# Adding Basic Ansible Playbook
    config.vm.provision  "basic", type:'ansible' do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "my_playbook.yml"
        ansible.become = true
	    ansible.extra_vars={
		    ansible_net_hostname: config.vm.hostname
	    }
    end 
```

Note that having the `ansible.verbose` option enabled will instruct Vagrant to show the full `ansible-playbook` command used behind the scenes, as well as any information regarding the processes described within the playbook. This information can be useful when debugging but they can also clutter the screen. Depending on the verbosity you would like on the output, you can change it as required (`“vvv”` will be the most verbose). Once you have added the new provisioner for the Ansible playbook to the Vagrant file, you can run the playbook without needing to recreate the entire VM again. 

* To re-run a playbook on a running VM: `vagrant provision`
* To run the new playbook on a shutdown VM: `vagrant up --provision`

Running the Vagrant file with the new Ansible playbook should result in output similar to that shown below:

```bat
==> default: Running provisioner: basic (ansible)...
    default: Running ansible-playbook...
PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ANSIBLE_HOST_KEY_CHECKING=false ANSIBLE_SSH_ARGS='-o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes -o ControlMaster=auto -o ControlPersist=60s' ansible-playbook --connection=ssh --timeout=30 --limit="default" --inventory-file=/Vagrant/.vagrant/provisioners/ansible/inventory --extra-vars=\{\"ansible_net_hostname\":\"Some_Hostname\"\} --become -v my_playbook.yml
Using /etc/ansible/ansible.cfg as config file

PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host default should use 
/usr/bin/python3, but is using /usr/bin/python for backward compatibility with 
prior Ansible releases. A future Ansible release will default to using the 
discovered platform python for this host. See https://docs.ansible.com/ansible/
2.9/reference_appendices/interpreter_discovery.html for more information. This 
feature will be removed in version 2.12. Deprecation warnings can be disabled 
by setting deprecation_warnings=False in ansible.cfg.
ok: [default]

TASK [set FQDN] ****************************************************************
ok: [default] => {"backup": "", "changed": false, "msg": ""}


TASK [Display the config] ******************************************************
ok: [default] => {
    "msg": "The hostname is Some_Hostname"
}

PLAY RECAP *********************************************************************
default                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

You have just created your first Ansible Playbook, added it to the Vagrant file and created your first automatic deployment of an Ubuntu VM. The final portion that I will describe is cleaning up your test VMs. 

## Cleaning Up

If you want to remove the VM from your host, you can run the following commands:

In order to stop the VM from running: `vagrant halt`
```bat
 vagrant halt
==> default: Attempting graceful shutdown of VM...
```

In order to delete the VM entirely: `vagrant destroy`
```bat
vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
```

I hope all went well and that you successfully managed to create your first VM using Vagrant and Ansible!

## Useful Resources:

* <https://docs.ansible.com/>
* <https://www.vagrantup.com/docs/>