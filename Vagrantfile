Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.hostname = "agcp-test-node"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  # Installs Ansible INSIDE this VM and runs the playbook against itself.
  # No WSL, no separate Ansible install on your Windows machine needed.
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "site.yml"
    ansible.inventory_path = "inventory/hosts_local.ini"
    ansible.limit = "all"
    ansible.install_mode = "pip"
  end
end
