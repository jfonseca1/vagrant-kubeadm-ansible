IMAGE_NAME = "tknerr/baseimage-ubuntu:18.04"
N = 2
Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.ssh.username = "vagrant"
    config.ssh.password = "vagrant"
    config.vm.boot_timeout = 600
    config.vm.provision "shell", inline: "apt-get update && apt-get install -y python3 python3-pip ansible"
    config.vm.network :forwarded_port, guest: 22, host: 2222, auto_correct: true
    
    # Use docker provider with an image specifically built for Vagrant
    config.vm.provider "docker" do |d|
        d.image = IMAGE_NAME
        d.has_ssh = true
        d.remains_running = true
        d.privileged = true  # Needed for systemd and many Kubernetes components
        d.create_args = ["--cap-add=NET_ADMIN", "--cap-add=SYS_ADMIN"]  # Required capabilities
    end
      
    config.vm.define "k8s-master" do |master|
        master.vm.hostname = "k8s-master"
        master.vm.network "private_network", ip: "192.168.56.10", docker_network: "k8s-net"
        
        # Run Ansible directly in the container instead of from the host
        master.vm.provision "shell", inline: <<-SHELL
          cd /vagrant
          ansible-playbook -i 'localhost,' -c local kubernetes-setup/master-playbook.yml -e "node_ip=192.168.56.10"
        SHELL
    end
    
    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.hostname = "node-#{i}"
            node.vm.network "private_network", ip: "192.168.56.#{i + 10}", docker_network: "k8s-net"
            
            # Run Ansible directly in the container instead of from the host
            node.vm.provision "shell", inline: <<-SHELL
              cd /vagrant
              ansible-playbook -i 'localhost,' -c local kubernetes-setup/node-playbook.yml -e "node_ip=192.168.56.#{i + 10}"
            SHELL
        end
    end
end
