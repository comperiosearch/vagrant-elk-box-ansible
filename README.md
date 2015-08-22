This vagrant box installs elasticsearch 1.7, logstash 1.5.3 and kibana 4.1.1
This is meant to replace the old [Vagrant ELK box](https://github.com/comperiosearch/vagrant-elk-box),  where provisioning by puppet has been replaced by ansible.

## Prerequisites

[VirtualBox](https://www.virtualbox.org/) and [Vagrant](http://www.vagrantup.com/) (minimum version 1.6)
Other providers, like VMWare may work, not tested!

## Checkout the project
This repo uses git submodules.
To clone the repo, use the --recurse-submodules option.  The submodules contain role definitions and nothing will work without that, unfortunately.  

    git clone --recurse-submodules  https://github.com/comperiosearch/vagrant-elk-box-ansible.git

If you need to pull in latest changes, please uses

     git pull --recurse-submodules
     git submodule update --init --recursive

## Up and SSH

To start the vagrant box run:

    vagrant up

To log in to the machine run:

    vagrant ssh

Elasticsearch will be available on the host machine at [http://localhost:9200/](http://localhost:9200/) 

Kibana at [http://localhost:5601/](http://localhost:5601/)

Marvel elasticsearch plugin at [http://localhost:9200/_plugin/marvel/](http://localhost:9200/_plugin/marvel/)

HQ elasticsearch plugin at [http://localhost:9200/_plugin/HQ/](http://localhost:9200/_plugin/HQ/)


## Vagrant commands


```
vagrant up # starts the machine
vagrant ssh # ssh to the machine
vagrant halt # shut down the machine
vagrant provision # applies the bash and ansible provisioning

```

##Elasticsearch
Installed via debian package, started on boot.
Controlled by

```bash

 sudo service elasticsearch

```


##Logstash
Installed via debian package, started on boot.
Controlled by

```bash

 sudo service logstash

```

Some sample Logstash data is installed on provisioning. Reading in log lines from include/example-logs/testlog

##Kibana 
Controlled by

```bash

sudo service kibana

```

## Ansible provisioning
Ansible is installed on the guest machine by the setup.sh bash script which is run as part of vagrant provisioning. Vagrant does actually have a "built-in" provisioner for ansible, but that is supposed to run on the host machine, and ansible does not run on windows making that option unavailable for a large amount of users. Myself being one of the unfortunate we roll our own setup installing ansible on the guest machine.  The last step in the provisioning script setup.sh is to run the playbook located at provisioning/playbook.yml. 

The code for the Ansible init script was heavily inspired by this blog http://akrabat.com/provisioning-with-ansible-within-the-vagrant-guest/
