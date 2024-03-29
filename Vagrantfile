# -*- mode: ruby -*-
# vi: set ft=ruby :
#require_relative 'vagrant_rancheros_guest_plugin.rb'
require 'ipaddr'
require 'yaml'

x = YAML.load_file('config.yaml')
puts "Config: #{x.inspect}\n\n"

$private_nic_type = x.fetch('net').fetch('private_nic_type')

Vagrant.configure(2) do |config|

    config.vm.define "server-01" do |server|
     c = x.fetch('server')
      server.vm.box= "generic/ubuntu1804"
      server.vm.provider :virtualbox do |v|
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.cpus = c.fetch('cpus')
        v.linked_clone = true if Gem::Version.new(Vagrant::VERSION) >= Gem::Version.new('1.8.0') and x.fetch('linked_clones')
        v.memory = c.fetch('memory')
        v.name = "server-01"
      end
      server.vm.network x.fetch('net').fetch('network_type'), ip: x.fetch('ip').fetch('server') , nic_type: $private_nic_type
      server.vm.hostname = "server-01"
      server.vm.provision "shell", path: "scripts/configure_rancher_server.sh", args: [x.fetch('admin_password'), x.fetch('version'), x.fetch('k8s_version')]
    
  end

  node_ip = IPAddr.new(x.fetch('ip').fetch('node'))
  (1..x.fetch('node').fetch('count')).each do |i|
    c = x.fetch('node')
    hostname = "node-%02d" % i
    config.vm.define hostname do |node|
      node.vm.box   = "generic/ubuntu1804"
      node.vm.provider "virtualbox" do |v|
        v.cpus = c.fetch('cpus')
        v.memory = c.fetch('memory')
        v.name = hostname
      end
      node.vm.network x.fetch('net').fetch('network_type'), ip: IPAddr.new(node_ip.to_i + i - 1, Socket::AF_INET).to_s, nic_type: $private_nic_type
      node.vm.hostname = hostname
      node.vm.provision "shell", path: "scripts/configure_rancher_node.sh", args: [x.fetch('ip').fetch('server'), x.fetch('admin_password')]
    end
  end

end
