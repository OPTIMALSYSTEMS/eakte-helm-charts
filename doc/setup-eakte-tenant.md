# Tenant einrichten
## Voraussetzung

Momentum Installation inklusive Tenant Management

## Einmalig: Tenant Profil anlegen

Um Tenants mit dem tenant-management erstellen zu können muss zuerst ein Profil angelegt werden.

Ein Profil kann z.B. mit Hilfe der swaggerUi unter `/tenant-management/swagger-ui.html` mit `POST /tenant-management/api/system/profile` hinterlegt werden.

Das Profil ist hier dokumentiert [https://help.optimal-systems.com/yuuvis\_develop/display/YMY/Tenant+Creation+Profile](https://help.optimal-systems.com/yuuvis_develop/display/YMY/Tenant+Creation+Profile).

![](res/img/swagger-profile.png)

**Beispiel Profil**

```json
{
  "user": {
    "withInvitation": false,
    "users": [
      {
        "email": "changeme",
        "firstName": "Tenant",
        "lastName": "Admin",
        "roles": [
          "YUUVIS_TENANT_ADMIN",
          "EAKTE_USER",
          "EAKTE_USER_READ",
          "EAKTE_USER_WRITE",
          "EAKTE_ADMIN"
        ],
        "username": "tenantadmin",
        "password": "changeme",
        "enabled": true,
        "createdTimestamp": 0,
        "temporaryPassword": true
      }
    ]
  },
  "roles": [
    {
      "name": "EAKTE_ADMIN",
      "description": "Enables to be admin for all eakte objects of the tenant"
    },
    {
      "name": "EAKTE_USER",
      "description": "Enables read access to the fileplan and tenant settings"
    },
    {
      "name": "EAKTE_USER_READ",
      "description": "Enables to search and read all permitted eakte objects of the tenant"
    },
    {
      "name": "EAKTE_USER_WRITE",
      "description": "Enables to write all permitted eakte objects of the tenant"
    }
  ],
  "email": {
    "host": "changeme",
    "from": "changeme",
    "username": "changeme",
    "password": "changeme",
    "fromDisplayName": "E-Akte Admin",
    "port": 587,
    "enableAuthentication": true,
    "enableSSL": false,
    "enableStartTLS": false
  },
  "general": {
    "displayNameHTML": "<div class=\"yuv-brand-logo\">${DISPLAY_TENANT_NAME}</div>",
    "defaultLocale": "de",
    "supportedLocales": ["de", "en"]
  }
}
```

## Tenant erstellen

Ein Tenant kann über das tenant-management mithilfe der `POST /tenant-management/api​/system​/tenants` Schnittstelle erstellt werden. Z.B. über die SwaggerUI unter `/tenant-management/swagger-ui.htm`l.

Die einzelnen Optionen und Schnittstellen eines Tenants sind hier dokumentiert: [https://help.optimal-systems.com/yuuvis\_develop/display/YMY/Tenant+Management+Endpoints](https://help.optimal-systems.com/yuuvis_develop/display/YMY/Tenant+Management+Endpoints)

**E-Akte App aktivieren**

Nach der Erstellung des Tenants muss man sich als Nutzer mit `YUUVIS_TENANT_ADMIN` Rolle anmelden und die E-Akte App aktivieren.

Dazu muss die App in den `/api/system/tenants/{{tenant}}/apps` aktiviert werden.

Dies kann mithilfe von Postman geschehen.

**Beispiel App-Konfiguration**

```xml
<?xml version="1.0" encoding="utf-8"?>
<apps xmlns="http://optimal-systems.org/ns/yuuvis/apps/"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://optimal-systems.org/ns/yuuvis/apps/yuuvis-core-apps.xsd">
    <app>
        <name>eakte</name>
        <state>enabled</state>
    </app>
    <app>
        <name>embeddedoffice</name>
        <state>enabled</state>
    </app>
</apps>
```

## Fileplan importieren

Es gibt drei Endpunkte um verschiedene Aktenplanformate zu importieren:

*   /fileplan-importer-api/xlsxToFileplan
*   /fileplan-importer-api/aktenplan21ToFileplan
*   /fileplan-importer-api/aktenplan21ZipToFileplan

Für den Import per xlsx soll folgende Struktur verwendet werden.

[Aktenplan XLSX](res/Aktenplan.xlsx)

Alle Varianten erlauben folgende Request Parameter beim Call des Endpunktes:

* `file`: Der Aktenplan als **xlsx** Datei (xlsxToFileplan), **xml** Datei (aktenplan21ToFileplan) oder **zip** Datei (aktenplan21ZipToFileplan)
* `storageLevel` (optional, Defaultwert: 0): Entscheidet ab welcher Ebene im Baum Akten angelegt werden können. Ebenen beginnen auf der untersten Ebene bei 1. Beispiel:
    *   Allgemeine Verwaltung (Ebene 1)
        *   Landkreis (Ebene 2)
        *   Gemeinde (Ebene 2)
            *   Gemeinderat (Ebene 3)

Bei Wert 0 kann überall abgelegt werden. Wird bspw. 3 als storageLevel gewählt kann erst ab der dritten Ebene abgelegt werden (Gemeinderat im Beispiel oben).

*   `omitLeafs` (optional, Defaultwert: false): Gibt an ob in der letzen Ebene im Baum immer abgelegt werden kann (false) oder nicht (true), egal welches storageLevel gewählt wurde.

Die erste Option (xlsxToFileplan) ist ein von uns vorgegebenes Excel Format. Hier sind zusätzlich folgende Parameter nötig:

*   `sheetName`: Der Name des relevanten Tabellenblatts in Excel
*   `titleColumnName`: Der Name der Spalte in der die Titel der einzelnen Ebenen stehen

  

Der hierbei erstellte Aktenplan kann zum Beispiel über die swagger-ui der E-Akte API hochgeladen werden.

Beispiel: [https://client.con.yuuvis.org/eakte/api/v1/swagger-ui.html](https://client.con.yuuvis.org/eakte/api/v1/swagger-ui.html)

Dort klickt man bei der Schnittstelle POST /fileplan auf Try it out, ergänzt den request body und drückt auf execute.

![](res/img/swagger-fileplan.png)

## Name und Wappen des Mandanten konfigurieren

Es gibt zwei Endpunkte um Namen und Wappen des Mandanten zu konfigurieren. Die Endpunkte können über die swagger-ui der E-Akte-API eingesehen werden `/eakte/api/v1/swagger-ui.html`:

*   `/settings`: Im Body als JSON können Name und der Pfad für das Logo angelegt werden:
    ```json
    {  
      "tenantName": "string",  
      "logoSrcSet": "string"  
    }
    ```
*   `/settings/logo/{filename}:` Logo für den Mandanten im JPG oder PNG Format mit dem angegebenen Dateinamen hochladen

## FAQs
1.  Kann ich mehr als einen Aktenplan in die eAkte importieren?
2.  Kann ich meinen Aktenplan editieren (z.B. Aktenplankennzeichen umbenennen oder neu gliedern)?
3.  Kann ich weitere Aktenplankennzeichen zu meinem Aktenplan hinzufügen?
4.  Kann ich die Ablageebene nach der initialen Einstellung bearbeiten?
5.  Was passiert mit meinen bestehenden Daten, wenn ich einen neuen Aktenplan importiere?

1\. - 4.: Derzeit darf nur ein Aktenplan pro Mandant existieren.  Es ist nur möglich den vorhandenen Aktenplan zu entfernen und mit einem neuen Aktenplan zu ersetzen. Bearbeiten jeglicher Art kann nur am ursprünglichen Dokument des Aktenplans vorgenommen werden, worauf dieses dann an Stelle des vorhandenen Plans hochgeladen werden kann.

5.: Derzeit wird keine Prüfung vorgenommen wenn sich der Aktenplan ändert. Die bestehenden Daten ändern sich also nicht. Dies kann dazu führen, dass vorhandene Akten nicht mehr per Navigation durch den Aktenplan gefunden werden, wenn sich das dazugehörige  Aktenplankennzeichen geändert hat. Über die normale Suche sollen aber alle Objekte weiter gefunden werden.