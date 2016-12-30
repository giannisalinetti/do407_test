Configurazione e comandi ad-hoc
===============================

Il file ansible.cfg
###################

La configurazione di Ansible viene gestita, di default, nel file `/etc/ansible/ansible.cfg`. In questo file, in formato INI, esistono diverse sezioni:

* **[defaults]**
* **[privilege_escalation]**
* **[paramiko_connection]**
* **[ssh_connection]**
* **[accelerate]**
* **[selinux]**
* **[colors]**

La sezione **[defaults]** definisce impostazioni di base come il default path dell'inventory (`/etc/ansible/inventory`), il path per le librerie di moduli custom, l'utente remoto che si connetterà agli host (`remote_user`) e l'utente verso cui verrà fatta la privilege escalation (`become_user`), ovvero chi si diventerà con un sudo.

La sezione **[privilege_escalation]** definisce le modalità di elevazione dei privilegi dell'utente che si connette agli host. Contiene solo quattro, ma fondamentali parametri i cui valori di default sono:
::

  become=True
  become_method=sudo
  become_user=root
  become_ask_pass=False

Le sezioni **[ssh_connection]** e **[paramiko_connection]** permettono di definire i parametri di connessione OpenSSH e Paramiko.

Questo significa che quando Ansible si connette ad un host per elvare i privilegi usa **sudo** per diventare **root**. Di default si aspetta di non dover passare alcuna password. Queste impostazioni possono essere sempre alterate a livello di playbook.

Notare che le impostazioni nel file sono quasi tutti commentate in quanto già impostate di default nel codice. Una menzione particolare alla property **forks** il cui valore di default è **5**. Questo significa che Ansible non effettuerà, di default, più di 5 fork contemporaneamente, e quindi non più di 5 connessioni in parallelo. Il tuning dei fork è molto importante per non sovraccaricare eccessivamente il controller e può essere anche gestito a livello del signolo playbook con la direttiva **serial**.

Una delle caratteristiche più interessanti di Ansible è la possibilità di fare override del file ansible.cfg creando altri file omonimi nella directory dei playbook, nella home dell'utente o puntando al contentuto espresso da una variabile d'ambiente. In generale si possono elencare quattro possibili distribuzioni del file `ansible.cfg`.

* Nella directory di configurazione di sistema (default):
  ::
    
    /etc/ansible/ansible.cfg

* Nella directory del progetto (**best practice**). Se esiste un file `ansible.cfg` in questo path verrà utilizzato quest'ultimo.
  ::

    /home/ansiblelab/myproject/ansible.cfg

* Nella home dell'utente. Se il file `ansible.cfg` è definito nella home dell'utente verrà utilizzato per tutti i playbook che non hanno un loro file nella working directory. Se invece esiste quest'ultimo avrà sempre e comunque la priorità.
  ::

    /home/ansiblelab/ansible.cfg

* Come variabile d'ambiente `ANSIBLE_CONFIG`. Questo approccio permette di bypassare le tre modalità sopra descritte acquistando precedenza su tutte.
  ::

    export ANSIBLE_CONFIG=/opt/test/ansible.cfg

In qualsiasi momento, per testare quale file di configurazione si sta usando, si può usare il comando
::

  ansible --version

L'output mostra il file `ansible.cfg` attualmente in uso:
::

  ansible 2.2.0.0
    config file = /etc/ansible/ansible.cfg
    configured module search path = Default w/o overrides

Si può ottenere questa informazione durante l'esecuzione di comandi appendendo l'opzione -v (verbose):
::

  ansible all --list-hosts -v

L'output mostra in testa quale file `ansible.cfg` si sta utilizzando:
::

  Using /etc/ansible/ansible.cfg as config file
    hosts (2):
      localhost
      yoda.traininglab.com

.. note:: L'override di un file di configurazione non si limita ad alterare le proprietà diversamente definite. Ansible userò solo e soltanto le impostazioni contenute al suo interno. Pertanto, se si ha bisogno di utilizzare anche delle impostazioni definite a livello più generale nel file `/etc/ansible/ansible.cfg` sarà necessario replicarle nel file di configurazione che si intende utilizzare.

Creazione di una working directory
**********************************

L'esempio seguente mostra la creazione di una working directory con file `ansible.cfg` e `inventory` indipendenti.
::

  mkdir /home/ansiblelab/firstproject
  
  cat > /home/ansiblelab/firstpriect/ansible.cfg << EOF
  inventory = /home/ansiblelab/firstproject/inventory
  EOF
  
  cat > /home/ansiblelab/firstproject/inventory << EOF
  localhost

  [dev]
  node1
  node2

  [prod]
  node3
  node4
  EOF

