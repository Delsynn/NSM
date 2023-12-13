Vagrant.configure("2") do |config|
  config.vm.box = "delsynn/ubuntuliveprometheus"
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
    vb.gui = true
    vb.memory = "1024"
    vb.cpus = 2
  end
end
