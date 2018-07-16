# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative 'vagrant'

# Absolute paths on the host machine.
host_badzillavm_dir = File.dirname(File.expand_path(__FILE__))
host_project_dir = ENV['BADZILLAVM_PROJECT_ROOT'] || host_badzillavm_dir
host_config_dir = ENV['BADZILLAVM_CONFIG_DIR'] ? "#{host_project_dir}/#{ENV['BADZILLAVM_CONFIG_DIR']}" : host_project_dir

# Absolute paths on the guest machine.
guest_project_dir = '/vagrant'
guest_badzillavm_dir = ENV['BADZILLAVM_DIR'] ? "/vagrant/#{ENV['BADZILLAVM_DIR']}" : guest_project_dir
guest_config_dir = ENV['BADZILLAVM_CONFIG_DIR'] ? "/vagrant/#{ENV['BADZILLAVM_CONFIG_DIR']}" : guest_project_dir

badzillavm_env = ENV['BADZILLAVM_ENV'] || 'vagrant'

default_config_file = "#{host_badzillavm_dir}/default.config.yml"
unless File.exist?(default_config_file)
  raise_message "Configuration file not found! Expected in #{default_config_file}"
end

vconfig = load_config([
  default_config_file,
  "#{host_config_dir}/config.yml",
  "#{host_config_dir}/#{badzillavm_env}.config.yml",
  "#{host_config_dir}/local.config.yml"
])

provisioner = vconfig['force_ansible_local'] ? :ansible_local : vagrant_provisioner
if provisioner == :ansible
  playbook = "#{host_badzillavm_dir}/playbook.yml"
  config_dir = host_config_dir

  # Verify Ansible version requirement.
  require_ansible_version ">= #{vconfig['badzillavm_ansible_version_min']}"
else
  playbook = "#{guest_badzillavm_dir}/playbook.yml"
  config_dir = guest_config_dir
end

# Verify Vagrant version requirement.
Vagrant.require_version ">= #{vconfig['badzillavm_vagrant_version_min']}"

ensure_plugins(vconfig['vagrant_plugins'])

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # Set the name of the VM. See: http://stackoverflow.com/a/17864388/100134
  config.vm.define vconfig['vagrant_machine_name']

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Networking configuration.
  config.vm.hostname = vconfig['vagrant_hostname']
  config.vm.network :private_network,
    ip: vconfig['vagrant_ip'],
    auto_network: vconfig['vagrant_ip'] == '0.0.0.0' && Vagrant.has_plugin?('vagrant-auto_network')

  unless vconfig['vagrant_public_ip'].empty?
    config.vm.network :public_network,
      ip: vconfig['vagrant_public_ip'] != '0.0.0.0' ? vconfig['vagrant_public_ip'] : nil
  end

  # SSH options.
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = vconfig['vagrant_box']

  # Display an introduction message after `vagrant up` and `vagrant provision`.
  config.vm.post_up_message = vconfig.fetch('vagrant_post_up_message', get_default_post_up_message(vconfig))

  # Synced folders.
  vconfig['vagrant_synced_folders'].each do |synced_folder|
    options = {
      type: synced_folder.fetch('type', vconfig['vagrant_synced_folder_default_type']),
      rsync__exclude: synced_folder['excluded_paths'],
      rsync__args: ['--verbose', '--archive', '--delete', '-z', '--copy-links', '--chmod=ugo=rwX'],
      id: synced_folder['id'],
      create: synced_folder.fetch('create', false),
      mount_options: synced_folder.fetch('mount_options', []),
      nfs_udp: synced_folder.fetch('nfs_udp', false)
    }
    synced_folder.fetch('options_override', {}).each do |key, value|
      options[key.to_sym] = value
    end
    config.vm.synced_folder synced_folder.fetch('local_path'), synced_folder.fetch('destination'), options
  end

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end

end