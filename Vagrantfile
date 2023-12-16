Vagrant.configure("2") do |config|
  # Master machine
  config.vm.define "master" do |master|
    master.vm.box = "delsynn/ubuntuliveprometheus"
    master.vm.box_version = "1.1.0"
    master.vm.network "private_network", ip: "192.168.50.4"
    master.vm.network "forwarded_port", guest: 3000, host: 3000
    master.vm.network "forwarded_port", guest: 9090, host: 9090
    master.vm.network "forwarded_port", guest: 9100, host: 9100
    master.vm.synced_folder ".", "/home/vagrant"
    master.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
      vb.gui = true
      vb.memory = "4096"
      vb.cpus = 2
    end
    master.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on master..."
      echo apt-get update
      sudo cp -f /home/vagrant/prometheus.yml /var/snap/prometheus/current/
      sudo snap restart prometheus.prometheus
      sudo systemctl start grafana-server
      sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      sudo tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      cd node_exporter-1.7.0.linux-amd64
      sudo ./node_exporter
    SHELL
  end

  # Slave machine 1
  config.vm.define "slave1" do |slave1|
    slave1.vm.box = "generic/alpine318"
    slave1.vm.network "private_network", ip: "192.168.50.5"
    slave1.vm.provider :virtualbox do |vb|
      vb.gui = true
      vb.memory = "512"
      vb.cpus = 1
    end
    slave1.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on slave1..."
      sudo apt-get update
      sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      sudo tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      cd node_exporter-1.7.0.linux-amd64
      sudo ./node_exporter
    SHELL
  end

  # Slave machine 2
  config.vm.define "slave2" do |slave2|
    slave2.vm.box = "generic/alpine318"
    slave2.vm.network "private_network", ip: "192.168.50.6"
    slave2.vm.provider :virtualbox do |vb|
      vb.gui = true
      vb.memory = "512"
      vb.cpus = 1
    end
    slave2.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on slave2..."
      sudo apt-get update
      sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      sudo tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      cd node_exporter-1.7.0.linux-amd64
      sudo ./node_exporter
    SHELL
  end
end