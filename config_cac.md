## De configuratie bestanden

Om een organisatie correct te vullen met objecten, zijn de volgende bestanden   
noodzakelijk:

```
credentials.yaml
organization.yaml
projects.yaml
inventory.yaml
schedules.yaml
templates.yaml
teams.yaml
roles.yaml
```

We zullen aan de hand van deze bestanden een nieuwe organisatie in de controller gaan configureren.  
Per bestand zullen we aangeven welke opties van belang zijn en hoe deze gevuld dienen te worden,  
zodat er een werkende configuratie ontstaat in AAP. 
Houd er rekening mee dat object namen in AAP uniek moeten zijn, dus we gaan er van uit dat de namen  
voorzien worden van de organisatie naam als voorvoegsel.  

In elk bestand komen we op de 2e regel de controller_{item}_{omgeving} tegen:

Dit geeft aan welk item hier wordt geconfigureerd en voor welke omgeving dit zal gelden.  
- item      het hieronder uitgelegde deel van de configuratie
- omgeving  voor welke omgeving(en) geldt dit item

De omgeving wordt aangegeven door de naam van de map in de group_vars directory.  

**Stelling:** 

We hebben een nieuw team(NEW_ORG) dat ansible code heeft ontwikkeld en dat via AAP moet toepassen op een omgeving  
met een aantal machines. De code staat in git en de lijst met systemen waarop deze moet worden toegepast  
is bijgehouden in een separate git repository (inventory).
We gaan deze configuratie nu laden in AAP vanuit CaC (configuration as code).

### credentials.yaml

Credentials (toegangstokens) zijn essentieel voor de werking van AAP, ze voorzien de automation van de juiste toegang  
tot systemen om code te kunnen ophalen en uitvoeren.

credentials kunnen heel divers zijn, we zullen er aantal hier benoemen en configureren, die voor de basiswerking van  
belang zijn.

```yaml
---
controller_credentials_xx:

  - name: 
    description:
    credential_type: 
    organization: 
    inputs:
      auth_url: 
      token: 
      url:
...
```
Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_credentials_ | Een van de opties: co, ct, ca, cp of all |
| name: | Naam van de credential met de organisatie naam ervoor |
| description: | Omschrijving van de credential |
| credential_type: | Het type credential die wordt gebruikt voor de credential |
| organization: | De organisatie waarvoor de credential geldt |
| inputs: | De velden die noodzakelijk zijn om de credential vast te leggen |

Als we deze gaan invullen voor de nieuwe organisatie, volgt daaruit de volgende, mogelijke 
configuratie:

Als eerste de credentials die voor alle omgevingen gelden:
```yaml
---
controller_credentials_all:

  - name: NEW_ORG_automation_hub_token_published
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: NEW_ORG
    inputs:
      auth_url: ''
      token: !vault $ANSIBLE_VAULT;1.1;AES256-- token --
      url: hub url naar published repo's

  - name: NEW_ORG_automation_hub_token_rh_certified
    description: 
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: NEW_ORG
    inputs:
      auth_url: ''
      token: !vault $ANSIBLE_VAULT;1.1;AES256-- token --
      url: hub url naar rh_certified

  - name: NEW_ORG_git
    description: 
    credential_type: Source Control
    organization: NEW_ORG
    inputs:
      ssh_key_data: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        -----BEGIN OPENSSH PRIVATE KEY-----
             -- key data --
        -----END OPENSSH PRIVATE KEY-----
      username: AAP_user
...
```
Als 2e de credentials die alleen voor de co(ontwikkel) omgeving gelden:
```yaml
---
controller_credentials_co:


  - name: NEW_ORG_ansible
    description: 
    credential_type: Machine
    organization: NEW_ORG
    inputs:
      become_method: sudo
      become_username: ''
      ssh_key_data: !vault |
        $ANSIBLE_VAULT;1.1;AES256
        -----BEGIN OPENSSH PRIVATE KEY-----
             -- key data --
        -----END OPENSSH PRIVATE KEY-----
      username: ansible
...
```
We zien in het bovenstaande, dat de credentials voor de automationhub in alle omgevingen  
gelijk zulen zijn, maar dat in de ontwikkel omgeving een andere ansible key gebruikt gaat  
worden voor de machines. Zo kunnen we borgen dat de omgevingen elkaar niet kunnen beïnvloeden.
Door in elke map een andere sleutel op te nemen voor het ansible account, is scheiding mogelijk.  

### organization.yaml

In dit bestand is de definitie van het team ( organisatie binnen AAP) opgenomen. Dit bestand is eenvoudig te configureren  
aangezien er niet veel in staat.

