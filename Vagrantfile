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
    r1.vm.box = "veos"
    r1.vm.hostname = "r1"

    r1.vm.provider "virtualbox" do |v|
      v.memory = 2048
    end
    r1.vm.synced_folder ".", "/vagrant", disabled: true

    r1.vm.network "private_network", ip: "192.168.100.11"
    r1.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      user stanley privilege 15 secret stanley
      management api http-commands
      interface Ethernet1
        no switchport
        ip address 192.168.100.11/24
      no shutdown
      end
      copy running-config startup-config"
    SHELL
  end

  config.vm.define "r2" do |r2|
    r2.vm.box = "veos"
    r2.vm.hostname = "r2"

    r2.vm.provider "virtualbox" do |v|
      v.memory = 2048
    end
    r2.vm.synced_folder ".", "/vagrant", disabled: true

    r2.vm.network "private_network", ip: "192.168.100.12"
    r2.vm.provision "shell", inline: <<-SHELL
      sleep 30
      FastCli -p 15 -c "configure
      user stanley privilege 15 secret stanley
      management api http-commands      
      interface Ethernet1
        no switchport
        ip address 192.168.100.12/24
      no shutdown
      end
      copy running-config startup-config"
    SHELL
  end
end

