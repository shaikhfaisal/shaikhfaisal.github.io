---
title: Using packer to build vagrant base boxes
layout: default
excerpt: <p>I recently needed to build a Vagrant base box and but did not want to use the boxes that were pre-built and available on the internet. 
         I wanted to build my own boxes from the original distribution ISOs. This is possible to do manually but is a laborious and slow process. 
         Enter [Packer](packer.io) -  a tool from the developers of Vagrant that automates this so you can go from original distribution ISO to
         vagrant box by typing in one command. This lends itself to automation which is almost always a good thing.</p>
---

### {{page.title}}
I recently needed to build a Vagrant base box and but did not want to use the boxes that were pre-built and available on the internet. 
I wanted to build my own boxes from the original distribution ISOs. This is possible to do manually but is a laborious and slow process. 
Enter [Packer](packer.io) -  a tool from the developers of Vagrant that automates this so you can go from original distribution ISO to
vagrant box by typing in one command. This lends itself to automation which is almost always a good thing.


This is my vanilla packer json file. It basically configures packer to do the following:

1. Request a compute resource from VirtualBox
2. Download and verify the distribution iso
3. Handover to preseed to automate the installation of the distribution
4. After installation, do some basic shell provisioning and then package the compute resource as a Vagrant box

**template.json**
{% highlight javascript %}
{%raw%}
{
  "description" : "Ubuntu LTS Trusty Tahr",

  "variables" : {
    "vm_name": "ubuntu-14.04-LTS.vagrant.box",
    "vm_domain": "example.com"
  },
  
  "builders":  [
    {
      "type": "virtualbox-iso",
      "guest_os_type": "Ubuntu_64",
  
      "iso_url": "http://releases.ubuntu.com/trusty/ubuntu-14.04-server-amd64.iso",
      "iso_checksum": "01545fa976c8367b4f0d59169ac4866c",
      "iso_checksum_type": "md5",
  
      "shutdown_command": "echo 'halt -p' > shutdown.sh; echo 'vagrant'|sudo -S sh 'shutdown.sh'",
  
      "disk_size": 20000,
  
      "format": "ovf",
      "output_directory": "vms/{{user `vm_name`}}", 
 
      "headless": false,
  
      "http_directory": "httpdir",
      "http_port_min": 10082,
      "http_port_max": 10089,
  
      "ssh_host_port_min": 2222,
      "ssh_host_port_max": 2229,
      "ssh_username": "vagrant",
      "ssh_password": "vagrant",
      "ssh_port": 22,
      "ssh_wait_timeout": "90m",
  
      "vm_name": "{{user `vm_name`}}",

      "boot_wait": "6s",

      "boot_command":  [

        "<esc>",
        "<esc>",
        "<enter><wait>",
        "/install/vmlinuz",
        " auto",
        " console-setup/ask_detect=false",
        " console-setup/layoutcode=uk",
        " console-setup/modelcode=pc105",
        " debconf/frontend=noninteractive",
        " debian-installer=en_GB",
        " fb=false",
        " initrd=/install/initrd.gz",
        " kbd-chooser/method=us",
        " keyboard-configuration/layout=UK",
        " keyboard-configuration/variant=UK",
        " locale=en_US",
        " netcfg/get_domain={{user `vm_domain`}}",
        " netcfg/get_hostname=vagrant",
        " noapic",
        " preseed/url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/ubuntu-14.04/preseed.cfg",
        " -- ",
        "<enter>"
      ],
      "vboxmanage": [
        ["modifyvm", "{{.Name}}", "--memory", "1500"]
      ]
    }
  ],

  "provisioners": [
    {
       "type": "shell",
       "script": "provisioners/shell/trusty_vagrant_setup_script.sh"
     }
  ],

  "post-processors": ["vagrant"]


}
{%endraw%}
{% endhighlight %}

This is my associated preseed file:

**httpdir/preseed.cfg:**
{% highlight javascript %}
{%raw%}
d-i debian-installer/language string en
d-i debian-installer/country string GB
d-i debian-installer/locale string en_GB.UTF-8
d-i localechooser/preferred-locale string en_GB.UTF-8
d-i localechooser/supported-locales en_GB.UTF-8

# Including keyboards
d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/variant select uk
d-i keyboard-configuration/modelcode string pc105

# Just roll with it
d-i netcfg/get_hostname string this-host
d-i netcfg/get_domain string this-host

d-i time/zone string UTC
d-i clock-setup/utc-auto boolean true
d-i clock-setup/utc boolean true

# Choices: Dialog, Readline, Gnome, Kde, Editor, Noninteractive
d-i debconf debconf/frontend select Noninteractive

d-i pkgsel/install-language-support boolean false
tasksel tasksel/first multiselect standard, ubuntu-server

d-i netcfg/choose_interface select auto

# Stuck between a rock and a HDD place
d-i partman-auto/method string lvm
d-i partman-lvm/confirm boolean true
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-auto/choose_recipe select atomic

### Partitioning
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman/choose_partition select finish
d-i partman-auto-lvm/guided_size string max
d-i partman-auto/choose_recipe select atomic
d-i partman/default_filesystem string ext4
d-i partman/confirm_write_new_label boolean true
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# No proxy, plx
d-i mirror/http/proxy string

# Root password, either in clear text
d-i passwd/root-password password vagrant
d-i passwd/root-password-again password vagrant

# Default user, change
d-i passwd/user-fullname string vagrant
d-i passwd/username string vagrant
d-i passwd/user-password password vagrant
d-i passwd/user-password-again password vagrant
d-i user-setup/encrypt-home boolean false
d-i user-setup/allow-password-weak boolean true

# No language support packages.
d-i pkgsel/install-language-support boolean false

# Individual additional packages to install
d-i pkgsel/include string build-essential ssh git vim ansible python-paramiko

#For the update
d-i pkgsel/update-policy select none

# Whether to upgrade packages after debootstrap.
# Allowed values: none, safe-upgrade, full-upgrade
d-i pkgsel/upgrade select safe-upgrade

# Give vagrant super powers - this should get removed by the provisioning script
#d-i preseed/late_command string in-target echo 'Defaults:%sudo !requiretty' >> /target/etc/sudoers.d/sudo ; \
#                                in-target echo 'vagrant ALL=(ALL) NOPASSWD: ALL' >> /target/etc/sudoers.d/sudo


# Go grub, go!
d-i grub-installer/only_debian boolean true

d-i finish-install/reboot_in_progress note
{%endraw%}
{% endhighlight %}

I like how packer uses established tools like preseed and kickstarter and doesn't try to re-invent the wheel.

To make these examples work, your file layout needs to be as below:

{% highlight javascript %}
template.json
|--httpdir
   |--preseed.cfg  
{% endhighlight %}

Download packer from [packer.io](packer.io) and then run the following command from your shell

{% highlight bash %}
./packer build test.json
{% endhighlight %}
