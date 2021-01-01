# -*- mode: ruby -*-
# vi: set ft=ruby :
# RR

MACHINES = {
  :master => {
        :box_name => "centos/7",
        :net => [
        {ip: '192.168.100.10', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net"}
  ] 
  },
  :slave => {
        :box_name => "centos/7",
        :net => [
        {ip: '192.168.100.11', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net"}
  ]
  },    
  :barman => {
        :box_name => "centos/7",
        :net => [
        {ip: '192.168.100.12', adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "net"}
  ]
  }
}

#################################################################################3
Vagrant.configure(2) do |config|
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
        # нужно если
        case boxname.to_s
        when "master"
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook/play1.yml"
      end
        when "slave"
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook/play2.yml"
      end
        when "barman"
        box.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook/play3.yml"
      end
        end    
    end
  end
end





