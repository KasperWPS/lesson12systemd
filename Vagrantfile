# -*- mode: ruby -*- 
# vi: set ft=ruby : vsa
Vagrant.configure(2) do |config|
        config.vm.box = "centos/8"
        config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end
    config.vm.define "lesssystemd" do |websrv|
        websrv.vm.hostname = "lesssystemd"
        #websrv.vm.provision "shell", path: "scripts/websrv.sh", args: [ip_repo]
    end
end
