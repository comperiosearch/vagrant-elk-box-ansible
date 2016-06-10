This vagrant box installs elasticsearch 2.1, logstash 2.1 and kibana 4.3
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

Sense, the wonderful elasticsearch query machine is found at [http://localhost:5601/app/sense](http://localhost:5601/app/sense)


### Elasticsearch
Installed via debian package, started on boot.
Controlled by

```bash

 sudo service elasticsearch

```


### Logstash
Installed via debian package, started on boot.
Controlled by

```bash

 sudo service logstash

```

Some sample Logstash data is installed on provisioning. Reading in log lines from include/example-logs/testlog

### Kibana 
Controlled by

```bash

sudo service kibana

```

### For testing purposes, You can run analysis againts local log files or datasets on the vagrant box. 

You can move the log files by following one of the two methods below: 
1- There is already a mounted folder called /vagrant from the vagrant host to the vagrant box, You can use it to copy the log file to in the vagrant box. 

The vagrant mounted folder's name is ( /vagrant ) on the box and you can find it on the vagrant host in the same loction you have cloned the git repo which contains the vagrantfile for the project.  

2- Use vagrant scp plugin 
Installing scp plugin:
vagrant plugin install vagrant-scp

How to use it: 
vagrant scp <some_local_file_or_dir> [vm_name]:<somewhere_on_the_vm>

### Creating logstash.conf 
In order to start analyzing the local logs, You should create a logstash configuration folder and you can locate it in the follwoing directory : 
./etc/logstash/conf.d/

logstach.conf example to run against multipe files in a local folder : 

input {
  file {
    path => "/vagrant/local_logs/*.log"
    # The path for the local log files that you have created and copied the log files to 
    start_position => "beginning"
    # You set start_position to "beginning" for exisitng log file, You can set it to "end" for varying log files. 
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { "type" => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    # localhost is the ELK provisioned box and 9200 is the port for elastic search
  }
  stdout { codec => rubydebug }
}

# You will need to restart the logstash service for the change to be effective

sudo service logstash restart 

## Ansible provisioning
Ansible is installed on the guest machine by the setup.sh bash script which is run as part of vagrant provisioning. Vagrant does actually have a "built-in" provisioner for ansible, but it runs on the host machine, making that option unavailable on windows. Myself being one of the unfortunate we roll our own setup installing ansible on the guest machine.  The last step in the provisioning script is running the playbook located at provisioning/playbook.yml. 

The code for the Ansible init script was heavily inspired by this blog http://akrabat.com/provisioning-with-ansible-within-the-vagrant-guest/
