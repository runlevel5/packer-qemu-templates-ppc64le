# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_plugin "vagrant-libvirt"

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

VAGRANTFILE_API_VERSION = "2"

arch = case RUBY_PLATFORM
       when /powerpc64le/
         "ppc64le"
       else
         raise "Your platform is not supported"
       end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "open-power/debian-ppc64le"
  config.vm.box_version = "10.3"
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.vm.provision "shell", path: "bootstrap.sh"
  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Kubernetes Master Server
  config.vm.define "kmaster" do |kmaster|
    kmaster.vm.hostname = "kmaster"
    kmaster.vm.network :private_network,
      ip: "172.42.42.100"
    kmaster.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

    kmaster.vm.provider :libvirt do |v|
      v.qemu_use_session = false
      v.uri = "qemu:///system"
      v.host = "kmaster"
      v.memory = 2048
      v.cpus = 2
      v.video_type = "vga"
    end
    kmaster.vm.provision "shell", path: "bootstrap_kmaster.sh"
  end

  NodeCount = 2

  # Kubernetes Worker Nodes
  (1..NodeCount).each do |i|
    config.vm.define "kworker#{i}" do |workernode|
      workernode.vm.hostname = "kworker#{i}"
      workernode.vm.network :private_network,
        ip: "172.42.42.10#{i}"
      workernode.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
      workernode.vm.provider :libvirt do |v|
        v.qemu_use_session = false
        v.uri = "qemu:///system"
        v.host = "kworker#{i}"
        v.memory = 2048
        v.cpus = 2
        v.video_type = "vga"
      end
      workernode.vm.provision "shell", path: "bootstrap_kworker.sh"
    end
  end
end