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
      cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
      cp -v /vagrant/tierra_files/named.conf.local /etc/bind/named.conf.local
      cp -v /vagrant/tierra_files/sistema.test.dns /var/lib/bind/sistema.test.dns
      cp -v /vagrant/tierra_files/57.168.192.in-addr.arpa.dns /var/lib/bind/57.168.192.in-addr.arpa.dns
      cp -v /vagrant/tierra_files/resolv.conf /etc/resolv.conf
      sudo systemctl restart bind9
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
      cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
      cp -v /vagrant/venus_files/named.conf.local /etc/bind/named.conf.local
      sudo systemctl restart bind9
    SHELL
  end
end