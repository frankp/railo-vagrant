railo-vagrant
=============

Simple setup to launch a VM using Vagrant running Ubuntu 12.04 and Railo


Prerequisites:
 
* VirtualBox
* Vagrant

Vagrant allows you to launch local VMs via the command line that can easily be destroyed and recreated. It's power is in the ability to script out all of the tasks you'd need when launching a VM using Chef or Puppet.

This project uses Vagrant + Chef to create a VM with the following:

* Ubuntu 12.04 LTS
* Tomcat
* Apache
* MySQL
* Railo (currently 3.3)
* shared folder inside of your VM for developing your code

This uses a very basic Chef recipe (Chef recipes are written in Ruby). Some caveats

* Most Chef best practices indicate using separate recipes for different steps - I've put it all in one
* I rely on Ubuntu's package manager where possible. (Most Chef recipes for Tomcat, for example, download it and extract directly from Apache, not relying on apt-get, yum, etc)
* It's a JAR-based deployment for Railo
* The Linux file permissions could possibly be tweaked 
* Designed for Ubuntu (though _should_ work for other distros)

Configure Host
--------------
* Doesn't configure your "hosts" file on host machine, since this could theoretically be run on Windows, Mac, or Linux. For the default settings, you'd change your hosts file (/etc/hosts or c:\windows\system32\drivers\etc\hosts):

```
192.168.33.10	testrailo.dev
```

Running it
----------
 - downloads the Ubuntu instance (only the first time)
 - installs Tomcat, Apache, MySQL, VIM and GIT
 - downloads and extracts Railo
 - creates up a MySQL database called oracle_rules
 - configures Tomcat, Apache and Railo with defaults, and gives you a bare-bones index.cfm.

```
$ git clone git@github.com:bdcravens/railo-vagrant.git
$ cd railo-vagrant
$ git submodule init
$ git submodule update
$ vagrant up 
```
After it starts, verify it runs by opening ```http://testrailo.dev```. You should see a cfdump of server scope. (Note: you'll need to set up the hosts entry as above first, or whatever you've changed the values to in Vagrantfile and/or your attributes file)

Railo Version
-------------
Originally written to load Railo 3.3.x. If you want to run Railo 4.0, rename ```cookbooks/main/recipes/default.rb``` to ```default.rb.old``` and rename ```railo4.rb``` to ```default.rb```. (Modularizing this and making it easy to select version is a TODO)

Making Changes
--------------
See ```cookbooks/attributes/default.rb``` (Yes, it's Ruby. You'll be fine.) If you want to perform more Tomcat or Railo config changes, see the respective files (server.xml.rb, web.xml.rb, or _admin.cfm.rb) in cookbooks/templates/default

You will want to empty out the code directory (including ```WEB-INF```) if you make any changes to the ```index.cfm``` or ```admin.cfm``` templates. To restart without redownloading things like Railo and Tomcat:

    $ vagrant reload

To start over from scratch:

    $ vagrant destroy
    $ vagrant up

Railo Administration location
-----------------------------
Usual location
 - http://testrailo.dev/railo-context/admin/server.cfm
 - http://testrailo.dev/railo-context/admin/web.cfm

Passwords
-----------
 - Railo passwords in cookbooks/attributes/default.rb (defaults to ```railoserver``` and ```railoweb```)
 - MySQL password in node.json (defaults to ```railodb``` for ```root```)

Hey Billy, you're an idiot because you ….
-----------------------------------------
I'm an expert at a few things, and a hack at most. Feel free to make changes and issue a pull request.

Where's the Adobe and OpenBD love? Why jars and not wars? etc
-------------------------------------------------------------
Could easily do this with all the above (though exact steps of setting password, making admin changes, etc, would vary) - the concepts are the same. Use this project as your starting point.


Error Resolution
-------------------------------------------------------------
### error
    The host class is reporting that NFS is not supported by this host,
    or `nfsd` may not be installed. Please verify that `nfsd` is installed
    on your machine, and retry.
### resolution
#### linux
    $ sudo apt-get install nfs-kernel-server

#### windows
Change the vagrantfile so you use virtualbox's shared folders instead of nfs, by setting nfs=>false:
config.vm.share_folder "blah", "~/blah", ".", :extra => 'dmode=755,fmode=644', :nfs => false
    
http://devblog.alexsapps.com/2012/11/solution-to-vagrant-up-host-class-is.html

Special thanks to ….
--------------------
* Mike Henke - For most of the changes after the initial proof-of-concept 
* Nathan Mische - [he did the same thing for Railo Express](https://github.com/nmische/cookbooks)
* Lew Goettner [He has a couple of CF related projects](https://github.com/lewg) Also, check out [his blog post](http://beacon.wharton.upenn.edu/404/2011/12/keeping-your-machine-clean-with-vagrant-chef)
* Various contributors to my questions related to this on the Railo mailing list: Peter Boughton, Chris Blackwell, Denny


