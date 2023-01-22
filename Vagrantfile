# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.nfs.functional = false
    config.vm.define "vagrant_hetznerserver" do |host|
        host.vm.box = "generic/debian11"

        host.vm.hostname = "vagrant-hetznerserver"

        host.vm.network :private_network, ip: "192.168.50.11"
        host.vm.provision :hosts, :sync_hosts => true

        config.vm.provider :libvirt do |libvirt|
            libvirt.driver = "kvm"
            libvirt.memory = 8192
            libvirt.cpus = 2
        end
    end

    config.vm.define "vagrant_hetzner_vm" do |host|
        host.vm.box = "opensuse/MicroOS.x86_64"

        host.vm.hostname = "vagrant-hetzner-vm"

        host.vm.network :private_network, ip: "192.168.50.12"
        host.vm.provision :hosts, :sync_hosts => true

        config.vm.provider :libvirt do |libvirt|
            libvirt.driver = "kvm"
            libvirt.memory = 8192
            libvirt.cpus = 2
        end
    end

    config.vm.define "vagrant_local_vm" do |host|
        host.vm.box = "opensuse/MicroOS.x86_64"

        host.vm.hostname = "vagrant-local-vm"

        host.vm.network :private_network, ip: "192.168.60.12"
        host.vm.provision :hosts, :sync_hosts => true

        config.vm.provider :libvirt do |libvirt|
            libvirt.driver = "kvm"
            libvirt.memory = 8192
            libvirt.cpus = 2
        end
    end

end
