# NSM
## Repository voor de presentatie van een monitoring applicatie

Ons plan was een monitoring omgeving op te zetten, een master en twee slave nodes, volledig gebruik makende van Vagrant.

We begonnen dan ook met onze eigen vagrant box te maken waarop dan alle nodig applicaties al op geïnstalleerd zijn.

Ubuntu live server leek hiervoor een goede keuze dus deze hebben wij dan ook gedownload van de [Official Ubuntu Website](https://ubuntu.com/download/server)

Dan hebben wij de volgende youtube tutorial gevolgd om te leren hoe wij een vagrant box kunnen aanmaken: [Understand how to create your own custom Vagrant Box](https://www.youtube.com/watch?v=Wqtlj9osK0g)

Eenmaal de creatie van een box was gelukt konden wij verder de nodige applicaties installeren op de virtuele machine om ze dan nogmaals te verpakken als box en ze op te laden op [Vagrant Cloud](https://app.vagrantup.com/)

Nu de box in orde was konden we beginnen aan het schrijven van de Vagrantfile. In deze maken wij 3 vm's aan, namelijk een master node waarop dus Prometheus en Grafana-server op geïnstalleerd zijn en twee slave nodes waarop enkel Node exporter geïnstalleerd is. Vooraleer Prometheus de slave nodes kan scrapen moeten deze Node exporter runnen waardoor ze luisteren op poort 9100 naar de scrape requests van Prometheus op de master node.

Zodat Prometheus weet welke targets hij moet scrapen moet men deze toevoegen aan de Prometheus config, hiervoor hebben wij gekozen om de machines in een "private network" te steken om zo controle te hebben over welke ip adressen gebruikt worden (192.168.50.4, 192.168.50.5, 192.168.50.6).

Verder moesten voor elke machine apart dus Node exporter installeren, maar hier stootten we op een klein probleem. Wanneer wij deze machines opstarten via Vagrant, kwam het opstart proces vast te zitten vanaf het moment dat Node exporter opstart op de eerste machine in de opstart queue. Dit omdat de applicatie niet in de background loopt bij de initeel gekozen manier van opstarten. Gelukkig vonden wij hier een oplossing voor waardoor Node exporter toch in de background kan lopen door een actuele service uit de applicatie te maken zoals u in de Vagrantfile kan zien. Nu verliep het opstarten van de machines zonder stoppen. 

Om niet telkens de configuratie van Prometheus en de service file voor Node exporter gaan aan te maken in de machines zelf hebben we deze in configs map gestoken in de Vagrant folder en kopieer we deze files naar de correcte locatie bij het opstarten van de machine zoals u ook kan zien in de Vagrantfile.

We konden de .tar van Node exporter toevoegen aan de Vagrantfolder maar wij wilden de installatie van alles zo simpel mogelijk maken. In deze opzet moet je enkel de git repo clonen, in de folder gaan en Vagrant up uitvoeren.
Dan kan men de nodige applicaties bezoeken via http://localhost:3000, http://localhost:9100, http://localhost:9090. En men zal zien op de Prometheus web gui dat bij targets alle machines zichtbaar en online zijn om te scrapen.