Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/bionic64'

  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 1024
  end

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.provision 'shell', inline: <<-SHELL
    apt-get update
    apt-get install -y \
      python-minimal \
      python-apt
    adduser vagrant sudo
  SHELL
end
