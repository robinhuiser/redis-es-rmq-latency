# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"
N = 3

# Build the traffic shaping device
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.define "tsd" do |tsd|
        tsd.vm.box = "centos/7"
        tsd.vm.hostname = "tsd"
        (1..N).each do |server_id|
            tsd.vm.network "private_network", ip: "192.168.3#{server_id}.254"
        end
        tsd.vm.provider "virtualbox" do |vb|
            vb.name = "tsd"
            vb.cpus = "1"
            vb.memory = "128"
        end

        # Run the generic playbook
        tsd.vm.provision :ansible do |ansible|
            ansible.limit = "tsd"
            ansible.playbook = "all-servers.yml"
            ansible.compatibility_mode = "2.0"
            ansible.become = true
        end

        # Run the traffic shaper playbook
        tsd.vm.provision :ansible do |ansible|
            ansible.limit = "tsd"
            ansible.playbook = "tsd.yml"
            ansible.compatibility_mode = "2.0"
            ansible.become = true
        end
    end
end

# Build the servers applicable to traffic shaping
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  (1..N).each do |server_id|
    config.vm.define "server#{server_id}" do |server|
      server.vm.box = "centos/7"
      server.vm.hostname = "server#{server_id}"
      server.vm.network "private_network", ip: "192.168.3#{server_id}.10"
      server.vm.provider "virtualbox" do |vb|
        vb.name = "server#{server_id}"
        vb.cpus = "2"
        vb.memory = "512"
      end

      if server_id == N
        # Run the generic playbook for all servers
        server.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "all-servers.yml"
          ansible.compatibility_mode = "2.0"
          ansible.become = true
        end

        # Run the server specific playbook
        (1..N).each do |my_id|
          server.vm.provision :ansible do |ansible|
            ansible.limit = "server#{my_id}"
            ansible.playbook = "server#{my_id}.yml"
            ansible.compatibility_mode = "2.0"
            ansible.become = true
          end
        end
      end
    end
  end
end
