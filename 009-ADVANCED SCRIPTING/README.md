
## Advanced Scripting
Running individual containers on multiple machines can be cumbersome. Docker Swarm eliminates the need to manually deploy containers on each node. A single-machine failure can bring down your entire application if your containers are running solely on that machine. Docker Swarm ensures high availability by automatically rescheduling containers to healthy nodes if a node fails. Docker Swarm simplifies container orchestration at scale, ensuring high availability, efficient resource utilization, and streamlined management of your containerized applications.

## Table of Contents
- Vangrantfile
- Docker script
- Master node script
- Worker node script
  
## Vagrant File
First, we must create a Vagrantfile; which is a configuration file used with Vagrant, a tool for managing virtual machines. It acts as a blueprint for setting up your development environment. In this case, we use it to create and set up 3 virtual machines in combination with bash scripts.

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby  :

# Creates an object of 3 virtual machines with memory 1024, unique IP and assigns a docker image

machines = {
  "master" => {"memory" => "1024", "cpu" => "1", "ip" => "100", "image" => "bento/ubuntu-22.04"},
  "node01" => {"memory" => "1024", "cpu" => "1", "ip" => "101", "image" => "bento/ubuntu-22.04"},
  "node02" => {"memory" => "1024", "cpu" => "1", "ip" => "102", "image" => "bento/ubuntu-22.04"}
}
# Sets the Vagrant configuration version to "2" 
Vagrant.configure("2") do |config|

  machines.each do |name, conf|
# This line defines a virtual machine within the Vagrant configuration
    config.vm.define "#{name}" do |machine|
# Sets the box (pre-built VM image) for the virtual machine.
      machine.vm.box = "#{conf["image"]}"
      machine.vm.hostname = "#{name}"
# Configures the network settings for the virtual machine. 
      machine.vm.network "private_network", ip: "10.10.10.#{conf["ip"]}"
      machine.vm.provider "virtualbox" do |vb|
        vb.name = "#{name}"
# Memory allocation
        vb.memory = conf["memory"]
        vb.cpus = conf["cpu"]
        
      end
      machine.vm.provision "shell", path: "docker.sh"
      
      if "#{name}" == "master"
        machine.vm.provision "shell", path: "master.sh"
      else
        machine.vm.provision "shell", path: "worker.sh"
      end

    end
  end
end

```

## Docker Script
This script runs in the Vagrant file, and is a convenient way to set up a system for working with Docker and Docker Compose, particularly for users who work with Vagrant and containerized environments. It streamlines the installation process and ensures the vagrant user has the necessary permissions to utilize Docker effectively.

```ruby

#!/bin/bash
curl -fsSL https://get.docker.com | sudo bash
sudo curl -fsSL "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$
(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo usermod -aG docker vagrant

```
## Master Node Script
This script also runs in the vagrant file. It initializes a Docker Swarm cluster in manager mode and generates a join token for worker nodes. The script stores the join token in a file on the manager node. 

```ruby

#!/bin/bash
sudo docker swarm init --advertise-addr=10.10.10.100
sudo docker swarm join-token worker | grep docker > /vagrant/worker.sh

```

## Worker Script
This line of code instructs a machine to join an existing Docker Swarm cluster as a worker node. this code instructs a worker node to join a Docker Swarm cluster managed by a node at 10.10.10.100:2377 using the provided join token.

```ruby

docker swarm join --token
SWMTKN-1-3pj8k0i4tn77bd93a0yxhgh36hxuef5q5oyg1732rztnfy29ll-a94q0ipwgrjs4xikzyb4yb3n5 10.10.10.100:2377

```

## Conclusion
In conclusion, Docker Swarm tackles the challenges of managing containers across multiple machines. It streamlines deployment, ensures high availability through automatic failover, and optimizes resource utilization. By simplifying container orchestration at scale, Docker Swarm empowers you to effectively manage your containerized applications.
