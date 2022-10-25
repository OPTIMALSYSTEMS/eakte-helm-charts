# E-Akte Momentum App installieren
## Vorbedingungen

Achtung

Keycloak & das customized Template als Erstes (vor Yuuvis Infrastruktur) installieren

## E-Akte App in Momentum installieren

**Konfiguration und Service Account für die E-Akte anlegen**

Einen Crosstenant User-Account einrichten im Master mit Rolle `EAKTE_ADMIN`:

[https://help.optimal-systems.com/yuuvis\_develop/display/YMY/Cross-Tenant+Service+Accounts](https://help.optimal-systems.com/yuuvis_develop/display/YMY/Cross-Tenant+Service+Accounts)

Zusätzlich muss folgender Service mit `kubectl apply -n yuuvis -f <path>` bereitgestellt werden

[authentication-internal.yaml](res/authentication-internal.yaml)

Das Konfigurations git (gitea) öffnen und eine leere application-eakte.yml unter yuuvis-config im Masterbranch anlegen und den im Crosstenant Schritt erstellten User unter service-user eintragen.

Neustarten des configservice im yuuvis namespace, damit dieser den aktuellen Config Stand mit dem Git synchronisiert.
`kubectl -n yuuvis rollout restart sts configservice`

```yml
momentum:
  service-user:
    name: service_user
    password: Serv1ceUser1!
    tenant: servicestenant
eakte:
  db:
    password: changeme
```

In der `authentication-prod.yaml`

Die eakte unter routing.endpoints hinzufügen

Zu `authorization.accesses` folgendes hinzufügen

`- endpoints: /eakte/**`

Bei `routing.defaultEntryPoint` /eakte/index.html setzen

configservice, authentication und api neustarten.
```bash
kubectl -n yuuvis rollout restart sts configservice
kubectl -n yuuvis rollout restart deployment authentication
kubectl -n yuuvis rollout restart deployment api
```

## E-Akte Helm Charts installieren

Git auschecken:

[https://github.com/OPTIMALSYSTEMS/eakte-helm-charts](https://github.com/OPTIMALSYSTEMS/eakte-helm-charts)

Helm-Repos hinzufügen:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Helm Dependencies updaten
```bash
cd eakte
helm dep up
cd ..
```

Beispiel Values File mit den wichtigsten Variablen, die im Helm Chart konfiguriert werden müssen:
[eakte-values.yaml](res/eakte-values.yaml)

E-Akte Values File anpassen changeme durch Passwort ersetzen

Bei accessKeyId und secretAccessKey die von OS vergebenen Credentials für den Feedback Service eintragen.

Bei imageCredentials müssen die von OS vergebenen Zugangsdaten zur Docker Registry eingetragen werden.

Bei yuuvis.systemAdmin muss ein Nutzer mit einer `YUUVIS_SYSTEM_INTEGRATOR` Rolle eingetragen werden, mit welchem die E-Akte App zu Momentum hinzugefügt wird.

E-Akte Namespace anlegen und helm Charts installieren 

`helm install eakte ./eakte -n eakte --create-namespace -f eakte-values.yaml`

## E-Akte pro Tenant konfigurieren

[Dokumentation für den Setup eines Tenants in der eAkte](setup-eakte-tenant.md)

[eAkte Keycloak konfigurieren](setup-eakte-keycloak.md)

## Yuuvis Viewer verwenden

Aktuell, wenn OnlyOffice mit Embedded Office Connector nicht benutzt wird, wird der Yuuvis Viewer als Default, für alle Datenformate, im gesamten Cluster gesetzt. Dies kann in der `application-eakte.yaml` konfiguriert werden. Eine tenant-spezifische Konfiguration ist möglich beim Setup des Tenants.

## Embedded Office installieren

Embedded Office nach folgender Anleitung installieren
und darauf achten OnlyOffice und Dependencies wirklich in den namespace onlyoffice mit `-n onlyoffice` zu installieren.

[https://gitlab.ecmind.ch/open/embeddedofficeservice\_momentum\_helm/-/wikis/home](https://gitlab.ecmind.ch/open/embeddedofficeservice_momentum_helm/-/wikis/home)

Folgendes Roleset in die embeddedoffice App installieren

[office-roles.xml](res/office-roles.xml)

Den OnlyOffice documentserver dabei mit folgenden Parametern konfigurieren

`helm install -n onlyoffice documentserver onlyoffice/docs --set securityContext.enabled=true --set persistence.size=1Gi --set jwt.secret=changeme`

Im Konfigurations git in der Datei application-eakte.yaml folgendes hinzufügen:

```yml
eakte:
  useOnlyOffice: true
```