```yaml
---
controller_organizations_xx:
  - name:
    description: 
    galaxy_credentials:
    assign_galaxy_credentials_to_org: 
...
``` 
Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_organizations_ | Een van de opties: co, ct, ca, cp of all |
| name: | Naam van de organisatie |
| description: | Omschrijving van de organisatie |
| galaxy_credentials: | De credentials die aan de  organisatie moeten worden togevoegd |
| assign_galaxy_credentials_to_org: | De credentials aan de organisatie toewijzen |

Als we deze gaan invullen voor de nieuwe organisatie, dan volgt daaruit het volgende bestand:

```yaml
---
controller_organizations_all:
  - name: NEW_ORG
    description: Nieuw team in AAP
    galaxy_credentials:
      - NEW_ORG_automation_hub_token_rh_certified
      - NEW_ORG_automation_hub_token_published
    assign_galaxy_credentials_to_org: true
...
```
Door bij de credentials de namen van de credentials gelijk te houden, hoeft hier niet per  
omgeving te worden afgeweken en kan worden volstaan met een "all" configuratie.   


### projects.yaml
In dit bestand leggen we de koppeling naar de git repositories aan in AAP. Deze koppelingen definieren de projecten  
waarop templates en inventories worden gebaseerd, om code te kunnen uitvoeren in AAP.

```yaml
---
controller_projects_xx:
  - name: 
    description: 
    organization: 
    scm_type: 
    scm_url: 
    scm_credential: 
    scm_branch: 
    scm_clean: 
    scm_delete_on_update: 
    scm_update_on_launch: 
    scm_update_cache_timeout: 
    allow_override: 
    timeout: 
```
Zoals we kunnen zien hierboven kunnen zien zijn er een flink aantal parameters te vullen, de meeste zijn eenvoudig  
op een default in te stellen, die we dan ook hier eenmalig aangeven.

Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_projects_ | Een van de opties: co, ct, ca, cp of all |
| name: | Naam van het project, te beginnen met de organisatienaam | 
| description: | Omschrijving van het project vrij in te vullen |
| organization: | De naam van de organisatie |
| scm_type: | Als dit een git repository is, moet hier "git" staan |
| scm_url: | De ssh url voor de repository, zoals je deze van de repo kan kopieren |
| scm_credential: | De naam van de voor de organisatie aangemaakte credential bv: NEW_ORG_git |
| scm_branch: | Meestal zal dit "main" zijn, maar kan ook een andere brach zijn |
| scm_clean: | Staat meestal op false |
| scm_delete_on_update: | Staat meestal op false |
| scm_update_on_launch: | Staat meestal op true |
| scm_update_cache_timeout: | Staat meestal op 0 |
| allow_override: | Staat meestal op false |
| timeout: | Staat meestal op 0 |
 
Als we deze gaan invullen, ontstaat het volgende bestand waarin zowel een inventory opgenomen is, als een code inventory:  
Hiervan zullen we er meestal 2 nodig hebben, een in de all met de meeste projecten en een afzonderlijke in   
de omgeving voor de inventory( die voor elke omgeving anders zal zijn).  

In de all staan alle code projecten:
```yaml
---
controller_projects_all:

  - name: NEW_ORG code install packages
    description: Installatie van nodige packages op linux
    organization: NEW_ORG
    scm_type: git
    scm_url: git@server....code_package_install.git
    scm_credential: NEW_ORG_git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: false
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0
...
```
In de omgevingen zulen de inventory projecten staan, per omgeving:

```yaml
---
controller_projects_co:

  - name: NEW_ORG inventory linux
    description: inventory project voor linux systemen
    organization: NEW_ORG
    scm_type: git
    scm_url: git@server.....git
    scm_credential: NEW_ORG_git
    scm_branch: master
    scm_clean: false
    scm_delete_on_update: false
    scm_update_on_launch: true
    scm_update_cache_timeout: 0
    allow_override: false
    timeout: 0
...
```

We zien hier dat er nu 2 projecten zullen worden aangemaakt binnen de organisatie NEW_ORG, een inventory  
en een code project waarin ansible code tbv. systeem configuratie is opgeslagen.

### inventory.yaml

Een inventory in AAP is en verzameling variabelen tbv machines die via ansible geconfigureerd moeten worden.  
Het bestand heeft de volgende inhoud, die zal moeten worden aangepast:

```yaml
---
controller_inventories_xx:
  - name: 
    description:
    organization:

controller_inventory_sources_xx:
  - name: 
    description: 
    organization:
    source:
    source_project:
    source_path:
    inventory:
    update_on_launch:
    overwrite:
...
```

