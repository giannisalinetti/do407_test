Introduzione
============

Il corso **DO407 Automation with Ansible** prepara gli amministratori di sistema ad installare, confifurare e utilizzare Ansible per i processi di automazione e configuration management. 

Lo scopo del corso è di mostrare, tramite l’utilizzo di laboratori e esercitazioni guidate, metodi basi e avanzati di creazione e manutenzione di playbook finalizzati alla gestione dei sistemi.

Classroom Environment:

* **workstation.lab.example.com**    172.125.250.254
* **servera.lab.example.com**        172.25.250.10
* **serverb.lab.example.com**        172.25.250.11
* **tower.lab.example.com**          172.25.250.12



Ansible è uno strumento di configuration management, automation e orchestrazione che permette di automtizzare e standardizzare la configurazione di host, vm, container, ecc.

Architettura
############

Una delle principali caratteristiche è l'infrastruttura agentless che utilizza. Il **control node**, si connette ai **managed hosts** e propaga verso di questi le automazioni e le configurazioni predefinite.
Il control node "conosce" i vari hosts grazie a **inventory** che definiscono elenchi di host, eventualmente aggregabili in gruppi, suddivisi ad esenpio per tipologia.

Un inventory può avere un formato simile:
::

  host1
  host2

  192.168.10.110
  192.168.10.111

  [dbservers]
  db.us.extraordy.com
  db.fr.extrarody.com
  db.it.extraordy.com

  [webservers]
  web.it.extraordy.com
  web.de.extraordy.com


La configurazione di Ansible è gestita, di default tramite il file `/etc/ansible.cfg`. E' possibile fare override di questo file e utilizzare un cfg alternativo, come si vedrà in seguito. La stessa regola vale per il file inventory.


Moduli e playbook
#################

Tramite ansible è possibile eseguire singoli comandi su uno o più host remoti. Ansible fornisce una serie di **moduli** scritti in Python che vengono installati insieme ai core files. Questi moduli permettono di interagire con diverse funzionalità e servizi dei vari sistemi. Ad esempio, il modulo **yum** permette di gestire pacchetti su sistemi RHEL/CentOS mentre il modulo **shell** permette di eseguire comandi diretti utilizzando un processo bash generato dinamicamente.

Quando è necessario eseguire più moduli in modo strutturato, ad esempio per installare e configurare un servizio, Ansible mette a disposizione i **playbook**. Un playbook è un file in formato YAML in cui si vanno a definire una serie di **task** che richiamano i moduli di cui sopra.

Il seguente playbook di esempio installa il pacchetto httpd e configura il relativo servizio.
::

  - name: Simple httpd setup
    hosts: all
    remote_user: devops
    become: true
    tasks:
      - name: Install httpd
        yum:
          name: httpd
          state: latest
      - name: Start and enable httpd service
        service:
          name: httpd
          state: started
          enabled: true

.. warning:: Una caratteristica **fondamentale** della sintassi YAML è l'indentazione. Errori di indentazione causeranno problemi di parsing del playbook. Si consiglia di usare una spaziatura di due o quattro caratteri (no tabs). 


Un control node ansible deve avere installato **Python 2.6** o **2.7**. Il pachetto **ansible** deve essere installato sul control node ma non sui managed hosts.
Ansible si connette ai managed hosts tramite **ssh** e vi trasferisce i moduli che devono essere eseguiti. Ovviamente anche gli host devono avere installato Python, nello specifico dalla versione 2.4 o superiore.

Per consentire la comunicazione tra control node e host tramite ssh è necessario definire disporre di una coppia di chiavi e copiare sugli host la chiave pubblica in modo da permettere un login passwordless.

Ansible permette di eseguire operazioni con i privilegi dell'utente che si connette remotamente (**devops** nell'esempio del playbook precedente) oppure di elevarne le credenziali tramite **su** o **sudo** (default sudo, direttiva `become: true`). 

Per limitare l'uso della password durante le operazioni è possibile configurare il file `/etc/sudoers` negli host in modo da consentire l'esecuzione di comandi senza password.
L'esempio seguente mostra la direttiva che disattiva l'inserimento della password per tutti gli utenti del gruppo **devops**:
::

  %devops ALL=(ALL)NOPASSWD: ALL

Modalità di Connessione
#######################

Ansible supporta diverse modalità di connessione. Di default, utilizza il plugin ssh ma sono disponibili diversi altri plugin:

* **ssh**, basato su OpenSSH. Prevede che sia supportata l'opzione **ControlPersist**.
* **local**, per connessioni locali
* **paramiko**, un'implementazione di OpenSSH in Python che supporta connessioni persistenti. Usato per comunicare con host RHEL/CentOS 6.
* **winrm**, plugin per la connessione verso host Windows.
* **docker**, per container Docker.
* **jail**, per gestire jail BSD
* **lxc**, per container lxc
* **lxd**, per container lxd

Host Inventory
##############

Ansible supporta inventory statici e dinamici. Un inventory statico è un file in formato INI che contiene elenchi di host e gruppi. Un inventory dinamico è un file Python eseguibile che permette di generare dizionari contenenti gruppi dinamici. Inventory statici e dinamici possono essere integrati assieme.

La location di default è il file `/etc/ansible/hosts`. Per eseguire ansible con un altro inventory:
::

  ansible [-i|--inventory] /path/to/inventory

Un inventory può definire singoli host o **host group**. Gli host group possono a loro volta essere aggregati in macrogruppi che vengono definiti con il suffisso **children**. 

Nell'esempio seguente i due gruppi **mailservers** e **mailclients** fanno parte di un gruppo **mail**
::

  localhost
  node1

  [mailservers]
  node3
  node4

  [mailclients]
  node5

  [mail:children]
  mailservers
  mailclients

E' possibile utilizzare vari tipi di globbing per semplificare la definizione degli host:

  Range, nel formato `[START:END]`, utilizzabili sia su indirizzi IP che su hostname:
  :: 
    
    10.10.1.[10:20]
    server[10:20].traininglab.com

Gli inventory possono includere **variabili**, sia a livello di host che di gruppo.
::

  [webservers]
  web1.example.com
  web2.example.com http_port=8080

  [webservers:vars]
  http_port:80

  [dbservers]
  web1.example.com
  db1.example.com

In questo esempio la variabile http_port è definita a livello host per web2.example.com e a livello group per webservers. Il valora a livello host farà override sul valore a livello group.



