# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

NOSDS = 3

ansible_provision = Proc.new do |ansible|
  ansible.playbook = "site.yml"
  # Note: Can't do ranges like mon[0-2] in groups because
  # these aren't supported by Vagrant, see
  # https://github.com/mitchellh/vagrant/issues/3539
  ansible.groups = {
    "mons" => ["swiftcephproxy"],
    "proxies" => ["swiftcephproxy"],
    "osds" => (0..NOSDS-1).map {|j| "swiftcephstorage#{j}"},
    "storages" => (0..NOSDS-1).map {|j| "swiftcephstorage#{j}"}
  }

  # In a production deployment, these should be secret
  ansible.extra_vars = {
   fsid: "4a158d27-f750-41d5-9e7f-26ce4c9d2d45",
   monitor_secret: "AQAWqilTCDh7CBAAawXt6kyTgLFCxSvJhTEmuw=="
  }
  ansible.limit = 'all'
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.define :swiftcephproxy do |swiftcephproxy|
    swiftcephproxy.vm.network :private_network, ip: "192.168.10.199"
    swiftcephproxy.vm.host_name = "swiftcephproxy"
    swiftcephproxy.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "384"]
    end
  end

  (0..NOSDS-1).each do |i|
    config.vm.define "swiftcephstorage#{i}" do |swiftcephstorage|
      swiftcephstorage.vm.hostname = "swiftcephstorage#{i}"
      swiftcephstorage.vm.network :private_network, ip: "192.168.10.10#{i}"
      swiftcephstorage.vm.network :private_network, ip: "192.168.11.10#{i}"
      (0..3).each do |d|
        swiftcephstorage.vm.provider :virtualbox do |vb|
          vb.customize [ "createhd", "--filename", "disk-#{i}-#{d}", "--size", "1000" ]
          vb.customize [ "storageattach", :id, "--storagectl", "SATAController", "--port", 3+d, "--device", 0, "--type", "hdd", "--medium", "disk-#{i}-#{d}.vdi" ]
          vb.customize ["modifyvm", :id, "--memory", "512"]
        end
      end

      # Run the provisioner after the last machine comes up
      if i == (NOSDS-1)
        swiftcephstorage.vm.provision "ansible", &ansible_provision
      end

    end
  end
end
