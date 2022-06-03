# -*- mode: ruby -*-
# vi: set ft=ruby :
# RR

MACHINES = {
  
  :master => {
        :box_name => "BOX1",
        :net => [
        {ip: '192.168.100.10', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net"}
  ]
  },
  
  :slave => {
        :box_name => "BOX1",
        :net => [
        {ip: '192.168.100.11', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net"}
  ] 
  }
}

#################################################################################3
Vagrant.configure(2) do |config|
  

  config.vm.define "master" do |c|
    c.vm.network "forwarded_port", adapter: 1, guest: 22, host: 2321, id: "ssh", host_ip: '127.0.0.1'
  end

  config.vm.define "slave" do |c|
    c.vm.network "forwarded_port", adapter: 1, guest: 22, host: 2421, id: "ssh", host_ip: '127.0.0.1'
  end
  
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.box_check_update = false
        box.vm.host_name = boxname.to_s
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        box.vm.provider "virtualbox" do |v|
          v.memory = "512"
          v.cpus = "1"
        end
        box.vm.network 'public_network', boxconfig[:public] if boxconfig.key?(:public)
        box.vm.provision "shell", path: "config/sshscript.sh"

        #case boxname.to_s
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook/main.yml"
          ansible.become = "true"
          #ansible.verbose = "v"
        end        
        box.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook/master.yml"
          ansible.become = "true"
          #ansible.verbose = "v"
        end
        box.vm.provision "ansible" do |ansible|
         ansible.playbook = "playbook/slave.yml"
          ansible.become = "true"
          #ansible.verbose = "v"
      end
    end
  end
end