Si è creato un file `ansible.cfg` minimale con un solo parametro custom (per il resto varranno le impostazioni di default scolpite nel codice) che definisce il path di un file inventory. Il file inventory, a sua volta, è stato popolato con due gruppi, **dev** e **prod** e da localhost, ovvero il control node stesso. 

Da questo momento in poi all'interno di `/home/ansiblelab/firstproject` si potranno eseguire **comandi ad-hoc** e **playbook** che impatteranno sugli host definiti nell'inventory.

Comandi ad-hoc
##############

Moduli
******

Ansible utilizza, per l'esecuzione dei comandi, una serie di **moduli**. Un modulo è un file Python che contiene classi o funzioni per svolgere determinate operazioni.
I moduli installati da pacchetto vengono copiati in RHEL/CentOS sotto `/usr/lib/python2.7/site-packages/ansible/modules`. In questa directory sono raggruppati per moduli **core** e per **extras**. Essendo codice Python è possibile leggerli e studiarli direttamente sulla macchina oppure clonare i repository GitHub:

* `<https://github.com/ansible/ansible-modules-core.git>`_ 
* `<https://github.com/ansible/ansible-modules-extras.git>`_

All'inteno i moduli core e extras contengono a loro volta sottodirectory che raggruppano i vari moduli per tipologia di funzione. 

Ad esempio, sotto `core/commands` si possono trovare i moduli `command.py` e `shell.py` per l'esecuzione di comandi remoti. Il primo permette di eseguire comandi utilizzando un interprete minimale mentre il secondo permette di eseguire comandi utilizzando un interprete bash. Il modulo **shell** è fondamentale quando si devono utlizzare dei bash builtin, eseguire
comandi in pipe, redirect, ecc.

In `core/packaging/os` si trovano invece moduli per gestire pacchetti sui vari sistemi come **yum** o **apt** mentre in `core/packaging/language` vi sono moduli specifici di linguaggi come Python (**pip**, **easy_install**) e Ruby (**gem**).

In `core/system` si trovano moduli per l'amministrazione di sistema come **service** per gestire l'esecuzione di servizi, **selinux**, **sysctl**, **systemd**, **mount**, ecc.

Non è necessario navigare all'interno delle directory di installazione dei moduli per utilizzarli. Per sfogliare l'elenco completo si utilizza il comando
::

  ansible-doc -l

Questo produce un output pulito e paginato in less con l'elenco dei moduli disponibili (ovvero installati). Per leggere la documentazione di un singolo modulo (che è in realtà scritta nel file .py del modulo stesso) si esegua
::

  ansible-doc <module_name>

La documentazione varia da modulo a modulo e può essere più o meno estesa. Per mostrare un output minimale (snippet) contente solo gli esempi di applicazione presenti in fondo alle pagine di documentazione si esegua
::

  ansible-doc -s <module_name>

Esecuzione di comandi
*********************

Per eseguire comandi in Ansible si usa la seguente sintassi:
::

  ansible <target> -m <module> [-a <module_arguments>] [-i inventory]

Un primo esempio molto semplice è l'uso del modulo **ping**, che serve semplicamente a verificare se un host è attivo. Utilizzando l'inventory dell'esempio precedente, si va a eseguire il comando sul gruppo **prod**:
::

  ansible prod -m ping

Si otterrà il seguente output:
::

  node3 | SUCCESS => {
      "changed": false, 
      "ping": "pong"
  }

  node4 | SUCCESS => {
      "changed": false, 
      "ping": "pong"
  }

Come si può notare, l'output dei comandi ad-hoc Ansible è in formato JSON.

Per eseguire un comando sugli host, si può usare il modulo **command**
::

  ansible all -m command -a 'id'

Questo comando esegui su tutti (**all**) gli host definiti nell'inventory il comando specificato nell'argomento, ovvero **id**. L'output di questo comando può variare in base all'utente con cui ci si connette o alla privilege escalation.

As esempio, se eseguito sul contoller stesso:
::

  ansible localhost -m command -a 'id' --connection=local

Produce il seguente output:
::

  localhost | SUCCESS | rc=0 >>
  uid=1000(student) gid=1000(student) groups=1000(student),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

L'opzione `--connection=local` in questo caso fa si che Ansible non cerchi di fare ssh su se stesso.

Ci si può connettere agli host con uno specifico utente, magari uno che è abilitato a fare sudo senza password (**best practice**).
::

  ansible dev -m command -a 'id' -u devops

In questo caso ci si connette al gruppo **dev** con l'utente **devops**.
Si otterrà un output simile al seguente:
::

  node1 | SUCCESS | rc=0 >>
  uid=1001(devops) gid=1001(devops) groups=1001(devops),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

  node2 | SUCCESS | rc=0 >>
  uid=1001(devops) gid=1001(devops) groups=1001(devops),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

