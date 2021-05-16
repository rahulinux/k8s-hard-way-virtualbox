# -*- mode: ruby -*-
# vi: set ft=ruby :

require "ipaddr"

# No VB Guest available
IMAGE_NAME = "ubuntu/bionic64"
IMAGE_VERSION = "20210514.0.0"
# Make sure LB address is same as your actual physical machine on network
# So that you can access anywhere in your local network
LB = {
    :hostname_prefix => "lb-",
    :public_ip_start => "192.168.2.100",
    :private_ip_start => "10.0.0.100",
    :ram => 256,
    :cpu => 1,
    :count => 1,
}

CONTROLLERS = {
    :hostname_prefix => "controller-",
    :public_ip_start => "192.168.2.111",
    :private_ip_start => "10.0.0.111",
    :ram => 1024,
    :cpu => 4,
    :count => 1,
}

WORKERS = {
    :hostname_prefix => "worker-",
    :private_ip_start => "10.0.0.222",
    :ram => 1024,
    :cpu => 4,
    :count => 1,
}

Vagrant.configure("2") do |config|
    # config.ssh.insert_key = false
    config.vbguest.auto_update = false
    lb_public_ip = IPAddr.new LB[:public_ip_start]
    lb_private_ip = IPAddr.new LB[:private_ip_start]
    1.upto(LB[:count]).each.with_index(1) do |machine, index|
        config.vm.define "#{LB[:hostname_prefix]}#{index}" do |lb_node|
            lb_node.vm.box = IMAGE_NAME
            lb_node.vm.box_version = IMAGE_VERSION
            lb_node.vm.network "private_network", ip: lb_private_ip.to_string()
            lb_node.vm.network "public_network", bridge: ["en0: Wi-Fi (Wireless)"], ip: lb_public_ip.to_string(), auto_config: false
            lb_node.vm.hostname = "#{LB[:hostname_prefix]}#{index}"
            lb_node.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
            lb_node.vm.synced_folder '.', '/vagrant', disabled: true
            lb_node.vm.provision "shell", privileged: true, run: "always",
                inline: <<-SHELL
                ifconfig enp0s9 #{lb_public_ip.to_string()} netmask 255.255.255.0 up
                route -n | grep -q 192.168.2.254 || route add default gw 192.168.2.254
                SHELL
            lb_node.vm.provider "virtualbox" do |vlb|
                vlb.memory = LB[:ram]
                vlb.cpus = LB[:cpu]
                vlb.customize ["modifyvm", :id, "--cableconnected1", "on"]
            end
            lb_public_ip = lb_public_ip.succ
            lb_private_ip = lb_private_ip.succ
        end
    end

    controller_private_ip = IPAddr.new CONTROLLERS[:private_ip_start]
    controller_public_ip = IPAddr.new CONTROLLERS[:public_ip_start]
    1.upto(CONTROLLERS[:count]).each.with_index(1) do |machine, index|
        config.vm.define "#{CONTROLLERS[:hostname_prefix]}#{index}" do |controller_node|
            controller_node.vm.box = IMAGE_NAME
            controller_node.vm.box_version = IMAGE_VERSION
            controller_node.vm.network "private_network", ip: controller_private_ip.to_string()
            controller_node.vm.network "public_network", bridge: ["en0: Wi-Fi (Wireless)"], ip: controller_public_ip.to_string(), auto_config: false
            controller_node.vm.hostname = "#{CONTROLLERS[:hostname_prefix]}#{index}"
            controller_node.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
            controller_node.vm.synced_folder '.', '/vagrant', disabled: true
            controller_node.vm.provision "shell", privileged: true, run: "always",
                inline: <<-SHELL
                ifconfig enp0s9 #{controller_public_ip.to_string()} netmask 255.255.255.0 up
                route -n | grep -q 192.168.2.254 || route add default gw 192.168.2.254
                SHELL
            controller_node.vm.provider "virtualbox" do |vcontroller|
                vcontroller.memory = CONTROLLERS[:ram]
                vcontroller.cpus = CONTROLLERS[:cpu]
                vcontroller.customize ["modifyvm", :id, "--cableconnected1", "on"]
            end
            controller_private_ip = controller_private_ip.succ
            controller_public_ip = controller_public_ip.succ
        end

    end

    worker_private_ip = IPAddr.new WORKERS[:private_ip_start]
    1.upto(WORKERS[:count]).each.with_index(1) do |machine, index|
        config.vm.define "#{WORKERS[:hostname_prefix]}#{index}" do |worker_node|
            worker_node.vm.box = IMAGE_NAME
            worker_node.vm.box_version = IMAGE_VERSION
            worker_node.vm.network "private_network", ip: worker_private_ip.to_string()
            worker_node.vm.hostname = "#{WORKERS[:hostname_prefix]}#{index}"
            worker_node.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
            worker_node.vm.synced_folder '.', '/vagrant', disabled: true
            worker_node.vm.provider "virtualbox" do |vcontroller|
                vcontroller.memory = WORKERS[:ram]
                vcontroller.cpus = WORKERS[:cpu]
                vcontroller.customize ["modifyvm", :id, "--cableconnected1", "on"]
            end
            worker_private_ip = worker_private_ip.succ
        end
    end
end
