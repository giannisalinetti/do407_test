Creazione di playbook
=====================

YAML: panoramica e concetti base
################################

Sintassi
********

**YAML** sta per **YAML Ain't Markup Language** ed è un linguaggio per la serializzazione di dati: il suo scopo è rappresentare strutture di dati. Assomiglia per molti versi, in particolare il tipo di dati rappresentati al più diffuso JSON.
Uno degli aspetti più importanti è l'indentazione, in quanto YAML non utilizza a differenza di JSON, parentesi per descrivere, blocchi, dizionari e liste. Il rispetto dell'indentazione è fondamentale per scrivere file YAML corretti.

.. warning: Quando si scrivono file YAML è fortemente sconsigliato l'uso dei tab ma solo degli spazi. 

In vim si può far sostituire l'azione del tab con un numero configurabile di spazi e rendere questa modifica persistente tramite il file `~/.vimrc`:
::
  
  vim ~/.vimrc
  set tabstop=2
  set shiftwidth=2
  set softtabstop=2
  set expandtab

In questo modo il tab verrà sostituito da 2 spazi.

Un file YAML conterrà dati di tipo intero, stringhe, liste e dizionari.

Un esempio di file YAML contentente diversi tipi di dati:
::

  ---
  title: AnsibleLab

  # This is just a comment

  author:
    first_name: Gianni
    last_name: Salinetti

  last_version: "0.0.3"
  
  # Another comment
  chapters:
    - number: 1
      title: 'Introduzione'

    - number: 2
      title: 'Installare Ansible'

    - number: 3
      title: 'Configurazione e comandi ad-hoc'

    - number: 4
      title: 'Creazione di playbook'

I file YAML iniziano con la stringa `---` e possono essere opzionalmente chiusi con la stringa `...` anche se solitamente viene utilizzata solo la stringa di apertura.
I tipi di dati definiti in questo esempio sono tre:

* **Stringhe**: author e last_version sono chiavi contenenti stringhe semplici. Le stringhe possono essere scritte con o senza quote, singoli o doppi.
* **Dizionari**: author è un dizionario che contiene una serie di **key=value**
* **Liste**: chapters è un lista i cui elementi sono contraddistinti da un **"-"**. Gli elementi sono a loro volta dizionari.

Il concetto di dizionario è comune in tutti i linguaggi e assume diversi nomi: dizionario, mappa, array associativo, hash, ecc. L'elemento comune è sempre lo stesso: una lista, ordinata o non ordinata, di key=value.
I dizionari possono essere rappresentati in modo alternativo utilizzando una sintassi in stile Python:
::

  ---
    author:
      { first_name: Dennis, last_name: Ritchie }

Anche le liste, come i dizionari possono essere rappresentate con la sintassi Python:
::

  ---
    chapters:
      [one, two, three]

Notare in ambedue i casi l'indentazione. In generale, nello scrivere playbook ansible è più conveniente utilizzare la sintassi standard ma la sintassi Python può tornare utile in alcuni casi.

Tool di verifica
****************

Per verificare i file yaml si può utilizzare il modulo `yaml` in python.
::

  python -c 'import yaml, sys; print yaml.load(sys.stdin)' < myfile.yml

In questo esempio si usa python direttamente da linea di comando (`python -c`) e si caricano due moduli: `yaml` e `sys`. Il primo contiene il metodo load necessario a parsare un input yaml e prende in argomento il contenuto dello stdin (`myfile.yml`), caricato tramite il metodo `sys.stdin`.

Un altro strumento utile di verifica è il tool **yamllint** (`<https://github.com/adrienverge/yamllint>`_).

Infine, in caso di redazione di playbook Ansible, si può fare verifica della sintassi con il comando
::

  ansible-playbook <playbook_file> --syntax-check

.. warning: L'opzione --syntax-check permette di verificare solo eventuali errori di sintassi, non testa errori di logica, path o url.

Implementazione di playbook Ansible
###################################

Si è gia parlato dei moduli Ansible e del loro funzionamento. Si è visto come da command line, con il comando `ansible` si possa eseguire uno specifico modulo.
I playbook permettono di eseguire più moduli in sequenze ordinate all'interno di attività dette **task**, contentute a loro volta in un blocco esecutivo che prende il nome di **play**. Un playbook può contenere più di un play, eseguiti in sequenza secondo l'ordine di apparizione.

Ogni play è riferito ad uno specifico set di **hosts**: questi saranno riferiti agli host e gruppi indicati nell'inventory.

In ogni play si anche possono definire:

* variabili, rappresentate con la chiave **vars**
* impostazioni di privilege escalation: **become**, **become_user**, **become_ask_password**
* impostazioni di connessione remota: **remote_user** (**user** nelle versioni < 2.0)

L'elenco è volutamente impompleto e nuovi elementi veranno aggiunti successivamente.

