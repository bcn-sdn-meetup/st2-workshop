Vagrant.require_version('>= 2.0')
unless Vagrant.has_plugin?('vagrant-vyos')
  system('vagrant plugin install vagrant-vyos') || exit!
  exit system('vagrant', *ARGV)
end

Vagrant.configure("2") do |config|

  config.vm.define "st2" do |st2|
    st2.vm.box = "stackstorm/st2"
    st2.vm.provider "virtualbox" do |v|
      v.name = "stackstorm"
    end

    st2.vm.network "private_network", ip: "192.168.100.10"
  end

  config.vm.define "r1" do |r1|
    r1.vm.box = "higebu/vyos"
    r1.vm.hostname = "r1"
    r1.vm.box_version = "1.1.7"

    r1.vm.provider "virtualbox" do |v|
      v.name = "r1"
      v.memory = 768
    end
    r1.vm.synced_folder ".", "/vagrant", disabled: true

    r1.vm.network "private_network", ip: "192.168.100.11"
  end

  config.vm.define "r2" do |r2|
    r2.vm.box = "higebu/vyos"
    r2.vm.hostname = "r2"
    r2.vm.box_version = "1.1.7"

    r2.vm.provider "virtualbox" do |v|
      v.name = "r2"
      v.memory = 768
    end
    r2.vm.synced_folder ".", "/vagrant", disabled: true

    r2.vm.network "private_network", ip: "192.168.100.12"
  end


end

