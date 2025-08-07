# E-Akte Helm Charts

## Abhängigkeiten

- [Helm](https://helm.sh/docs/intro/install/)
- [Kubernetes](https://kubernetes.io/docs/setup/)
- Yuuvis Momentum in der Version 2023winter:
- Core Services Stand 4.17.13, Repository Stand 4.17.14
- Client: 
    - Api-Web 1.17.13
    - Tenant management 1.14.2
    - Userservice 1.13.2
    - Viewerservice 3.0.10
    - Bpm-engine:0.13.2
    - Bpm-admin:0.9.1
    - Renditionservice:2.8.0
    - Rendition-Repository:2.0.0

## E-Akte Installation

1. [E-Akte App installieren](doc/setup-eakte-app.md)
2. [Tenant erstellen](doc/setup-eakte-tenant.md)

Nach der Installation kann die API der E-Akte unter folgenden Pfad aufgerufen werden: /eakte/api/v1/swagger-ui.html.
Somit kann die E-Akte problemlos erweitert und automatisiert werden.
