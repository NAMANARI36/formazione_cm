# esercizio-bonus

Descrizione generale relativa all'esercizio bonus del percorso di formazione. <br/>
**Obiettivo:** provisionare da zero un ambiente Jenkins (Controller + Agent) su una VM locale con Vagrant e Ansible, e usarlo per eseguire una pipeline che lancia un **playbook Ansible che genera altri playbook Ansible**, archiviandoli come artifact della build.

## Preview dell'infrastruttura

```
Host
 │
 ├─ Vagrant ──► VM Rocky Linux 9
 │                └─ Ansible provisioning con 4 role:
 │                     ├─ podman              ──► pacchetti + podman.socket
 │                     ├─ podman_network      ──► rete jenkins-net
 │                     ├─ jenkins_controller  ──► Quadlet + JCasC
 │                     └─ jenkins_agent       ──► Quadlet + secret JNLP
 │
 └─ Pipeline + playbook-generatore
         └─► generated/hello.yml
             generated/podman-install-dnf.yml
                  └─► archiveArtifacts
```

## Tecnologie utilizzate

| Funzione | Dettagli |
|---|---|
| Provisioning VM | Vagrant + VirtualBox, Rocky Linux 9 |
| Configuration management | Ansible |
| Container runtime | Podman + Quadlet |
| CI/CD | Jenkins Controller + Agent inbound con web socket, pipeline dichiarativa e plugin JCasC |
| Immagini | Dockerfile custom per controller e agent |
| Gestione segreti | Ansible Vault |

## Struttura cartelle

```
esercizio-bonus/
├─ vagrant/
│   └─ Vagrantfile                        
├─ ansible/                               
│   ├─ setup-ambiente-jenkins.yml         
│   ├─ ansible.cfg                        
│   ├─ requirements.yml                   
│   ├─ inventory/                         
│   └─ roles/
│       ├─ podman/                        
│       ├─ podman_network/                
│       ├─ jenkins_controller/            
│       └─ jenkins_agent/                 
├─ generatore/
│   └─ playbook-generatore.yml            
└─ pipeline/
    └─ Jenkinsfile                        
```

## Step principali

**1 — Provisioning (Vagrant + Ansible)** <br/>
Vagrant crea la VM Rocky Linux 9 con rete privata e IP statico. Ansible si connette come utente `vagrant` usando la chiave privata generata da Vagrant. <br/>
Il playbook **setup-ambiente-jenkins.yml** applica in sequenza quattro role:
- **podman** — installa Podman, abilita il *socket podman* e lo rende accessibile dai container
- **podman_network** — crea la rete *jenkins-net* con subnet e gateway dedicati
- **jenkins_controller** — prepara il contesto di build, renderizza il JCasC, builda l'immagine custom, deploya la unit Quadlet e attende che la UI risponda
- **jenkins_agent** — recupera il secret JNLP dal controller, builda l'immagine custom dell'agent, deploya la unit Quadlet con il secret iniettato a runtime e verifica che il nodo risulti online

**2 — Jenkins Controller (immagine custom + JCasC)** <br/>
Immagine basata su *jenkins/jenkins:lts* con i plugin preinstallati da **plugins.txt** e setup wizard disabilitato. La configurazione è interamente dichiarativa via **JCasC**. La UI è esposta sulla porta **8080** della VM tramite *PublishPort*.

**3 — Jenkins Agent (inbound via WebSocket)** <br/>
Immagine basata su *jenkins/inbound-agent:latest* con Podman, Ansible e openssh-client installati. Il collegamento al controller è automatico grazie a Ansible che fa una GET autenticata su *jenkins-agent.jnlp*, estrae il **secret** con una regex e lo inietta come variabile d'ambiente nella unit Quadlet dell'agent. Le build delle immagini avvengono **sull'host** tramite il socket Podman montato nel container secondo il pattern *Podman-out-of-Podman*.

**4 — Pipeline generatore** <br/>
Pipeline dichiarativa che esegue *ansible-playbook generatore/playbook-generatore.yml*. Il playbook gira in locale, crea la directory *generated/* e vi scrive due playbook Ansible:
- *hello.yml* — playbook minimale che stampa "Hello World"
- *podman-install-dnf.yml* — playbook capace di installare Podman con il packet manager *dnf* sugli host target

Attraverso il blocco *post { success }* i file generati vengono archiviati con **archiveArtifacts**, così restano scaricabili dall'interfaccia Jenkins

## Come si esegue

```bash
# 1. Avvio della VM
cd vagrant
vagrant up

# 2. Provisioning dell'ambiente Jenkins
cd ../ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook setup-ambiente-jenkins.yml --ask-vault-pass

# 3. Jenkins è raggiungibile su http://192.168.56.10:8080
#    Crea quindi il job Pipeline puntandolo a pipeline/Jenkinsfile
```
