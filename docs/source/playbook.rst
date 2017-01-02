Creazione di playbook
=====================

Panoramica sulla sintassi YAML
##############################

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

  author:
    first_name: Gianni
    last_name: Salinetti

  last_version: 0.0.3

  chapters:
    - number: 1
      title: Introduzione

    - number: 2
      title: Installare Ansible

    - number: 3
      title: Configurazione e comando ad-hoc

    - number: 4
      title: Creazione di playbook



Il concetto di dizionario è comune in tutti i linguaggi e assume diversi nomi: dizionario, mappa, array associativo, hash, ecc. L'elemento comune è sempre lo stesso: una lista, ordinata o non ordinata, di key=value.

Concetti base
*************

Tool di verifica
****************

Uso di moduli nei playbook
##########################

Implementazione di playbook Ansible
###################################

Concetti base
*************

Formattazione
*************

Esecuzione
**********




