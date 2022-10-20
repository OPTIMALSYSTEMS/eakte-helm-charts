# 1. eAkte App, EmbeddedOffice & OnlyOffice installieren

## Inhaltsverzeichnis

```
InhaltsverzeichnisVorbedingungen
E-Akte App in Momentum installieren
E-Akte Helm Charts installierenE-Akte pro Tenant konfigurieren
Yuuvis Viewer konfigurieren
```
## Vorbedingungen

```
Achtung
Keycloak & das customized Template als erstes (vor Yuuvis Infrastruktur) installieren
```
# E-Akte App in Momentum installieren

```
Postman installieren https://www.postman.com/downloads/
E-Akte-Collection importieren
Ein Postman Environment erstellen und dort folgende Variablen setzen
Variable Beschreibung
username Nutzer mit YUUVIS_SYSTEM_INTEGRATOR Rechten
password Passwort des Nutzers
tenant Mandant mit dem gewählten Nutzer
protocol Protokoll unter welchen yuuvis zur Verfügung steht. z.B. https
address Adresse unter welcher die E-Akte verfügbar ist. z.B. yuuvis.org
Das E-Akte Schema auschecken
https://git.optimal-systems.org/redline-consulting/saas-produkte/eakte/eakte-schema.git
Als Nutzer mit einer muss in dem Body die entsprechende Datei gesetzt werden.YUUVIS_SYSTEM_INTEGRATOR Rolle einloggen und die E-Akte mittels der Postman Requests installieren. Bei jedem Request
```
```
Bei der eakte-systemhooks.json muss changeme durch das amqp/rabbitmq Passwort ersetzt werden.
Einen Crosstenant User-Account einrichten im Master mit Rolle EAKTE_ADMIN:
https://help.optimal-systems.com/yuuvis_develop/display/YMY/Cross-Tenant+Service+Accounts
Zusätzlich muss folgender Service mit kubectl apply -n yuuvis -f <path> bereitgestellt werden
```
```
Das Konfigurations git (gitea) öffnen und eine leere application-eakte.yml unter yuuvis-config im Masterbranch anlegen.
```

```
momentum:
service-user:
name: service_user
password: Serv1ceUser1!
tenant: servicestenant
eakte:
db:
password: changeme
```
In der authentication-prod.yaml
Die eakte unter routing.endpoints hinzufügen
Zu authorization.accesses folgendes hinzufügen

- endpoints: /eakte/**
Bei routing.defaultEntryPoint /eakte/index.html setzen
configservice, authentication und api neustarten.
Die Datei mit folgenden Inhalt füllen.
Den im Crosstenant Schritt erstellten User unter service-user eintragen.
Neustarten von configservice im yuuvis namespace

### E-Akte Helm Charts installieren

Git auschecken:
https://git.optimal-systems.org/redline-consulting/saas-produkte/eakte/eakte-helm-charts

E-Akte Values File anpassen changeme durch Passwort ersetzen
Bei awsAccessKeyId und awsSecretAccessKey die von OS vergebenen Credentials für den Feedback Service hinterlassen.l
Bei imageCredentials müssen die von OS vergebenen Zugangsdaten zur Docker Registry eingetragen werden.

E-Akte Namespace und helm Charts installieren anlegen
kubectl create ns eakte
helm install eakte ./eakte -n eakte -f eakte-values.yaml

### E-Akte pro Tenant konfigurieren

EMBEDDED_OFFICE Rolle erstellen und im Keycloak als default Rolle setzen
Aktenplanformat konfigurieren & Aktenplan importieren

### Yuuvis Viewer konfigurieren

Aktuell, wenn Only Office nicht benutzt wird, wird der Yuuvis Viewer als Default für den gesamten Cluster gesetzt. In Zukunft wird eine tenant-
spezifische Konfiguration angeboten


## Embedded Office installieren

Nach folgender Anleitung installieren
und darauf achten onlyoffice und dependencies wirklich in den namespace onlyoffice mit -n onlyoffice zu installieren.
https://gitlab.ecmind.ch/open/embeddedofficeservice_momentum_helm/-/wikis/home
Folgendes Roleset in die embeddedoffice App installieren

Den OnlyOffice documentserver dabei mit folgenden Parametern konfigurieren
helm install -n onlyoffice documentserver onlyoffice/docs --set securityContext.enabled=true --set persistence.
size=1Gi --set jwt.secret=changeme
Im Konfigurations git in der Datei application-eakte.yaml folgendes hinzufügen:

```
eakte:
useOnlyOffice: true
```

