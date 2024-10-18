Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.define "master" do |master|
    master.vm.hostname = "tierra.sistema.test"
    master.vm.network "private_network", ip: "192.168.57.103"
    master.vm.provision "shell", name: "update", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y bind9
      cp -v /vagrant/named /etc/default/named
      sudo systemctl restart named
    SHELL
  end

  config.vm.define "dnslave" do |dnslave|
    dnslave.vm.hostname = "venus.sistema.test"
    dnslave.vm.network "private_network", ip: "192.168.57.102"
    dnslave.vm.provision "shell", name: "update", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y bind9
      cp -v /vagrant/named /etc/default/named
      sudo systemctl restart named
    SHELL
  end
end
