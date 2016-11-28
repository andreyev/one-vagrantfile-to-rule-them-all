# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
settings = YAML::load_file(File.join(__dir__, 'Vagrantdata.yml'))
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
vmname = File.basename(Dir.getwd)
required_plugins = %w(vagrant-google vagrant-aws vagrant-cachier vagrant-digitalocean vagrant-scp vagrant-vbguest vagrant-vbox-snapshot vagrant-puppet-install vagrant-librarian-puppet vagrant-dnsmasq)
required_plugins.each do |plugin|
  system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end

servers=[
  { :hostname => "sample", :fromport => 80, :toport => 8080, },
]

Vagrant.configure(2) do |config|
  config.dnsmasq.domain = '.dev'
  config.vm.box = "dummy"
  config.vm.synced_folder ".", "/tmp/"+ vmname
  #override.ssh.pty = true
  config.ssh.pty = true
  config.vm.provider "virtualbox" do |provider, override|
    override.vm.network "private_network", type: "dhcp"
    override.vm.box = "centos/7"
    override.cache.scope = :box
    provider.memory = settings['vb.memory']
    provider.cpus = settings['vb.cpus']
  end
  config.vm.provider :digital_ocean do |provider, override|
    override.vm.box = 'digital_ocean'
    override.vm.box_url = "https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box"
    provider.token = settings['do.token']
    provider.image = settings['do.image']
    provider.region = settings['do.region']
    provider.size = settings['do.size']
    provider.setup = true
    override.ssh.private_key_path = settings['general.private_key_path']
    override.nfs.functional = false
  end
  config.vm.provider :aws do |provider, override|
    provider.access_key_id = settings['aws.access_key_id']
    provider.secret_access_key = settings['aws.secret_access_key']
    provider.keypair_name = settings['aws.keypair_name']
    provider.ami = settings['aws.ami']
    provider.region = settings['aws.region']
    provider.instance_type = settings['aws.instance_type']
    provider.keypair_name = settings['aws.keypair_name']
    provider.associate_public_ip = true
    provider.elastic_ip = true
    provider.subnet_id = 'subnet-c95a1891'
    override.ssh.private_key_path = settings['general.private_key_path']
    override.ssh.username = settings['aws.ssh_username']
    override.nfs.functional = false
  end
  servers.each do |machine|
    config.vm.define vmname + '-' + machine[:hostname] do |default|
      default.vm.hostname = vmname + '-' + machine[:hostname]
      default.vm.network "forwarded_port", guest: machine[:fromport], host: machine[:toport]
      default.vm.provider :aws do |provider, override|
        provider.tags = { 'Name' => vmname + '-' + machine[:hostname], }
      end
      default.puppet_install.puppet_version = "3.8.1"
      default.librarian_puppet.puppetfile_dir = "."
      default.vm.provision "puppet" do |puppet|
        puppet.manifests_path = "."
        puppet.module_path = "modules"
        puppet.manifest_file = 'provision-' + machine[:hostname] + '.pp'
        puppet.facter = {}
          ENV.each do |key, value|
          next unless key =~ /^FACTER_/
          puppet.facter[key.gsub(/^FACTER_/, "")] = value
        end
      end
    end
  end
end
