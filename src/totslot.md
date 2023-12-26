# Tot slot

Nog even wat over de folder opbouw. Heel deze opdracht is opgeladen op Github.
Wanneer we deze repository pullen krijgen we 3 mappen:
- De .github folder met de workflow voor het aanmaken van de statische documentatie website die u hier ziet.
- De configs folder waarin de te kopïeren bestanden staan, deze folder syncen wij met de machines die we opstarten met de Vagrantfile.
- En de src folder, hierin staan de markdown bestanden waaruit deze site wordt opgemaakt.

Dan in de root van de repository staat de Vagrantfile, een diagram in draw.io over onze opzet en de capture van het scrapen dat door prometheus wordt uitgevoerd.

Eenmaal deze repository gepulled is kunnen wij in de NSM folder gaan en het volgende commando uitvoeren:

```
vagrant up
```

Hierna zullen alle machines in de Vagrantfile opstarten en alle commando's worden uitgevoerd.

Eenmaal de volledige file doorlopen, zijn er dus drie machines lopende, een monitoring machine die de twee slave machines scraped/monitored.

We kunnen dan de volgende diensten van de monitor machine via onze lokale webbrowser bereiken:
- Prometheus op localhost:9090
- Node Exporter op localhost:9100
- Grafana op localhost:3000

Als we de Prometheus interface bezoeken zal men onder targets zien dat deze zowel zichzelf als de twee slave machines aan het scrapen/monitoren is.

Indien men de gescrapte metrics wil visualiseren kan men naar Grafana gaan, en deze instellen met als data source Prometheus na welke men de verschillende gescrapte metrics kan visualiseren.