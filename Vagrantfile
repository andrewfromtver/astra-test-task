# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_BOX_1 = './fedora33'
VAGRANT_BOX_2 = './oracle_linux8'
VAGRANT_BOX_3 = './ubuntu_bionic'

Vagrant.configure(2) do |config|

  config.vm.define "vm3" do |vm3|
    vm3.vm.box = VAGRANT_BOX_3
    vm3.vm.hostname = 'vm3.test.local'
    vm3.vm.provider "virtualbox" do |v|
      v.name = 'vm3'
      v.memory = 4096
      v.cpus = 2
    end
    vm3.vm.network "private_network", ip: "192.168.10.12"
    vm3.vm.network "forwarded_port", guest: 80, host: 8082
    vm3.vm.provision "shell", inline: <<-SHELL
      sed -i 's/127.0.2.1 vm3.test.local vm3/192.168.10.12 vm3.test.local vm3/' /etc/hosts

      apt-get remove docker docker-engine docker.io containerd runc
      apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
      mkdir -p /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      apt-get update
      apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

      wget https://github.com/docker/compose/releases/download/v2.7.0/docker-compose-linux-x86_64
      chmod +x docker-compose-linux-x86_64
      mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose

      apt install firewalld
      firewall-cmd --add-service=http --zone=public --permanent
      firewall-cmd --reload

      cd /opt
      git clone https://github.com/zabbix/zabbix-docker.git
      docker-compose -f /opt/zabbix-docker/docker-compose_v3_alpine_pgsql_latest.yaml up -d

    SHELL
    vm3.vm.synced_folder "shared/", "/shared"
  end

  config.vm.define "vm2" do |vm2|
    vm2.vm.box = VAGRANT_BOX_2
    vm2.vm.hostname = 'vm2.test.local'
    vm2.vm.provider "virtualbox" do |v|
      v.name = 'vm2'
      v.memory = 4096
      v.cpus = 2
    end
    vm2.vm.network "private_network", ip: "192.168.10.11"
    vm2.vm.network "forwarded_port", guest: 443, host: 8081
    vm2.vm.provision "shell", inline: <<-SHELL
      sed -i 's/127.0.1.1 vm2.test.local vm2/192.168.10.11 vm2.test.local vm2/' /etc/hosts
      sed -i '/^192.168.10.11 vm2.test.local vm2/a 192.168.10.10 vm1.test.local vm1' /etc/hosts
      
      dnf -y install https://yum.puppet.com/puppet7-release-el-8.noarch.rpm
      dnf -y install https://yum.theforeman.org/releases/3.3/el8/x86_64/foreman-release.rpm
      dnf -y module enable foreman:el8
      dnf -y install foreman-installer
      
      firewall-cmd --zone=public --permanent --add-service=https
      firewall-cmd --reload
      
      foreman-installer
      SHELL
    vm2.vm.synced_folder "shared/", "/shared"
  end

  config.vm.define "vm1" do |vm1|
    vm1.vm.box = VAGRANT_BOX_1
    vm1.vm.hostname = 'vm1.test.local'
    vm1.vm.provider "virtualbox" do |v|
      v.name = 'vm1'
      v.memory = 4096
      v.cpus = 2
    end
    vm1.vm.network "private_network", ip: "192.168.10.10"
    vm1.vm.network "forwarded_port", guest: 80, host: 8080
    vm1.vm.network "forwarded_port", guest: 443, host: 443
    vm1.vm.provision "shell", inline: <<-SHELL
      sed -i 's/127.0.1.1 vm1.test.local vm1/192.168.10.10 vm1.test.local vm1/' /etc/hosts
      sed -i '/^192.168.10.10 vm1.test.local vm1/a 192.168.10.11 vm2.test.local vm2' /etc/hosts
      sed -i 's/net.ipv6.conf.all.disable_ipv6 = 1/net.ipv6.conf.all.disable_ipv6 = 0/' /etc/sysctl.conf
      
      sysctl -p
      
      dnf install -y ansible
      
      git clone https://github.com/freeipa/ansible-freeipa.git
      cp /shared/test /home/vagrant/ansible-freeipa/
      cp /shared/install-cluster.yml /home/vagrant/ansible-freeipa/
      cd /home/vagrant/ansible-freeipa
      export ANSIBLE_HOST_KEY_CHECKING=False
      ansible-playbook -v -i test install-cluster.yml
    SHELL
    vm1.vm.synced_folder "shared/", "/shared"
  end

end
