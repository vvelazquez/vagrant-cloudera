# -*- mode: ruby -*-
# vi: set ft=ruby :

# You need to have the following vagrant plugins installed
#       vagrant-lxc
#       vagrant-hostmanager

boxes = [
  { :name => :namenode, :ip => '10.110.55.40', :cpus =>2, :memory => 1024, :instance => 'm1.small' },
  { :name => :datanode1, :ip => '10.110.55.41', :cpus =>4, :memory => 4096, :instance => 'm1.medium' },
  { :name => :datanode2, :ip => '10.110.55.42', :cpus =>4, :memory => 4096, :instance => 'm1.medium' },
  { :name => :hbasenode, :ip => '10.110.55.45', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
  { :name => :hivenode, :ip => '10.110.55.46', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
  { :name => :client, :ip => '10.110.55.46', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
  { :name => :mysql, :ip => '10.110.55.50', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
  { :name => :zookeeper1, :ip => '10.110.55.60', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
  { :name => :zeppelin, :ip => '10.110.55.70', :cpus =>1, :memory => 1024, :instance => 'm1.small' },
]

BASE_NAME = 'cloudera'
DOMAIN_NAME = "%s.vagrant" % BASE_NAME
LXC_BRIDGE = 'lxcbr0'
AWS_REGION = ''
AWS_AMI = ''


Vagrant.configure("2") do |config|

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
    config.hostmanager.aliases_on_separate_lines = false
  end

  boxes.each do |opts|
  	config.vm.define opts[:name] do |node|
      node.vm.hostname = "%s.%s" % [opts[:name].to_s, DOMAIN_NAME]
  	  node.vm.box = "trusty64"
      node.vm.box_url = "http://files.vagrantup.com/trusty64.box"

      if Vagrant.has_plugin?("vagrant-hostmanager")
        # Also setup aliases for /etc/hosts with and without domainname
        node.hostmanager.aliases = [opts[:name].to_s]
      end
      
      node.vm.provider :aws do |aws, override|
        override.vm.box = "dummy"
    	override.ssh.username = "ubuntu"
    	override.ssh.private_key_path = ""
    	aws.access_key_id = ""
    	aws.secret_access_key = ""
    	aws.keypair_name = ""
    	aws.region = AWS_REGION
    	aws.ami    = AWS_AMI
    	aws.private_ip_address = opts[:ip]
		aws.subnet_id = "" 
    	aws.instance_type = opts[:instance]
  	  end
      
      node.vm.provider :virtualbox do |vb, override|
        override.vm.network :private_network, ip: opts[:ip]
        vb.name = "%s.%s" % [BASE_NAME,opts[:name].to_s]
        vb.customize ["modifyvm", :id, "--memory", opts[:memory]]
        vb.customize ["modifyvm", :id, "--cpus", opts[:cpus] ] if opts[:cpus]
      end
      
      node.vm.provider :lxc do |lxc, override|
      	override.vm.box = "fgrehm/trusty64-lxc"
   	    override.vm.box_url = "https://atlas.hashicorp.com/fgrehm/boxes/trusty64-lxc/versions/1.2.0/providers/lxc.box"
        # override.vm.network :private_network, ip: opts[:ip], lxc__bridge_name: LXC_BRIDGE
        lxc.container_name = "%s.%s" % [BASE_NAME,opts[:name].to_s]
        # lxc.customize 'cgroup.memory.limit_in_bytes', opts[:memory].to_s + "M"
        lxc.customize 'network.type', 'veth'
        lxc.customize 'network.link', LXC_BRIDGE
      end

      # install librarian-puppet and run it to install puppet common modules.
      # This has to be done before puppet provisioning so that modules are available
      # when puppet tries to parse its manifests
      config.vm.provision :shell, :path => "provision/scripts/main.sh"
      
      node.vm.provision :puppet do |puppet|
    	puppet.manifests_path = "provision/puppet/manifests"
        puppet.manifest_file = 'site.pp'
        puppet.module_path = [ 'provision/puppet/modules' ]
        puppet.options = "--verbose --debug"
  	  end
    end
  end
end

