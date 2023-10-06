# Configuration As Code voor Organisaties(Teams)

Om gebruik te maken van de mogelijkheid om Ansible Automation Controller via  
code te configureren, is het volgende noodzakelijk:  

- Een ingerichte git repository om de configuratie op te slaan  
- Een pipeline die de configuratie verwerkt  
- De kennis van de configuratie in dit document  

## Git repository

De git repository wordt geleverd comform de standaard voor de organisatie.  
In deze repository worden de volgende bestanden en directory structuur opgeleverd:  

```
.
├── collections
│   └── requirements.yml
├── group_vars
│   ├── accp
│   │   ├── credentials.yaml
│   │   ├── inventory.yaml
│   │   └── templates.yaml
│   ├── all
│   │   ├── credentials.yaml
│   │   ├── inventory.yaml
│   │   ├── templates.yaml
│   │   ├── organization.yaml
│   │   ├── projects.yaml
│   │   ├── roles.yaml
│   │   ├── teams.yaml
│   │   └── users.yaml
│   └── dev
│       ├── credentials.yaml
│       ├── inventory.yaml
│       └── templates.yaml
├── host_vars
│   ├── controller_accp
│   │   └── controller_auth.yaml
│   └── controller_dev
│       └── controller_auth.yaml
├── inventory.yaml
├── main.yml
└── README.md
```

Er zijn een aantal bestanden die voor de pipeline belangrijk zijn en **NIET** gewijzigd mogen worden.  
De bestanden die voor de gebruiker van de organisatie van belang zijn, staan onder de directory  
/group_vars/  
Waar NEW_ORG de teamnaam zal zijn voor de organisatie in Ansible Automation Platform.  
Alle overige bestanden zijn van belang voor de juiste werking van Configuration As Code en mogen niet  
worden gewijzigd/verwijderd!  

## De pipeline

De pipeline zorgt ervoor dat de wijzigingen die in GIT worden gedaan (op de branch voor de omgeving) worden  
toegepast op de Automation controller van het platform in de juiste omgeving. Hiervoor wordt standaard  
scripting gebruikt die door de opensource community is ontwikkeld.  
De aangemaakte configuratie in GIT wordt toegevoegd aan de controller, bestaande items die in de controller al  
bestaan en niet in de git zijn opgenomen, worden **niet** verwijderd.  
Echter bij een volledige herinstallatie, zullen deze niet meer worden aangemaakt. Zorg er dus voor dat **alles**  
in code is opgenomen, zodat het ook na een herinstallatie weer terugkomt.  

[Configuratie beschrijving](config_cac.md)

