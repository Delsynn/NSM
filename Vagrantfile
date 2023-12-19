Vagrant.configure("2") do |config|
  # Master machine
  config.vm.define "master" do |master|
    master.vm.box = "delsynn/ubuntuliveprometheus"
    master.vm.box_version = "1.1.0"
    master.vm.network "private_network", ip: "192.168.50.4"
    master.vm.network "forwarded_port", guest: 3000, host: 3000
    master.vm.network "forwarded_port", guest: 9090, host: 9090
    master.vm.network "forwarded_port", guest: 9100, host: 9100
    master.vm.synced_folder "./configs", "/home/vagrant"
    master.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
    end
    master.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on master..."
      sudo snap start prometheus.prometheus
      sudo cp -f /home/vagrant/prometheus.yml /var/snap/prometheus/current
      sudo cp -f /home/vagrant/node_exporter.service /etc/systemd/system
      sudo snap restart prometheus
      sudo systemctl enable grafana-server
      sudo systemctl start grafana-server
      cd /tmp
      wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
      sudo useradd -rs /bin/false node_exporter
      sudo systemctl daemon-reload
      sudo systemctl start node_exporter
      sudo systemctl enable node_exporter
    SHELL
  end

  # Slave machine 1
  config.vm.define "slave1" do |slave1|
    slave1.vm.box = "bento/ubuntu-16.04"
    slave1.vm.network "private_network", ip: "192.168.50.5"
    slave1.vm.synced_folder "./configs", "/home/vagrant"
    slave1.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
      vb.gui = false
      vb.memory = "2048"
      vb.cpus = 2
    end
    slave1.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on slave1..."
      sudo apt-get update
      sudo cp -f /home/vagrant/node_exporter.service /etc/systemd/system
      cd /tmp
      sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      sudo tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
      sudo useradd -rs /bin/false node_exporter
      sudo systemctl daemon-reload
      sudo systemctl start node_exporter
      sudo systemctl enable node_exporter
    SHELL
  end

  # Slave machine 2
  config.vm.define "slave2" do |slave2|
    slave2.vm.box = "bento/ubuntu-16.04"
    slave2.vm.network "private_network", ip: "192.168.50.6"
    slave2.vm.synced_folder "./configs", "/home/vagrant"
    slave2.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
      vb.gui = false
      vb.memory = "2048"
      vb.cpus = 2
    end
    slave2.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on slave2..."
      sudo apt-get update
      sudo cp -f /home/vagrant/node_exporter.service /etc/systemd/system
      cd /tmp
      sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      sudo tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
      sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
      sudo useradd -rs /bin/false node_exporter
      sudo systemctl daemon-reload
      sudo systemctl start node_exporter
      sudo systemctl enable node_exporter
    SHELL
  end
end