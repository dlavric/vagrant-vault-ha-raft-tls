# -*- mode: ruby -*-
# vi: set ft=ruby :

# download Vault by specifying the version
# VAULT=""

Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"
    config.vm.provision "shell", inline: "cp /vagrant/hosts /etc/hosts"
    config.vm.provider "virtualbox"

    (1..3).each do |i|
      config.vm.define "vault-#{i}" do |vault|
        vault.vm.hostname = "vault-#{i}"
        vault.vm.network "private_network", ip: "192.168.56.#{150+i}"
        vault.vm.provision "shell",
          path: "scripts/vault_nodes/vault_node_#{i}.sh"
      end
    end

end