We gaan stap- voor stap door het bestand en vullen de diverse parameters in om een werkende inventory toe te voegen.

Het inventory.yaml bestand heeft 2 secties:
- controller_inventories_xx
- controller_inventory_sources_xx

De inventory is het deel dat in de controller zichtbaar wordt en dat te gebruiken is in playbooks, de inventory_sources  
is het deel dat de vulling van de inventory verzorgd, dat kunnen er meerdere zijn per inventory, we gaan hier uit van  
1 source per inventory.

#### controller_inventories_xx

Hieronder de eerste sectie die we gaan vullen:  
Er staan 3 parameters die ervoor zorgen datr de inventory wordt aangemaakt.  
```yaml
controller_inventories_xx:
  - name: 
    description: 
    organization: 
```

Name: De naam in controller voor de inventory, door altijd als eerste de naam van de organisatie op te geven, houden we de zaken uniek. 
Description: De beschrijving van de inventory, deze wordt met name belangrijk als er meerdere inventories worden aangemaakt.
organization: De naam van de organisatie waarvoor de inventory wordt gemaakt

Als we deze gaan vullen, ontstaat de volgende sectie:
```yaml
controller_inventories_all:
  - name: NEW_ORG inventory
    description: Linux inventory
    organization: NEW_ORG
```

We zien dat we waar de namen aangemaakt moeten worden, we de naam van het team gebruiken als voorvoegsel, dit moeten we doen  
om de naamgeving uniek te houden binnen de hele AAP configuratie. Dor dit te doen voorkomen we naamgevings conflicten als we  
veel teams gaan ontsluiten in AAP.

#### controller_inventory_sources_xx
Dan de 2e sectie binnen het bestand:

Voor een inventory die gebaseerd is op een git repository, dient er verwezen te worden naar een project.  
De koppeling naar dat project vullen we hier in:

```yaml
controller_inventory_sources_xx:
  - name:
    description:
    organization:
    source:
    source_project:
    source_path:
    inventory:
    update_on_launch:
    overwrite:
...
```
Door de inventory projectnamen in alle omgevingen gelijk te houden, kunnen we hier volstaan met een configuratie in _all.  

```yaml
controller_inventory_sources_all:
  - name: New_ORG inventory
    description: Inventory
    organization: NEW_ORG
    source: scm
    source_project: NEW_ORG inventory linux 
    source_path: hosts.yml
    inventory: NEW_ORG inventory
    update_on_launch: true
    overwrite: true
...
```


### templates.yaml

Templates zijn de uitvoerbare delen van de configuratie binnen AAP.  
Via templates kunnen we code opgeslagen in git, tegen hosts runnen die ook zijn opgeslagen in git,  
dit noemen we dan inventories.
Hier komen de voorgaande configuraties bij elkaar tot een uitvoerbare eenheid.

```yaml
---
controller_templates_xx:

  - name: 
    description: 
    organization: 
    project: 
    inventory: 
    playbook: 
    job_type: 
    fact_caching_enabled: 
    credentials:
    concurrent_jobs_enabled: 
    ask_scm_branch_on_launch: 
    ask_tags_on_launch: 
    ask_verbosity_on_launch: 
    ask_variables_on_launch: 
    extra_vars:
    execution_environment: 
    survey_enabled: 
    survey_spec: {}
...
```

Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_templates_ | Een van de opties: co, ct, ca, cp of all |
| name: | Naam van de template, te beginnen met de organisatienaam |
| description: | Omschrijving van de template, vrij in te vullen |
| organization: | De naam van de organisatie |
| project: | De naam van het project, zoals deze is opgenomen in projects.yaml |
| inventory: | De naam van de inventory, zoals deze is opgenomen in inventory.yaml |
| playbook: | Het playbook in het git project dat moet worden uitgevoerd |
| job_type: | Een van 2 opties: Run of Check, Run is uitvoeren en Check is alleen controleren |
| fact_caching_enabled: | Moeten gevonden facts in de cache worden opgenomen? |
| credentials: | De te gebruiken credentials voor deze taak |
| concurrent_job_enabled: | Mogen er meerdere taken naast ekaar worden gestart | 
| ask_scm_branch_on_launch: | Moet er om de branch die moet worden gebruikt gevraagd worden |
| ask_tags_on_launch: | Moet er om tags worden gevraagd voordat de taak wordt gestart |
| ask_verbosity_on_launch: | Moet er om het loglevel worden gevraagd voor uitvoering |
| ask_variables_on_launch: | Moeten er variabelen worden ingevuld |
| extra_vars: | Voeg extra variabelen toe aan de defitie van de taak |
| execution_environment: | Welke execution environment dient gebruit te worden |
| survey_enabled: | Is er een vragenlijst, en moet deze worden getoond |
| survey_spec: | {} invulling van een eventuele vragenlijst. |

