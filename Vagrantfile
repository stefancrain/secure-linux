# -*- mode: ruby -*-
# vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"

# Read YAML file with box details
require 'yaml'
hosts = YAML.load_file(File.join(File.dirname(__FILE__), 'hosts.yml'))

# Set cpus to number of host cpus
cpus = case RbConfig::CONFIG['host_os']
  when /darwin/ then `sysctl -n hw.ncpu`.to_i
  when /linux/ then `nproc`.to_i
  else 2
end

Vagrant.configure(2) do |config|
  hosts.each do |host|

    # Create vars from box name
    distro_split = host["box"].split('/')
    distro_group = distro_split[0]
    host_name = "SL-"+distro_group+"-"+distro_split[1]

    # Configure host
    config.vm.define host_name do |node|
      node.vm.box = host["box"]
      node.vm.synced_folder ".", "/vagrant", type: "nfs"
      node.vm.hostname = host_name
      node.vm.network "private_network", ip: host["ip"]
      node.vm.provision "shell", inline: "date -s \"$(curl -I google.com 2>&1 | grep Date: | cut -d' ' -f3-6)Z\""
      node.vm.provider "virtualbox" do |vb|
        vb.customize [
          "modifyvm", :id,
          "--memory", 1024,
          "--cpus", cpus,
          "--name", host_name,
          "--natdnshostresolver1", "on",
          "--ioapic", "on",
          "--audio", "none",
          "--uartmode1", "file", File::NULL,
          "--groups", "/"+distro_group,
        ]
      end
    end
  end
end