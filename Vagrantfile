NUM_WORKER_NODES=2
IP_NW="10.0.0."
IP_START=10

Vagrant.configure("2") do |config|
  config.vm.provision "shell", env: {"IP_NW" => IP_NW, "IP_START" => IP_START}, inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START)) master-node" >> /etc/hosts
      echo "$IP_NW$((IP_START+1)) worker-node01" >> /etc/hosts
      echo "$IP_NW$((IP_START+2)) worker-node02" >> /etc/hosts
  SHELL
  config.ssh.username = "vagrant"
	config.ssh.private_key_path = ["#{ENV['HOME']}/.ssh/id_rsa"]	
	config.vm.synced_folder ".", "/vagrant", type: "rsync"
  config.vm.box = "bento/ubuntu-22.04"
  config.vm.box_check_update = true

  config.vm.define "master" do |master|
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    config.vm.provider "virtualbox" do |vb, override|
      vb.hd_size = "10G"
      vb.cpu_count = 2
      vb.memory_mb = 2048
      vb.image_name = "focal"
      vb.mount_point = {
					"/Users" => "/mnt/Users" # Sample mount path for MacOS Users
					# "/home" => "/mnt/Users" # Sample mount path for Linux Users
			}
    end
    master.vm.provision "shell", path: "scripts/common.sh"
    master.vm.provision "shell", path: "scripts/master.sh"

    master.trigger.after :up do |trigger|
      trigger.info = "Get IP Address"
      trigger.ruby do |env, machine|
        puts machine.ssh_info
      end
    end
  end
  
  (1..NUM_WORKER_NODES).each do |i|
    config.vm.define "node0#{i}" do |node|
      node.vm.hostname = "worker-node0#{i}"
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
      node.vm.provider "virtualbox" do |vb|
        vb.hd_size = "5G"
        vb.memory = 2048
        vb.cpus = 1
        vb.image_name = "focal"
        vb.mount_point = {
					"/Users" => "/mnt/Users" # Sample mount path for MacOS Users
					# "/home" => "/mnt/Users" # Sample mount path for Linux Users
			}    
      end
      node.vm.provision "shell", path: "scripts/common.sh"
      node.vm.provision "shell", path: "scripts/node.sh"

      node.trigger.after :up do |trigger|
        trigger.info = "Get IP Address"
        trigger.ruby do |env, machine|
          puts machine.ssh_info
        end
      end
    end
  end

end 