Als we deze gaan invullen voor het project dat we gemaakt hebben, dan volgt het volgende bestand:

```yaml
---
controller_templates_all:

  - name: NEW_ORG install packages
    description: Install packages on servers
    organization: NEW_ORG
    project: NEW_ORG code install packages
    inventory: NEW_ORG inventory linux
    playbook: main.yml
    job_type: run
    fact_caching_enabled: false
    credentials:
      - NEW_ORG_ansible
    concurrent_jobs_enabled: false
    ask_scm_branch_on_launch: false
    ask_tags_on_launch: false
    ask_verbosity_on_launch: false
    ask_variables_on_launch: false
    extra_vars:
      hosts: all
    execution_environment: Default execution environment
    survey_enabled: false
    survey_spec: {}
```

Met als gevolg dat als deze via de pipeline naar AAP wordt gezonden, dat de template zal worden aangemaakt.  
Echter deze template zal niet voor iedereen in de organisatie zichtbaar/uitvoerbaar zijn, darvoor missen we  
nog een klein stukje configuratie, met name rechten, daar komen we verderop op terug.

#### Surveys
Surveys verdienen wat extra aandacht omdat deze nogal lastig zijn om in de code op te nemen. Om een survey toe te  
voegen aan je job template, moet ten eerste de sleutel "survey_enabled" op "true" worden gezet, anders is er wel een survey  
maar zul je hem nooit zien.
**Survey's zullen vaak per omgeving verschillen! De templates zullen dan ook per omgeving moeten worden aangemaakt**

Een survey wordt opgegeven als een dict object, die als een dict onder de survey_spec wordt opgenomen. De specificatie van de  
op te geven opties en items zie je hieronder:
Een survey regel bestaat uit de volgende sleutels:
|  sleutel | omschrijving |
| --- | --- |
| name: | De naam voor de survey |
| description: |  De beschrijving van de survey |
| spec: | De specificatie van de survey komt hierna |

De meeste van deze velden kunnen '' (leeg) blijven, behalve de spec, daar moeten de vragen in staan, die in de survey moeten  
worden gesteld.
Een survey regel heeft dus in de basis de volgende structuur:

```yaml
survey_spec:
  - name: ''
    description: ''
    spec:
```
Oder het "spec" veld moeten de vragen van de survey worden op genomen, elke survey vraag heeft ook een vastgelegd format:

| key | value |
| --- | --- |
| question_name: | De te stellen vraag aan de gebruiker |
| question_description: | omschrijving voor de vraag |
| required: | Is een antwoord verplicht "true" of "false" |
| type: | Het type vraag, dat kan een van de volgende optie(s) zijn: |
| | text |
| | textarea |
| | password |
| | multiplechoice |
| | multiselect |
| | integer |
| | float |
| variable: | De variabele waarin het antwoord wodt teruggegeven aan het playbook |
| min: | Het minimaal aantal verwachte tekens |
| max: | Het maximaal aantal tekens |
| default: | welke antwoord wordt alvast ingevuld |
| choices: | De lijst met keuzes in geval van een meerkeuze vraag, met format ['item1','item2'] | 
| new_question: | Deze staat op "true" als er geen volgende vraag meer is en "false" als er nog een vraag volgt |

**voorbeeld 1:**
Als er slechts 1 vraag wordt gesteld is de syntax van een survey spec als volgt:

```yaml
survey_spec:
  - name: ''
    description: ''
    spec:
      - question_name: Geef een waarde
        question_description: ''
        required: true
        type: integer
        variable: var_getal
        min: 0
        max: 1024
        default: ''
        choices: ''
        new_question: true
```

**Voorbeeld 2**
Als een survey meerdere vragen moet stellen, voeg dan gewoon de extra vergen als volgt toe:

```yaml
survey_spec:
  - name: ''
    description: ''
    spec:
      - question_name: Geef een waarde
        question_description: ''
        required: true
        type: integer
        variable: var_getal
        min: 0
        max: 1024
        default: ''
        choices: ''
        new_question: false
      - question_name: Geef een tweede waarde
        question_description: ''
        required: true
        type: multiplechoice
        variable: var_keuze
        min: 0
        max: 1024
        default: ''
        choices:
          - keuze1
          - keuze2
          - keuze3
        new_question: true

```