Concetti base
*************

I play in yaml sono organizzati all'interno di un'unica grande lista, pertanto ogni play dovrà iniziare con un **-** e senza indentazine.
::

  ---
  - name: This is play1
    hosts: all
    remote_user: devops
    become: true
    tasks:
      - name: Install httpd package
        yum:
          name: httpd
          state: latest
      - name: Start and enable service
        service:
          name: httpd
          state: started
          enabled: true
      - name: Create web content
        copy:
          content: "Web root on {{ inventory_hostname }}"
          dest: /var/www/html/index.html
          mode: 0755

  - name: This is play2
    hosts: localhost
    tasks:
      - name: Test installation
        uri:
          url: "http://{{ inventory_hostname }}"
          status_code: 200

Il playbook di questo esempio contiene molte informazioni e molto materiale che verrà trattato gradualmente.
Si possono notare intanto i due play, rappresentati da due elementi di una lista, contraddistinti da un **-** e descritti da un (opzionale) campo **name** che ne fornisce una descrizione.

La chiave **hosts** definisce nel primo play **all** come target, ovvero tutti gli host definiti nell'inventory: su questi host verrà installato `httpd`, avviato e abilitato il relativo servizio, e creato un file index.html.
Nel secondo play hosts definisce **localhost** come target per eseguire un semplice modulo **uri** che testa una url e verifica lo status code restituito.

Tutti i moduli vengono sempre eseguiti come item della lista **tasks**.

Formattazione
*************

I moduli **yum**, **service**, **copy** e **uri** hanno tutti una caratteristica comune: contengono uno o più argomenti. Gli argomenti dei moduli nei playbook possono essere scritti in diversi modi:

**Linea singola**: tutti gli argomenti sono inseriti in linea con il nome del modulo.
::

  tasks:
    - yum: name=httpd state=latest
    - service: name=httpd state=started enabled=true

**Multilinea**: La fine di ogni linea deve essere interrotta dopo uno spazio. Difficile da debuggare senza la visualizzazioe di spazi, eof e carriage returns.
::
  
  tasks:
    - yum: name=httpd 
           state=latest
    - service: name=httpd 
               state=started 
               enabled=true

**Dizionario**: gli argomenti sono elencati in un dizionario YAML.
::

  tasks:
    - yum:
        name: httpd
        state: latest
    - service:
        name: httpd
        state: started
        enabled: true

I formati più comunemente usati e riscontrabili in rete sono quello a linea singola e quello a dizionario.

.. note:: Per visualizzare i CR e gli spazi in vim usare il comando `:set list`

Da Ansible 2.0 è possibile raggruppare i task all'interno di **blocks**, sottogruppi che possono aiutare a dividere le operazioni per tipologia.
::

  tasks:
  - block:
    - yum: Install httpd
        name: httpd
        state: latest
    - yum: Install mariadb
        name: mariadb
        staete: latest
  - block.
    - service:
        name: httpd
        state: started
    - service:
        name: mariadb
        state: started

Il task dell'esempio contiene due blocchi: il primo raggruppa operazioni di installazione dei pacchetti httpd e mariadb, il secondo avvia i relativi servizi.

Esecuzione
**********

I playbook vengono eseguiti con l'utility `ansible-playbook`. Per eseguire un playbook è sufficiente eseguire il comando seguito dal nome del playbook.
::

  ansible-playbook playbook.yml

Tra le opzioni del comando `ansible-playbook` è opportuno segnalare:

* **-C, --check**: Effettua un dry run del playbook senza apportare modifiche effettive. Ad ogni modo può generare, in particolare quando si usano variabili generate dinamicamente nell'esecuzione condizionale, falsi errori.

* **--step**: Esegue i task uno per uno in modo interattivo.

* **--syntax-check**: Verifica la sintassi del file playbook senza eseguirlo

* **-v, --verbose**: Modalità verbose. Accetta più livelli di verbosità definibili con le opzioni -v, -vv, -vvv-, -vvvv. Utile in particolare per debuggare problemi di connessione ssh.

* **-u REMOTE_USER, --user=REMOTE_USER**: Definisce l'utente con cui si effettuerà la connessione remota.

* **-k, --ask-pass**: Impostala richiesta di una password per la connessione (necessario se non sono installati i certificati).

* **-c CONNECTION, --connection=CONNECTION**:

* **-b, --become**: Effettua privilege escalation

* **--become-method:BECOME_METHOD**: Definisce la modalità di privilege escalation (default=sudo). Alternative: [ sudo | su | pbrun | pfexec | doas | dzdo | ksu ]

* **--become-user:BECOME_USER**: La privilege escalation eseguirà le operazioni come questo utente

* **-K, --ask-become-pass**: Imposta la richiesta di una password per la privilege escalation


