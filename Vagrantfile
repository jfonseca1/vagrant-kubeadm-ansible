IMAGE_NAME = "generic/ubuntu2204" # Has both SSH and Ansible pre-installed  # This image has SSH set up  # Docker image
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.ssh.private_key_path = ["~/.vagrant.d/insecure_private_key", "~/.ssh/id_rsa"]
    config.vm.boot_timeout = 600
    config.vm.provision "shell", inline: "apt-get update && apt-get install -y python3 python3-pip"
    config.vm.network :forwarded_port, guest: 22, host: 2222, auto_correct: true
    # Set Docker as the provider with base settings
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
        master.vm.provision "ansible" do |ansible|
            ansible.compatibility_mode = "2.0"
            ansible.config_file = "ansible.cfg"
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: "192.168.56.10",
            }
        end
    end
    
    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.hostname = "node-#{i}"
            node.vm.network "private_network", ip: "192.168.56.#{i + 10}", docker_network: "k8s-net"
            node.vm.provision "ansible" do |ansible|
                ansible.compatibility_mode = "2.0"
                ansible.config_file = "ansible.cfg"
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.56.#{i + 10}",
                }
                # Add custom SSH args to use vagrant insecure key
                ansible.raw_ssh_args = ["-o IdentitiesOnly=yes", "-i ~/.vagrant.d/insecure_private_key"]
            end
        end
    end
end