### teams.yaml
De configuratie tooling via de pipeline biedt de mogelijkheid om teams toe te voegen, echter als er
gebruik wordt gemaakt van LDAP of AD, is hier geen configuratie noodzakelijk.
Voor de volledigheid wordt deze optie wel in de documentatie opgenomen.


```yaml
---
controller_teams_xx:
  - name: 
    description: 
    organization:
...
```
Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_teams_ | Een van de opties: co, ct, ca, cp of all |
| name: | Naam van team, vooraf gegaan door de organisatienaam |
| description: | Omschrijving van het team |
| organization: | De organisatie waar het team onder valt |

Gaan we volgens de bovenstaande regels teams toevoegen, dan volgt het volgende bestand:

```yaml
---
controller_teams_all:
  - name: NEW_ORG developers
    description: devel users
    organization: NEW_ORG
  - name: NEW_ORG admins
    description: admin users
    organization: NEW_ORG
...
```

### roles.yaml

De gebruikers en teams zijn bekend, nu is het tijd om de diverse gebruikers rechten te gaan geven  
om zaken te zien, uitvoeren of zelfs te wijzigen. Dat wordt gedaan via de roles, die elk team krijgt  
toegewezen.
**Dit is het belangrijkste deel van de configuratie.**  
In de roles.yaml komen alle eerdere stappen bij elkaar, hier worden rechten aan gebruikers groepen  
gekoppeld, zonder deze koppeling, zie je als gebruiker in de aangemaakte organisatie niets.


```yaml
---
controller_roles_xx:
  - team: 
    job_templates: 
    role: 

  - team: 
    credentials:
    role: 
      
  - team: 
    inventory:
    role: 

  - team: 
    projects:
    role: 
...

```
Hieronder de lijst met parameters en de noodzakelijke vulling ervan:

| parameter | Omschrijving |
| --- | --- |
| controller_roles_ | Een van de opties: co, ct, ca, cp of all |
| team: | De teamnaam |
| Job_template: | De namen van de job templates waar het team rechten op heeft |
| credentials: | De namen van de credentials waar het team rechten op krijgt |
| projects: | De namen van de projecten waar het team rechten op krijgt |
| inventory: | De namen van de inventories waar het team rechten op krijgt |
| role: | De rol die het team krijgt op de objecten, zie de volgende lijst |

Rechten worden gegeven door de role parameter te vullen met een van de volgende waardes:

| role | omschrijving |
| --- | --- |
| read | Alleen lees rechten, mag het wel zien, maar niet uitvoeren/aanpassen |
| use | Het team mag het gebruiken om uit te voeren en lezen |
| execute | Het team mag het uitvoeren (job templates) |
| adhoc | Het team mag de inventory gebruiken voor ad Hoc commando's |
| update | Het team mag de inventory updaten (in AAP, niet gebruiken!) |
| admin | Alle rechten om uit te voeren of aan te passen |


Gaan we volgens de bovenstaande regels rechten toekennen aan teams, dan volgt het volgende bestand:

Aangezien bij RIVM een AD configuratie wordt gebruikt,  zullen de "user" sleutels ontbreken.  
Bij "team" dient dan de AD groupsnaam te worden ingevuld, waarna het toekennen van de rechten ook  
zal werken, zie het voorlbeeld hieronder:

```yaml
---
controller_roles_all:

  - team: L-LDAP-DEV
    job_templates:
      - NEW_ORG install packages
    role: execute

  - team: L-LDAP-DEV
    credentials:
      - NEW_ORG_git
      - NEW_ORG_ansible
      - NEW_ORG_automation_hub_token_rh_certified
      - NEW_ORG_automation_hub_token_published
    role: use

  - team: L-LDAP-DEV
    inventories:
      - NEW_ORG inventory
    role: use

  - team: L-LDAP-DEV
    projects:
      - NEW_ORG NEW_ORG code install packages
    role: use
...
```

De L-LDAP-DEV is de groepnaam die is geconfigureerd in AD of LDAP en gekoppeld is aan de organisatie  
NEW_ORG.
**Belangrijk**, zorg ervoor dat de groepen rechten krijgen op alle gerelateerde objecten, die bij elkaar horen,
anders is de configuratie niet functioneel! Dus al en gerbuiker een "template_x" moet kunnen uitvoeren, moet deze dus ook rechten hebben voor de volgende,
gerelateerde objecten:

- credentials
- inventory
- project(s)
