Vagrant.require_version('>= 2.0')
unless Vagrant.has_plugin?('vagrant-host-shell')
  system('vagrant plugin install vagrant-host-shell') || exit!
  exit system('vagrant', *ARGV)
end

unless Vagrant.has_plugin?('vagrant-junos')
  system('vagrant plugin install vagrant-junos') || exit!
  exit system('vagrant', *ARGV)
end

Vagrant.configure("2") do |config|

  config.vm.define "st2" do |st2|
    st2.vm.box = "stackstorm/st2"
    st2.vm.provider "virtualbox" do |v|
      v.name = "stackstorm"
    end

    st2.vm.network "private_network", ip: "192.168.100.10"
    st2.vm.provision "file", source: "files/napalm.yaml", destination: "/tmp/napalm.yaml"
    st2.vm.provision "shell", inline: <<-SHELL
	sudo cp /tmp/napalm.yaml /opt/stackstorm/configs/napalm.yaml
	export ST2_AUTH_TOKEN=`st2 auth -t -p 'Ch@ngeMe' st2admin`
	st2 pack install napalm
	st2ctl reload --register-configs
    SHELL
  end

  config.vm.define "r1" do |r1|
    r1.vm.box = "juniper/ffp-12.1X47-D15.4-packetmode"
    r1.vm.hostname = "r1"

    r1.vm.provider "virtualbox" do |v|
      v.memory = 1024
    end
    r1.vm.network "private_network", ip: "192.168.100.11"
    r1.vm.network "private_network", virtualbox__intnet: "net2", auto_config: false
  end

  config.vm.define "r2" do |r2|
    r2.vm.box = "juniper/ffp-12.1X47-D15.4-packetmode"
    r2.vm.hostname = "r2"

    r2.vm.provider "virtualbox" do |v|
      v.memory = 1024 
    end

    r2.vm.network "private_network", ip: "192.168.100.12"
    r2.vm.network "private_network", virtualbox__intnet: "net2", auto_config: false
  end
end

