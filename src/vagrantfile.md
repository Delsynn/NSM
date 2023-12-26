# Aanmaken van de Vagrantfile

Nu wij de monitoring box hebben opgeladen op Vagrant Cloud kunnen wij deze gebruiken in de aanmaak van onze Vagrantfile.

Om de volledige opzet te laten werken zijn we na veel testen en aanpassen van de Vagrantfile (syntax of lijnen toevoegen) uitgekomen op de volgende "code":

```
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
```
## Master Box

Bij de monitoring machine (master) kan men zien dat wij onze zelfgemaakte box gaan downloaden:

```
master.vm.box = "delsynn/ubuntuliveprometheus"
```
We maken een privé netwerk en zullen alle machines in hetzelfde subnet steken:

```
master.vm.network "private_network", ip: "192.168.50.4"
```
Wij gaan de ports van de verschillende applicaties forwarden zodat we deze via onze browser lokaal kunnen bereiken:

```
master.vm.network "forwarded_port", guest: 3000, host: 3000
master.vm.network "forwarded_port", guest: 9090, host: 9090
master.vm.network "forwarded_port", guest: 9100, host: 9100
```
En we syncen onze lokale configs folder met de /home/vagrant folder op de machine, dit omdat hier enkele configuratie files insteken voor prometheus en een .service file voor node exporter zodat we van Node Exporter een applicatie kunnen maken die in de achtergrond loopt:

```
master.vm.synced_folder "./configs", "/home/vagrant"
```
Deze bestanden zullen wij dan naar de correcte locaties kopiëren op de master machine.

Het gaat hier over de prometheus.yaml configuratie:

```
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.50.4:9100', '192.168.50.5:9100', '192.168.50.6:9100']
```

Hier kan men zien dat wij de ip's van de drie machines hebben toegevoegd als doelwitten voor het scrapen van de prometheus applicatie op de poort waarvan node exporter gebruik maakt (9100):

- 192.168.50.4:9100
- 192.168.50.5:9100
- 192.168.50.6:9100

En dan nog een .service file voor de Node Exporter applicatie, dit is nodig zodat Node Exporter in de achtergrond kan lopen:

```
[Unit]
Description=Node Exporter
After=network.target
 
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
 
[Install]
WantedBy=multi-user.target
```
Dit was nodig want we merkten dat tijdens het opstarten van de verschillende machines in de vagrantfile het proces kwam vast te zitten wanneer de Node Exporter applicatie opstart bij de eerste machine in de Vagrantfile (dus de master machine/monitor). Dit omdat deze applicatie out-of-the-box gewoon in de voorgrond wordt uitgevoerd.

Verder kennen we voldoende resources aan de machine toe:

```
master.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
```

En uiteindelijk gaan via de "shell" provisioner wat commando's uitvoeren op de machine:

```
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
```

Zoals men kan zien doen wij de volgende zaken:
- We starten prometheus.
- We kopiëren de configuratiefile van prometheus en de .service file van node exporter naar de correctie locaties.
- We herstarten prometheus.
- We zorgen dat grafana-server vanzelf opstart in de toekomst, dit hadden wij eigenlijk al kunnen doen bij het voorbereiden van de machine (aanmaken van een symbolische link).
- En we starten grafana-server op.
- Dan gaan wij in de /tmp folder om node exporter te downloaden, dit zodat deze file niet steeds wordt gedownload in de gesyncte folder.
- We downloaden, extracten, en verplaatsten de node exporter applicatie naar de correcte folder (/usr/local/bin)
- Een system user wordt aangemaakt voor de uitvoeren van de node_exporter service.
- We herladen de systemd manager configuratie zodat de node_exporter.service file werkt.
- En zorgen dat de node-exporter service ook opstart bij het rebooten van de machine.
- We starten dan uiteindelijk de node-exporter service op.

## Slave boxes

De enige verschillen bij de slave boxes is dat we gebruik maken van een generieke ubuntu box die wij op Vagrant Cloud hebben gevonden. Een ip in hetzelfde subnet as de master box eraan toekennen en dezelfde commando's uitvoeren als op de master box om de node_exporter service op deze machines op te starten.

```
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
```