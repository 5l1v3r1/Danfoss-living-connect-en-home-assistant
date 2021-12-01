# Danfoss-living-connect-en-home-assistant
In dit artikel leg ik uit hoe je deze thermostaten integreert en configureert in HA. Evenals de verschillende manieren om ze te besturen.

# Integratie van de danfoss living connect thermostaat
De integratie van de Zwave Living connect-thermostaat werkt op dezelfde manier als elk ander Zwave-apparaat. Voeg in het Zwave-configuratiemenu een node toe. De thermostaat staat in de integratiemodus zodra de batterijen worden geplaatst.

Eenmaal geïntegreerd, wordt het volgende apparaat verkregen:

![image](https://user-images.githubusercontent.com/46684538/144219126-aa9c9bbc-ccef-4745-9d15-8248b757be35.png)
> Voorbeeld slaapkamer thermostaat via Zwave2mqtt (kan op meerdere manieren)

De enige instelling in de "zwave" interface voor deze thermostaat is het wekinterval. Aan te passen aan uw behoeften, voor kamers waar de temperatuur snel moet worden gewijzigd, is het mogelijk om de waarde in te stellen op 300 (5 minuten), voor kamers waar het niet nodig is om meerdere keren per dag te veranderen, heeft het de voorkeur in te stellen op minimaal 600 (10 minuten) of zelfs 900 (15 minuten) om de batterijen te sparen.

![image](https://user-images.githubusercontent.com/46684538/144219566-00e59cfd-1e0a-4d68-80c1-a02df59ba712.png)

Concreet wordt de temperatuurinstelling uiterlijk na deze wektijd gewijzigd, in dit voorbeeld wordt de wekinterval ingesteld op 900, de wijziging van de gewenste temperatuur vindt uiterlijk na 15 minuten plaats.

Hiermee moet rekening worden gehouden bij het programmeren van de thermostaten.

Om deze thermostaat in de "lovelace" weergave weer te geven, moet u de kaart "thermostaat" kiezen en de betreffende thermostaat selecteren.
We zien direct dat er informatie ontbreekt, de kamertemperatuur stijgt niet, alleen de setpoint wordt weergegeven.
Dit probleem wordt veroorzaakt door de DANFOSS- thermostaat  , die de door de interne sonde gemeten temperatuur niet verhoogt . Maar voordat ik dit probleem verhelp, zal ik mijn thermostaten aanpassen.

Om de parameters van de thermostaataanpassing te beperken (standaard is het mogelijk om van 7 ° C tot 35 ° C in te stellen ), wijzig ik de personalisatieparameters om de instellingen te beperken tussen 10 en 23 ° C.
Geef in uw configuratiebestand, als dit nog niet is gebeurd, uw aanpassingsbestand aan (in dit voorbeeld heet het customize.yaml)

```
homeassistant:
  customize: !include customize.yaml
  packages: !include_dir_named packages
  
## split yaml
automation: !include automations.yaml
group: !include groups.yaml
script: !include scripts.yaml
scene: !include scenes.yaml
timer: !include timers.yaml
switch: !include switch.yaml
sensor: !include sensors.yaml
#binary_sensor: !include binarysensors.yaml
counter: !include counters.yaml
notify: !include notifications.yaml
.....
```

In het bestand customize.yaml voeg ik deze regels toe (te reproduceren voor elke thermostaat)

```
climate.thermostat_slpkm_tyas:
  max_temp: 23
  min_temp: 10
  hvac_modes: ["heat","idle"]
```

Dit heeft tot gevolg dat de instelling van het setpoint tussen 10 en 23°C wordt begrensd. De regel hvac_modes: ["heat", "idle"] , stelt me in staat om compatibiliteit met homekit te garanderen.
Start in het configuratiemenu de aanpassingsbestanden opnieuw om rekening te houden met de nieuwe limieten.

## AppDaemon
Om de temperatuur van de kamer in de thermostaat te verhogen is er een programma ontwikkeld door: Maciej Bieniek.
Met dit programma kunt u informatie ophalen van een temperatuursensor bij de thermostaat die zich in dezelfde kamer bevindt. Dit programma heet: [thermostat update](https://github.com/bieniu/ha-ad-thermostats-update) het moet worden gebruikt met de AppDaemon- plug-in .

AppDaemon is een python, multithreaded runtime-omgeving voor Home Assistant-software. Het biedt ook een configureerbaar dashboard (HADashboard) dat geschikt is voor wandplanken.

Om het te installeren, gaat u gewoon naar het supervisormenu, Addonstore en zoekt u naar de appdaemon 4-plug-in in de community-add-onsectie.

[AppDaemon-documentatie.](https://github.com/hassio-addons/addon-appdaemon/blob/main/appdaemon/DOCS.md)


![image](https://user-images.githubusercontent.com/46684538/144263287-8ada35a4-65de-497c-98c7-cd06d997561d.png)

##Add-on configuratie
In het configuratiegedeelte van de add-on kunt u de standaardwaarden laten staan of indien nodig het logniveau of de poort wijzigen.
![image](https://user-images.githubusercontent.com/46684538/144263385-e189b970-8a2c-4c99-ab99-59d79803c705.png)

Bij het starten van de add-on krijgt u een foutmelding in het logboek waarin staat dat het bestand appdaemon.yaml niet bestaat, waardoor het opstarten wordt geblokkeerd.

Maak gewoon het appdaemon.yaml- bestand in de appdaemon- directory en plak de volgende informatie erin:

```
secrets: /config/secrets.yaml
appdaemon:
  latitude: 52.379189
  longitude: 4.899431
  elevation: 2
  time_zone: Europe/Amsterdam
  plugins:
    HASS:
      type: hass
http:
  url: http://127.0.0.1:5055
hadashboard:
admin:
api:
```
Als het poortnummer is gewijzigd, overweeg dan om dit in het script te corrigeren.
Vul uw GPS-coördinaten in in plaats van het voorbeeldbestand, evenals uw tijdzone en hoogte en controleer het pad van uw geheime bestand.
![image](https://user-images.githubusercontent.com/46684538/144263510-2135cb59-b944-4a37-be2a-7df3bb3cd4fe.png)

HACS installeren
Het thermostaat-updatescript kan direct in de apps-map van appdaemon worden geïnstalleerd, maar ik gebruik liever HACS om te profiteren van tracking-updates en andere mogelijkheden.

#HACS, is een winkel die is opgezet door de gemeenschap van thuisassistenten. In deze Store is het mogelijk om gepersonaliseerde kaarten, thema's, plug-ins, integraties, scripts… .etc… te downloaden.




