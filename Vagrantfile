Vagrant.require_version ">= 1.8.2"

domain = 'localdomain'
box = "bento/ubuntu-18.04"
ram_size_mb = '16384'
data_disk_size_gb = 100

vault_config_file = 'shared/vault_config'
if !File.file?(vault_config_file)
  raise "Run ./staging/scripts/vault-prepare first!"
end
vault_config = Hash[File.read(vault_config_file).split("\n").
                     map{|s| s.split('=')}]

Vagrant.configure(2) do |config|
  config.vm.box = box
  config.vm.synced_folder 'shared', '/vagrant'

  # Configure a second disk
  config.persistent_storage.enabled = true
  config.persistent_storage.mountname = "data"
  config.persistent_storage.filesystem = "ext4"
  config.persistent_storage.mountpoint = "/mnt/data"
  config.persistent_storage.size = data_disk_size_gb * 1024
  config.persistent_storage.location = "disk.vdi"

  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-docker'
    shell.args = ['vagrant']
    shell.env = {"DOCKER_LARGE_DISK" => "/mnt/data"}
    shell.reset = true
  end
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-pip'
  end
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-vault'
  end

  config.vm.provider :virtualbox do | vbox |
    vbox.gui = false
    vbox.memory = ram_size_mb
  end
  config.vm.network :private_network, ip: '192.168.81.11'
  config.vm.network "forwarded_port", guest: 1443, host: 1443
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-hostname'
    shell.args = 'testing'
  end

  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-orderly-web'
    shell.env = vault_config
    shell.privileged = false
  end
end
