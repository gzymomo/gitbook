```json
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
 
  config.vm.define :docker do |docker|
    docker.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--name", "docker", "--memory", "1024","--cpus","2"]
    end
    docker.vm.box = "centos_7"
    docker.vm.hostname = "docker"
    #将虚拟机放到跟宿主机一样的局域网
    docker.vm.network :public_network, ip: "192.168.10.184"
  end

end
```

