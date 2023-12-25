# Creatie van de monitoring box

Bij het creëren van een Vagrant box begint men bij het opzetten van een virtuele machine, hiervoor hadden wij gekozen voor het gebruik van de Ubuntu Server ISO die we hebben gedownload van de officiële [site](https://ubuntu.com/download/server).

De volgende stap was onderzoeken hoe het aanmaken van Vagrant box juist in zijn werking gaat, na wat opzoek werk stuitten we op de volgende zeer behulpzame [video](https://www.youtube.com/watch?v=Wqtlj9osK0g).

In de video legde men uit, dat voor een succesvol werkende box aan te maken wij de volgende zaken moesten doen:

1. We beginnen wij het aanmaken van een virtuele machine in Virtualbox, bij het doorlopen van de creatie wizard is het belangrijk de volgende zaken te doen:

    - Het kiezen van een duidelijke naam, dit hebben wij nodig bij het omzetten van de VM naar een box.
    - Dan het selecteren van Ubuntu server ISO waar men deze heeft opgeslagen.
    - Zeker "Skip Unattended Installation" aanvinken want we zelf nog enkele opties kiezen bij de installatie.
    - Dan kunnen wij best voldoende geheugen kiezen aangezien wij meerdere applicaties op deze box zullen runnen maar in realiteit kan men dit geheugen nog zelf overschrijven in de Vagrantfile. Dit is echter wel handig tijdens het testen van de machine vooraleer wij deze omzetten naar een box.
    - Hetzelfde kan gezegd worden voor CPU, dus kunnen we evengoed voor 2 CPU's gaan tijdens het testen van de machine.
    - En als laatste kunnen we nog onze opslagcapaciteit kiezen, hier kan men best voor een goede hoeveelheid gaan.

2. Tijdens de installatie zijn er enkele zaken die wij zelf moeten kiezen, voornamelijk kunnen wij met de standaard keuzes gaan maar er zijn er enkele die wij best zelf kiezen namelijk:

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

3. Nu kunnen wij verder met het voorbereiden van de machine zodat deze kan geconverteerd worden naar een bruikbare, werkende box. Hiervoor konden we verder de video volgen:

    - Voor de eerste settings moeten we overschakelen naar de root user met het commando "sudo su -" We zullen voor het paswoord gevraagd worden, dus vagrant.
    - Hij raad aan om de "hostnamectl", "timedatectl" en "localectl" uit te voeren om na te gaan of de instellingen qua hostnaam, tijd en locatie correct zijn. Deze kunnen nog veranderd worden indien nodig.
    - Vervolgens kijken we best na of het "sudo" package geïnstalleerd is door "apt install sudo" uit te voeren.
    - Dan moeten we een configuratie file aanmaken met gebruik van visudo, de video legt uit dat deze plugin file moet aangemaakt worden in de /etc/sudoers.d/ folder met de naam "vagrant.tmp" (/etc/sudoers.d/vagrant.tmp). Hij gebruikt hiervoor het commando "visudo -f /etc/sudoers.d/vagrant.tmp".
    Dit bestand bestaat nog niet maar door dit commando openen wij een nieuw bestand met de naam vagrant.tmp met de nano tekst editor. Wij moeten slechts 1 lijn tekst in dit bestand steken:
    
            `vagrant ALL=(root) NOPASSWD: ALL`
    - 