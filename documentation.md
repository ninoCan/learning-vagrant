# Vagrant course

We will now turn to the use of Vagrant for creation/automatization/continuous
development of applications in _Vagrant environments_, which are basically 
automated virtual machines. For more information about vagrant
you can check the [info page](https://www.vagrantup.com/intro/index.html).

## Creation

We have already install vagrant, we can check its version with:

    $ vagrant --version
    Vagrant 2.2.5
In the following, we are going to follow the [Getting
Started](https://www.vagrantup.com/intro/getting-started/index.html) and the 
[docs](https://www.vagrantup.com/docs/) on
the official page.
Before creating our first vagrant configuration file, create a dedicated
directory first:
    
    $ mkdir Vagrant-project && cd Vagrant-project
Now there are two way of proceeding: one is by touching a "Vagrantfile" and
editing it manually, or we can just let vagrant do the job for us with the
command "vagrant init"; the second option is ideal for beginners.
Instead of installing a VM from scratch (which should be anyway possible), we
will make use of a vagrant box (the equivalent of a docker image). Available
boxes can be looked up on the vagrant's [box catalogue](https://app.vagrantup.com/boxes). Boxes are sorted according to the \<user\>/\<box-name\>. 
As in the previous weeks, we brought up a LAMP stack on top of ubuntu 18.04,
we will now choose to initialize the official box for ubuntu 18.04:

    $ vagrant init ubuntu/bionic64
I did already touch a Vagrantfile on this folder, but vagrant overwrote this
file with a newbie friendly configuration file with basic commands ready to 
use and explanations of what they do. We can just uncomment them to set our
preferences. But let us remove this file and build it manually instead.
We can add the box without initialization of the vagrant file by using:

    $ vagrant add ubuntu/bionic64
In this way the Vagrantfile should be edited by hand adding:

    Vagrant.configure("2") do |config|
      config.vm.box = "hashicorp/precise64"
    end
The rest of the commands will be put in between the ""Vagrant.configure" line and the end. Additionally, we could have specified the version of the box we want to use:

    config.vm.box = "1.1.0"
or even specify the direct URL of the box with:

    config.vm.box = "https://vagrantcloud.com/ubuntu/bionic64"
With Vagrant we do not necessarily have to run VirtualBox a as a provider,
we can as well set other providers such as VMware, Hyper-V or even docker, with:

    $ vagrant up --provider=<provider-name>
directly at the start, or by adding in the Vagrantfile the following:

    config.vm.provider "<provider-name>"# do |vb|
      #vb.customize ["<custom>","settings"]
    #end
What is before the comment is the basic line neeeded. The commented part are 
extra options that can be added on top of the provider specification.
## Running the machine <!-- and synced folders -->
We are already able to boot the Vagrant environment running:

    $ vagrant up
A file "ubuntu-bionic-18.04-cloudimg-console.log" has been added to the 
current folder. The virtual machine is now up and ready to use, but since
vagrant does not run it with a UI, to access it we need to  use the following 
command:

    $ vagrant ssh
Machine can be explored in this way, and when we are done we can simply logout
with the `$ exit` or `$ logout`bash commands. 

By default, the project directory we create is automatically shared in the 
virtual machine; its mount point is locate in the "/vagrant" folder which 
differs form the home folder "/home/vagrant" in which the user is logged into
after ssh authentication. File created in the synced folders are automatically
created on the host machine, as well.

The virtual machine can be suspended by saving the current running state and 
stop it with the command:
 
    $ vagrant suspend
The advantages are that is a fast way to leave the currently set virtual
machine but the disadvantage of a keeping part of physical disk space and RAM 
of the host busy.
Otherwise, the virtual machine can be halted with:

    $ vagrant halt
this is, to all intents and purposes, equal to powering off a virtual machine
that has been created with Oracle VirtualBox. It is slower then suspending but
only physical memory will be occupied.
Finally, the virtual machine can be completely deleted (if no vagrant process is running) with the command:

    $ vagrant destroy
thus freeing both disk and RAM: the disadvantage is that for complex setting 
it will take some time to fire up again the virtual machine, but preferences
can be saved and trasfered easily from a machine to the other. 
## Automated Provisioning

To automate the installation of packages and make sure that they will be 
installed at upping time we can create a bootstrap bash file in the project
directory. For example we want to set up apache with an eye to have a LAP 
virtual machine. Let us go ahead an touch the following:

    $ nano bootstrap.sh
    #!/usr/bin/env bash
    apt-get update
    apt-get install -y apache2
    if ! [ -L /var/www ]; then
        rm -rf /var/www
        ln -fs /vagrant /var/www
    fi
To tell vagrant to run this script when setting up the machine, add the 
following line in the Vagrantfile:

    $ config.vm.provision :shell, path: "bootstrap.sh"
where the `path` specifies the relative location to the project
root; one can alternatively call a single command with the `, inline: "<cmd>"`
option. This may be very useful when used in combination with an embedded 
script: scripts can be defined in the Vagrantfile in two ways: 

- _**RUBY SYNTAX**_:
  
  We can use the ruby sintax to define a script:
  
      $script= <<-SCRIPT
      echo some text with ruby echo
      date > /etc/vagrant_provisioned_at
      SCRIPT
  where the dollar sign define the ruby variable here and it is not the shell.
  This can be used instead of a file with:
  
      config.vm.provision "shell", inline: $script
- _**generic script**_:
  Let us simply define a multiline string variable, and assign it a shebang, with:
  
        script=<<SCRIPT
        #!/usr/bin/env bash
        echo "some text with bash echo"
        mkdir ./NewFolder
        SCRIPT
  And excecute it with:
  
      config.vm.provision "shell", inline: script
#####   <font color='red'>DISCLAIMER:</font>These methods could actually be the same.
    
In the machine has not been initialized already, the provision will be 
executed automatically at `$ vagrant up`; on the other hand, with an already 
running VM, provisions can be applied, without the initial import step, by using:

    $ vagrant reload --provision

## Connecting the Vagrant environment 
It is possible to connect the vagrant environment in several different ways.
    
### HTTP/SSH/SSL Sharing ACHTUNG: vagrant stopped working after installation
 One way is Vagrant Share: a plugin that allows sharing of the environment 
 with  anyone in the world. Vagrant does not come with the plugin installed by
 default, but this can be installed via 
 
    $ vagrant plugin install vagrant-share
It creates a publicly accessible URL and the accessing party does not need to 
have Vagrant installed in order to view the environment. Being it the standard
sharing method, it can be used by running:
    
    $ vagrant share
 a url will be automatically generated and the accessed through normal 
 browsers. The shared environment can be quit with `Ctrl-C`.
 A public endpoint can be disable with the `--disable-http` flag, and other
 protocols are available:
 - HTTPS/SSL(require a paid version of ngrok) via the 
 443 port in the development environment. 
 - SSH can be easily created by the `--ssh` flag. Anyone can access by running
 `$ vagrant connect --ssh name: **INCOMPLETE**
 
### Networking

Networks are configured withing the Vagrantfile using the following method:

    config.vm.network "<network-identifier>", guest: <port>, host: <port>
Multiple networks can be configured by multiple calls of this method.

The network identifiers can be:
- **forwarded_port**:
  This configuration expects two parameters as above the guest and host ports.
  Additional parameters can be specified for the protocol: 
  `, protoco: "<tcp\udp>"`; port collision auto-correction `,
  auto_correct: true`
-  **private_network**
   It allows to access the guuest machine to be accessed from not publicly 
   accessible address. Multiple machines within the same private network can
   with this method communicate with each other. The easiest configuration is
   the DHCP ip assignemt through the parameter `, type "dhcp"`.
   Otherwise a static ip can be set with the parameter `, ip: "102.168.41.2"`
   Another static address via IPv6 can be configured as well.
- **public_network**
  This works in the same  way, but in addition this configuration can be used 
  to build bridge networks as well.
  
## Multiple machines

Multiple machines can be easily set up in the same Vagrantfile with the call
of `config.vm.define` method.
