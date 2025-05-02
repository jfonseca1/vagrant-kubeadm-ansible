IMAGE_NAME = "tknerr/baseimage-ubuntu:18.04"
N = 2

Vagrant.configure("2") do |config|
  config.ssh.insert_key = true
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.vm.boot_timeout = 600
  
  # Base provisioning - install Python for Ansible
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    DEBIAN_FRONTEND=noninteractive apt-get install -y python3 python3-pip ansible
  SHELL

  # Use different base ports for each node to avoid conflicts
  config.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh", auto_correct: true

  # Docker provider configuration
  config.vm.provider "docker" do |d|
    d.image = IMAGE_NAME
    d.has_ssh = true
    d.remains_running = true
    d.privileged = true
    d.create_args = [
    "--cap-add=NET_ADMIN",
    "--cap-add=SYS_ADMIN",
    "--volume=/sys/fs/cgroup:/sys/fs/cgroup:rw" # Important for cgroups
  ]
  end

  # Master node configuration
  config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", ip: "192.168.56.10", docker_network: "k8s-net"
    
    # Additional provisioning for master
    master.vm.provision "shell", inline: <<-SHELL
      # Disable swap
      swapoff -a
      sed -i '/ swap / s/^/#/' /etc/fstab
      
      # Load required kernel modules
      modprobe br_netfilter
      echo 'br_netfilter' >> /etc/modules-load.d/k8s.conf
      
      # Set sysctl params
      echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/k8s.conf
      echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf
      sysctl --system
      
      # Run Ansible playbook
      cd /vagrant
      ansible-playbook -i 'localhost,' -c local kubernetes-setup/master-playbook.yml
    SHELL
  end

  # Worker nodes configuration
  (1..N).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.hostname = "node-#{i}"
      node.vm.network "private_network", ip: "192.168.56.#{i + 10}", docker_network: "k8s-net"
      
      # Additional provisioning for workers
      node.vm.provision "shell", inline: <<-SHELL
        # Disable swap
        swapoff -a
        sed -i '/ swap / s/^/#/' /etc/fstab
        
        # Load required kernel modules
        modprobe br_netfilter
        echo 'br_netfilter' >> /etc/modules-load.d/k8s.conf
        
        # Set sysctl params
        echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/k8s.conf
        echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf
        sysctl --system
        
        # Run Ansible playbook
        cd /vagrant
        ansible-playbook -i 'localhost,' -c local kubernetes-setup/node-playbook.yml
      SHELL
    end
  end
end
