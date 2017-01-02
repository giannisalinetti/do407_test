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


Concetti base
*************

Formattazione
*************

Esecuzione
**********




