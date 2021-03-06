# About  
This is a bleeding edge playground repository to get more familiar with Puppet 4, future parser, cfacter, PuppetDB 3, MCollective and puppetserver 2.  
It is intended to be a quick way to spawn up a fully working Puppet 4 environment.  
It will automatically install the latest version of the full above stack.  
If it breaks because of a new version, let me know and I'll try to fix it.  
It was last suceessfully tested with:  
puppet-agent-1.5.2  
puppetserver-2.4.0  
puppetdb-4.1.2  

In the Vagrantfile there are 2 VMs defined.  
A puppetserver ("puppet") and a puppet node ("node1") both running CentOS 7.0.  
Classes get configured via hiera (see `code/environments/production/hieradata/*`).  

# Requirements
I'm using Vagrant 1.7.3 and VirtualBox as the provisioner.  
I'm running this on Mac OS X but it should also run on other operating systems, but I haven't tested it.  
Older versions of Vagrant are not supported, as Puppet 4 support was only [added in 1.7.3](https://github.com/mitchellh/vagrant/issues/3740) 

Also tested and working on: SLES 11.3, VirtualBox-4.3-4.3.28_100309_sles11.0-1.  

The puppetserver VM is configured to use 3GB of RAM.  
The node is using the default (usually 512MB).  

# Usage
After cloning the repository make sure the submodules are also updated:  
```
$ git clone https://github.com/roman-mueller/puppet4-sandbox
$ cd puppet4-sandbox
$ git submodule update --init --recursive
```

Whenever you `git pull` this repository you should also update the submodules again.  

Now you can simply run `vagrant up puppet` to get a fully set up puppetserver.  
The `code/` folder will be a synced folder and gets mounted to `/etc/puppetlabs/code` inside the VM.  

If you want to attach a node to the puppetserver simply run `vagrant up node1`.  
Once provisioned it is automatically connecting to the puppetserver and it gets automatically signed.  

After that puppet will run automatically every 30 minutes on the node and apply your changes.  
You can also run it manually:  
```
$ vagrant ssh node1
[vagrant@node1 ~]$ sudo /opt/puppetlabs/bin/puppet agent -t
Info: Caching certificate for node1
Info: Caching certificate_revocation_list for ca
Info: Caching certificate for node1
Info: Retrieving pluginfacts
Info: Retrieving plugin
(...)
Notice: Applied catalog in 0.52 seconds
```

PuppetDB gets installed and started by default.  
The local port 8080 gets forwarded to the Vagrant VM to port 8080.  
So you can access the PuppetDB dashboard via http://127.0.0.1:8080/dashboard/index.html.  
All necessary keys get created when it starts for the first time.  
The puppetserver is configured to store reports in the DB, so you can start playing with that too right away.  
The [PuppetDB CLI tools](https://docs.puppet.com/puppetdb/master/pdb_client_tools.html) are installed on the "puppet" node and should work right away:
```
[vagrant@puppet ~]$ /opt/puppetlabs/bin/puppet query 'nodes [ certname ]{ limit 10 }'
[ {
  "certname" : "puppet"
}, {
  "certname" : "node1"
} ]
``` 

MCollective gets installed and configured as well.  
It should also work out of the box, and node1 will register after the initial Puppet run:  
```
[root@puppet vagrant]# /opt/puppetlabs/bin/mco ping
node1                                    time=29.86 ms
puppet                                   time=67.98 ms


---- ping statistics ----
2 replies max: 67.98 min: 29.86 avg: 48.92 
```

# Security
This repository is meant as a non-production playground setup.  
It is not a guide on how to setup a secure Puppet environment.  

In particular this means:  

- Auto signing is enabled, every node that connects to the puppetserver is automatically signed.  
- MCollective uses a PSK and no SSL.  
- The MCollective client gets deployed on every agent that connects to the puppetserver.
- Passwords or PSKs are not randomized and easily guessable. 

For a non publicly reachable playground this should be acceptable.  


# Hacks
I wrote a shell provisioner ("puppetupgrade.sh") which updates `puppet-agent` before running it for the first time.  
That way newly spawned Vagrant environments will always use the latest available version.  

There is no DNS server running in the private network.  
All nodes have each other in their `/etc/hosts` files.  

Starting the puppetserver sometimes hits the systemd timeout (https://tickets.puppetlabs.com/browse/SERVER-557).  
To work around this, the file `/etc/systemd/system/puppetserver.service.d/local.conf` gets created which overrides the timeout and sets it to 500 seconds.

# Tickets
Working on the bleeding edge usually produces some problems and tickets, here are the ones opened while working on this repo:  
- https://tickets.puppetlabs.com/browse/PUP-4461 manifest changes are ignored when using hiera_include: fixed  
- https://tickets.puppetlabs.com/browse/CPR-186 PuppetDB 2.3.4 not in PC1 yum repositories: fixed  
- https://tickets.puppetlabs.com/browse/CPR-184 Puppet 4 repository "repodata" folder missing: fixed  
- https://tickets.puppetlabs.com/browse/CPR-183 puppetlabs-release-pc1 RPM missing: fixed
- https://tickets.puppetlabs.com/browse/SERVER-557 systemd timeout is reached: still open  
- https://tickets.puppetlabs.com/browse/FACT-1117 cfacter not working on CentOS7: still open  
- https://tickets.puppetlabs.com/browse/MODULES-2487 improve ::firewall port deprecation warning  
- https://tickets.puppetlabs.com/browse/MODULES-2488 use dport instead of deprecated port  

