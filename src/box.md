# Creatie van de monitoring box

Bij het creëren van een Vagrant box begint men bij het opzetten van een virtuele machine, hiervoor hadden wij gekozen voor het gebruik van de Ubuntu Server ISO die we hebben gedownload van de officiële [site](https://ubuntu.com/download/server).

De volgende stap was onderzoeken hoe het aanmaken van Vagrant box juist in zijn werking gaat, na wat opzoekwerk stuitten we op de volgende zeer behulpzame [video](https://www.youtube.com/watch?v=Wqtlj9osK0g).

In de video legde men uit, dat voor een succesvol werkende box aan te maken wij de volgende zaken moesten doen:

1. ## Aanmaak VM 

    We beginnen wij het aanmaken van een virtuele machine in Virtualbox, bij het doorlopen van de creatie wizard is het belangrijk de volgende zaken te doen:

    - Het kiezen van een duidelijke naam, dit hebben wij nodig bij het omzetten van de VM naar een box.
    - Dan het selecteren van Ubuntu server ISO waar men deze heeft opgeslagen.
    - Zeker "Skip Unattended Installation" aanvinken want we zelf nog enkele opties kiezen bij de installatie.
    - Dan kunnen wij best voldoende geheugen kiezen aangezien wij meerdere applicaties op deze box zullen runnen maar in realiteit kan men dit geheugen nog zelf overschrijven in de Vagrantfile. Dit is echter wel handig tijdens het testen van de machine vooraleer wij deze omzetten naar een box.
    - Hetzelfde kan gezegd worden voor CPU, dus kunnen we evengoed voor 2 CPU's gaan tijdens het testen van de machine.
    - En als laatste kunnen we nog onze opslagcapaciteit kiezen, hier kan men best voor een goede hoeveelheid gaan.

2. ## Installatie VM

    Tijdens de installatie zijn er enkele zaken die wij zelf moeten kiezen, voornamelijk kunnen wij met de standaard keuzes gaan maar er zijn er enkele die wij best zelf kiezen namelijk:

    - Natuurlijk de taal, wij kozen voor English.
    - We werden gevraagd of wij de installer wensten te updaten, hievoor kozen wij ja.
    - Dan de toetsenbord layout, hier kozen wij voor Belgian om zo een Azerty-layout te krijgen.
    - Bij de type van installatie kozen wij voor de standaard Ubuntu Server.
    - Bij netwerk connecties hebben wij niets veranderd.
    - Niets ingevuld bij de proxy configuratie.
    - Ook de standaard archive mirror gelaten.
    - Niets veranderd bij de guided storage configuratie.
    - Ook niet bij de storage configuratie.
    - Dan komen we bij de profiel setup, hier kan men al een user aanmaken genaamd vagrant en voor paswoord ook vagrant kiezen. Men kan ook een naam en servernaam kiezen maar dat is minder belangrijk.
    - Niet nodig om naar Ubuntu Pro te upgraden, dus "Skip for now".
    - We kunnen al de OpenSSH server installeren, dit kunnen wij best doen want hier maakt Vagrant gebruik van wanneer wij via het "vagrant ssh" commando met een de box proberen te connecteren.
    - Dan als laatste krijgen we bij de Ubuntu Live server de keuze uit enkele "Featured Server Snaps". Dit zijn packages voor bepaalde veel gebruikte applicaties. Aangezien wij Prometheus als monitoring applicatie gingen gebruiken konden we deze al uit de lijst kiezen zodat de installatie hiervan al in orde was.

3. ## Voorbereiding conversie

    Nu kunnen wij verder met het voorbereiden van de machine zodat deze kan geconverteerd worden naar een bruikbare, werkende box. Hiervoor konden we verder de video volgen:

    - Voor de eerste settings moeten we overschakelen naar de root user met het commando "sudo su -" We zullen voor het paswoord gevraagd worden, dus vagrant.
    - Hij raad aan om de commando's hostnamectl", "timedatectl" en "localectl uit te voeren om na te gaan of de instellingen qua hostnaam, tijd en locatie correct zijn. Deze kunnen nog veranderd worden indien nodig.
    - Vervolgens kijken we best na of het "sudo" package geïnstalleerd is door "apt install sudo" uit te voeren.
    - Dan moeten we een configuratie file aanmaken met gebruik van visudo, de video legt uit dat deze plugin file moet aangemaakt worden in de /etc/sudoers.d/ folder met de naam "vagrant.tmp" (/etc/sudoers.d/vagrant.tmp). Hij gebruikt hiervoor het commando:
        
        ```
        $ visudo -f /etc/sudoers.d/vagrant.tmp
        ```
        Dit bestand bestaat nog niet maar door dit commando openen wij een nieuw bestand met de naam vagrant.tmp met de nano tekst editor. Wij moeten slechts 1 lijn tekst in dit bestand steken en het dan opslaan:
        
        ```
        vagrant ALL=(root) NOPASSWD: ALL
        ```

    - Vervolgens moeten we de vagrant user nog toevoegen aan de sudo groep, dit kunnen we doen met het volgende commando:
        
        ```
         $ usermod -aG sudo vagrant
         ```
    
    - Hierna moeten we terug naar de vagrant gebruiker overschakelen voor het downloaden van de publieke ssh sleutel voor deze gebruiker dit kunnen we doen met het commando "su - vagrant"
    - We zitten dan in de homefolder van de vagrant gebruiker (/home/vagrant) waar we dan de .ssh folder kunnen aanmaken en de publieke sleutel van de vagrant gebruiker kunnen downloaden met de volgende commando's:

        ```
        $ mkdir -m 700 .ssh
        $ wget -O .ssh/authorized_keys https://raw.$ githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub
        $ cat .ssh/authorized_keys
        $ chmod 600 .ssh/authorized_keys
        ```
    - We kunnen dan de historiek van de vagrant gebruiker commando's uitwissen met door het history -c commando uit te voeren en terug overschakelen naar de root user.
    - Nu moeten we over gaan op het installeren van de virtual box guest additions maar hiervoor moeten we eerst nog enkele packages installeren. We installeren de nodige packets met het commando:

        ```
        $ apt install -y dkms linux-headers-$(uname -r) build-essential
        ```
        Wanneer de installatie van deze packages is afgelopen kunnen we de machine afsluiten met het "poweroff" commando.
    - Nu kunnen wij in virtualbox in de settings gaan van deze specifieke machine en bij storage onder de Controller: IDE bij Optical Drive op het cd logo klikken en dan uit het dropdown menu de VBoxGuessAdditions.iso kiezen.
    - Nu moeten we enkel nog de machine terug opstarten om de Guest additions te installeren.
    - Eenmaal opgestart switchen we weer over naar de root user met het commando "sudo su -"
    - We moeten dan de Optical Drive met de Guest additions iso mounten en installeren met de commando's:

        ```
        $ mount /dev/cdrom /mnt
        $ /mnt/VBoxLinuxAdditions.run
        ```
    - Dit is alles wat nodig is om de basis template van de machine aan te maken en we kunnen nu overgaan tot het converteren van deze machine naar het vagrant box formaat.

4. ## Applicaties
    
    Dit is echter niet het moment waarop wij deze machine in box formaat converteren, wij wilden eerst de nodige applicaties installeren op deze machine, in ons geval dus nog de Grafana applicatie zodat wij de gescrapete metrics kunnen visualiseren.

    - Hiervoor starten wij de machine terug op en op de root user voeren we de volgende commando's uit:

        ```
        $ apt-get install -y adduser libfontconfig1 musl
        $ wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.2.3_amd64.deb
        $ dpkg -i grafana-enterprise_10.2.3_amd64.deb
        ```
    - Hierna kunnen we testen of Grafana correct is geïnstalleerd met de volgende commando's:

        ```
        $ systemctl start grafana-server
        $ systemctl status grafana-server
        ```
    Dit is alles wat wij voor de monitoring box moeten voorbereiden aangezien Prometheus bij de installatie van de virtuele machine als is toegevoegd. Wij kunnen de Node Exporter module van Prometheus ook aan deze machine toevoegen zodat de machine zichzelf kan monitoren maar aangezien de installatie van Node Exporter op de slave machine via de Vagrantfile gebeurt (dit omdat we een al bestaande ubuntu box van vagrant cloud gebruiken) hebben we besloten om dit ook zo op de monitor machine te doen.

5. ## Conversie

    De video toont dan hoe we de virtuele machine kunnen converteren naar het vagrant box formaat met het commando:

    ```
    $ vagrant package --base monitor --output monitor.box
    ```
    Mijn vm had de naam monitor dus dit gebruiken wij als de base en de geconverteerde box zal dus monitor.box noemen. Je kan hier eender welke naam kiezen maar het moet wel op .box eindigen. Je kan dit commando van eender waar uitvoeren normaal gezien maar je kan best voor alle zekerheid in de folder gaan waar de virtuele machine zich bevind.

    Hierna heb je een vagrant box die je dan kan uploaden naar je eigen vagrant cloud. Na het inloggen op je account kan een nieuwe box toevoegen er een naam aangeven.

    Hierna krijg je de mogelijkheid om een provider toe te voegen en kies je de volgende opties:

    ```
    Provider: Virtualbox
    File hosting: Upload to Vagrant Cloud
    Box Architecture: unknown
    ```
    Je kan eventueel een hash maken van je vagrant box en dit toevoegen bij checksum maar dit is niet noodzakelijk.

    En nu kan men deze box gebruiken in alle toekomstige Vagrantfile en kunnen wij overgaan tot het aanmaken van de Vagrantfile.