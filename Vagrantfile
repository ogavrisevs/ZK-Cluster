
# max 9
instance_count = 3

servers = []
id = 0
while id < instance_count
  name = 'node' + (id).to_s
  ip = '192.168.100.' + (100 + id).to_s
  servers << {:name => name, :ip => ip}
  id += 1
end

Vagrant.configure("2") do |config|

  #
  # We will spin up ZK nodes
  #
  servers.each do |options|
    config.vm.define "#{options[:name]}" do |node|
      node.vm.box = "Centos.6.5"
      node.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"

      node.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "512"]
        vb.customize ["modifyvm", :id, "--cpus", "1"]
      end

      node.vm.network :private_network, ip: "#{options[:ip]}"
      node.vm.hostname = "#{options[:name]}"
      node.vm.provision :shell, :inline => "echo hello my name is `hostname`"
    end
  end

  #
  # And one ansible host for provisioning ZK nodes
  #
  config.vm.define "ansible" do |ansible|
    ansible.vm.box = "Centos.6.5"
    ansible.vm.hostname = "ansible"
    ansible.vm.network :private_network, ip: "192.168.100.200"
    ansible.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"

    #
    # istall ansible and dependent roles from Ansible Galaxy
    #
    ansible.vm.provision :shell, :inline => "yum install -y ansible"
    ansible.vm.provision :shell, :inline => "/usr/bin/ansible-galaxy install --force --role-file /vagrant/requirements.yml"

    #
    # Lets test can we reache zk nodes from vagrant host (ansible is req. on Vagrant host!)
    #
    ansible.vm.provision "ansible" do |ansible_prov|
      ansible_prov.playbook = "ping.yml"
      ansible_prov.host_key_checking = "false"
      ansible_prov.limit = "node*"
    end

    #
    # Preparing ansible
    #
    ansible.vm.provision :shell, :inline =>
      "   chmod 600 /vagrant/.ssh/id_rsa \
       && echo 'export ANSIBLE_HOST_KEY_CHECKING=False' >> /home/vagrant/.bashrc \
       && echo '192.168.100.10[0:%s]' > /vagrant/inventory" % (instance_count-1).to_s

    #
    # Run ansible inside VM, install dep. for ZK cluster
    #
    ansible.vm.provision :shell, :inline =>
      "/usr/bin/ansible-playbook \
         --inventory-file=/vagrant/inventory \
         --user=vagrant \
         --private-key=/vagrant/.ssh/id_rsa \
         /vagrant/zk.yml
      "
  end
end


