# -*- mode: ruby -*-
# vi: set ft=ruby :

## Cassandra cluster settings
server_count = 3
network = '192.168.2.'
first_ip = 10

servers = []
seeds = []
cassandra_tokens = []
(0..server_count-1).each do |i|
  name = 'node' + (i + 1).to_s
  ip = network + (first_ip + i).to_s
  seeds << ip
  servers << {'name' => name,
              'ip' => ip,
              'initial_token' => 2**127 / server_count * i}
end
clientip = network + (seeds.last.split('.').last.to_i + 1).to_s

#Vagrant::Config.run do |config|
Vagrant.configure("2") do |config|
  servers.each do |server|
    config.vm.define server['name'] do |config2|
      config2.vm.box = "precise"
      config2.vm.box_url = "http://files.vagrantup.com/precise64.box"
      config2.vm.hostname = server['name']
      config2.vm.network :private_network, ip: server['ip']
      config2.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 1024]
      end
      config2.vm.provision :shell, :inline => "if [ \"$(chef-client --version |awk '{print $2}')\" != \"10.16.4-1\" ]; then bash <(wget https://www.opscode.com/chef/install.sh -q --tries=10 -O -) -v 10.16.4-1 2>> /dev/null; fi"
      config2.vm.provision :chef_solo do |chef|
        chef.log_level = :debug
        chef.cookbooks_path = ["vagrant/cookbooks", "vagrant/site-cookbooks"]
        chef.add_recipe "updater"
        chef.add_recipe "java"
        chef.add_recipe "cassandra"
        chef.json = {
          :java      => {'install_flavor' => 'oracle',
                         :oracle          => {
                           "accept_oracle_download_terms" => true} },
          :cassandra => {'cluster_name' => 'graphite',
                         'seeds' => seeds,
                         'listen_address' => server['ip'],
                         'rpc_address' => server['ip']}
        }
      end
    end
  end
  config.vm.define "client" do |config2|
    config2.vm.box = "precise"
    config2.vm.box_url = "http://files.vagrantup.com/precise64.box"
    config2.vm.hostname = "client"
    config2.vm.network :private_network, ip: clientip
    config2.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", 1024]
    end
  end
end
