# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES=2
DEFAULT_IP="192.168.50.100"
DNS_SUFFIX="test.com"

def node_ip(i)
  "192.168.50.#{i+100}"
end

def generate_hosts(default)
  hosts = "#{DEFAULT_IP} #{default} #{default}.#{DNS_SUFFIX} \n" if defined? default
  for i in 1..MACHINES do
    hosts << "#{node_ip i} node#{i}\n"
  end
  hosts
end

hosts=generate_hosts("tower")

Vagrant.configure("2") do |config|

  config.vm.box = "rhel-7.4-vbox"

  config.vm.box_check_update = false

  config.vbguest.auto_update = false
  
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  config.ssh.forward_x11 = true

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = "2"
    vb.memory = "2048"
  end

  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.manager = 'subscription_manager'
    config.registration.unregister_on_halt = false
    config.registration.username = ENV['SUBSCRIPTION_USERNAME']
    config.registration.password = ENV['SUBSCRIPTION_PASSWORD']
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.define "tower", primary: true  do |c|
    c.vm.hostname = "tower"
    c.vm.network :private_network, ip: DEFAULT_IP
    c.vm.network "forwarded_port", guest: 80, host: 8888
    c.vm.network "forwarded_port", guest: 443, host: 8443
  end

  (1..MACHINES).each do |i|
    config.vm.define "node#{i}" do |c|
      c.vm.hostname = "node#{i}"
      c.vm.network :private_network, ip: node_ip(i)
      c.vm.provider "virtualbox" do |vb|
        vb.cpus = "1"
        vb.memory = "512"
      end
    end
  end

  config.vm.define 'doze' do |c|
    i = MACHINES + 1
    c.vm.box = "jborean93/WindowsServer2012R2"
    c.vm.hostname = "node#{i}"
    c.vm.network :private_network, ip: node_ip(i)
#    c.vm.provision "shell", inline: <<-SHELL
#      winrm quickconfig -q
#      winrm set winrm/config/winrs @{MaxMemoryPerShellMB="512"}
#      winrm set winrm/config @{MaxTimeoutms="1800000"}
#      winrm set winrm/config/service @{AllowUnencrypted="true"}
#      winrm set winrm/config/service/auth @{Basic="true"}
#      sc config WinRM start= auto
#    SHELL
  end

  config.vm.define 'doze2' do |c|
    i = MACHINES + 2
    c.vm.box = "mwrock/Windows2016"
    c.vm.hostname = "node#{i}"
    c.vm.network :private_network, ip: node_ip(i)
  end

  config.vm.provision "shell", inline: <<-SHELL
      ln -s /vagrant /root/src
      ln -s /vagrant /home/vagrant/src

      echo -e #{hosts} >>/etc/hosts

      cp -R /vagrant/gen/key /root/.ssh
      cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

      yum makecache fast
      yum install -y xauth 
      yum install -y emacs
   SHELL
end