E' possibile definire in modo persistente l'utente remoto nel file `ansible.cfg` con la direttiva **remote_user** nella sezione [defaults]:
::

  [defaults]
  inventory = home/ansiblelab/firstproject/inventory
  remote_user = devops

Privilege Escalation
********************

Per eseguire un comando con privilegi elevati si aggiunge l'opzione `-b|--become`
::

  ansible dev -m command -a 'id' -u devops -b

In questo caso l'output sarà il seguente:
::

  node1 | SUCCESS | rc=0 >>
  uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

  node2 | SUCCESS | rc=0 >>
  uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
  
L'output del comando `id` questa volta mostra l'utente **root**, a dimostrazione dell'azione dell'effettiva escalation dei privilegi. L'esempio in questione da per scontato che l'utente devops possa fare sudo senza password. In caso contrario è necessario fornire anche informazioni di autenticazione tramite l'opzione `-K|--ask-become-pass`:
::

  ansible dev -m command -a 'id' -u devops -b --ask-become-pass

In questo modo verrà richesta al prompt la password per la privelege esacalation.

Per eseguire il comando come utente diverso da root si può usare l'opzione `--become-user=<utente>`:
::

  ansible dev -m command -a 'id' -u devops -b --become-user=jboss

Anche in questo caso le impostazioni di privilege escalation possono essere scolpite nel file `ansible.cfg`:
::

  [defaults]
  inventory = home/ansiblelab/firstproject/inventory
  remote_user = devops

  [privilege_escalation]
  become = True
  become_method = sudo
  become_user = root
  become_ask_pass = False


Command vs Shell
****************

Quando è davvero necessario usare il modulo **shell** piuttosto che il modulo **command**? Ad esempio quando è necessario estrapolare delle variabili d'ambiente, oppure se si devono utlizzare pipe, redirect o shell builtins. Il comando seguente
::

  ansible prod -m command -a 'set'

Produce un errore in quanto il modulo command non istanzia nessuna shell. Per ottenere l'elenco delle variabili d'ambiente si usa quindi il modulo shell:
::

  ansible prod -m shell -a 'set'


Quando è davvero necessario usare il modulo **shell** piuttosto che il modulo **command**? Ad esempio quando è necessario estrapolare delle variabili d'ambiente, oppure se si devono utlizzare pipe, redirect o shell builtins. Il comando seguente
::

  ansible prod -m command -a 'set'

Produce un errore in quanto il modulo command non istanzia nessuna shell. Per ottenere l'elenco delle variabili d'ambiente si usa quindi il modulo shell:
::

  ansible prod -m shell -a 'set'


In alcuni casi può essere utile produrre l'output in formato singola linea, ad esempio per ulteriore processamento. In questo caso si può utilizzare l'opzione `-o`:
::

  ansible prod -m shell -a 'set' -o

Inventory dinamici
##################

Spesso gli inventory statici sono sufficienti per contenti on premise con un parco macchine fisso nel tempo ma quando si lavora in contesti cloud, dove è necessario poter scalare orizzontalmente, come IaaS OpenStack o PaaS OpenShift è necessario poter generare degli inventory dinamicamente in base agli host (o container, o jail, ecc) presenti.
Per questo motivo si possono creare inventory dinamici, ovvero file eseguibili scritti preferibilmente in Python (ma può essere usato qualsiasi altro linguaggio) che producano un output formattato in formato hash/mappa/dizionario. 

Quando Ansible trova un file eseguibile nel path degli inventari lo esegue e utilizza l'inventario generato dinamicamente. Sotto `<https://github.com/ansible/ansible/tree/devel/contrib/inventory>`_ sono già presenti numerosi template per inventory dinamici che supportano numerose piattaforme cloud come OpenStack, Amazon AWS, Rackspace, piattaforme di virtualizzazione on premise come oVirt/RHEV, PaaS OpenShift, Satellite, providers esterni come Digital Ocean o Linode.

Se si scrive un inventory dinamico occorre tenere a mente che questo dovrà supportare almeno due opzioni:

* `--list` per produrre un output JSON in formato dizionario con gruppi e relativi host
* `--host <hostname>` per produrre un output JSON in formato dizionario delle variabili associato all'host specificato

E' possibile avere in una stessa directory file inventory statici e dinamici e puntare, nel file `ansible.cfg` non ad un singolo file ma a tutta la directory. Ansible in questo caso processerà tutti i file al suo interno.
::

  [defaults]
  inventory = /home/ansiblelab/inv_dir

.. warning:: Quando si lavora con più inventory statici e dinamici in parallelo è necessario prestare attenzione ai nomi dei file. Ansible processa i file in ordine alfanumerico e pertanto possono verificarsi errori se un inventory dinamico viene processato dopo di un file inventory statico che contiene richiami a gruppi definiti dinamicamente.


