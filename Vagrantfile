Vagrant.configure("2") do |config|
  # Master machine
  config.vm.define "master" do |master|
    master.vm.box = "bento/ubuntu-16.04"
    master.vm.network "private_network", ip: "192.168.50.4"
    master.vm.network "forwarded_port", guest: 3000, host: 3000
    master.vm.network "forwarded_port", guest: 9090, host: 9090
    master.vm.network "forwarded_port", guest: 9100, host: 9100
    master.vm.synced_folder ".", "/home/vagrant"
    master.vm.provider :virtualbox do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
    end
    master.vm.provision "shell", inline: <<-SHELL
      echo "Running shell commands on master..."
      sudo apt-get install prometheus -y
      sudo systemctl enable prometheus
      sudo systemctl start prometheus
      sudo cp -f /home/vagrant/prometheus.yml /etc/prometheus
      sudo systemctl restart prometheus
      sudo apt-get install -y adduser libfontconfig1 musl
      wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.2.2_amd64.deb
      sudo dpkg -i grafana-enterprise_10.2.2_amd64.deb
      sudo systemctl enable grafana-server
      sudo systemctl start grafana-server
      wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
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