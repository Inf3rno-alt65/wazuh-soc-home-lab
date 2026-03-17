# Wazuh SOC Home Lab – Monitoraggio SIEM e Simulazione Attacco

## Panoramica del progetto

In questo progetto ho realizzato un **SOC Home Lab** utilizzando **Wazuh SIEM** per monitorare un endpoint Windows e simulare attività malevole all’interno di un ambiente controllato.

L’obiettivo del laboratorio è stato quello di simulare un **scenario realistico di monitoraggio SOC**, raccogliendo log di sicurezza da una macchina Windows, analizzandoli tramite Wazuh e generando eventi di sicurezza attraverso una macchina attaccante.

Durante il laboratorio ho configurato:

* un **server Wazuh SIEM su Ubuntu**
* un **endpoint Windows 11 con Wazuh Agent**
* una **macchina Kali Linux utilizzata per simulare attacchi**

Questo progetto mi ha permesso di acquisire esperienza pratica nelle seguenti attività:

* configurazione di una piattaforma **SIEM**
* integrazione di **endpoint monitoring**
* raccolta e analisi dei **log di sicurezza Windows**
* simulazione di **attacchi informatici**
* troubleshooting di problemi di connessione tra agent e SIEM

---

# Architettura del laboratorio

Il laboratorio è composto da **tre macchine virtuali** eseguite tramite VirtualBox.

| Macchina         | Sistema operativo | Ruolo               | IP             |
| ---------------- | ----------------- | ------------------- | -------------- |
| Wazuh Server     | Ubuntu            | SIEM                | 192.168.56.101 |
| Windows Endpoint | Windows 11        | Macchina monitorata | 192.168.56.102 |
| Kali Linux       | Kali Linux        | Macchina attaccante | 192.168.56.103 |

Le macchine comunicano tra loro tramite una rete **Host-Only configurata in VirtualBox**.

Il Wazuh agent installato su Windows invia i log al server Wazuh tramite **porta TCP 1514**.

---

# Tecnologie utilizzate

Durante il progetto ho utilizzato le seguenti tecnologie:

* **Wazuh SIEM**
* **Ubuntu Linux**
* **Windows 11**
* **Kali Linux**
* **VirtualBox**
* **PowerShell**
* **Windows Event Logs**
* **Hydra (password brute-force tool)**
* **crackmapexec**

---

# Installazione del Wazuh Agent su Windows

Per installare l’agent sulla macchina Windows ho utilizzato il seguente comando PowerShell:

```
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.3-1.msi -OutFile $env:tmp\wazuh-agent

msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.56.101' WAZUH_AGENT_GROUP='default' WAZUH_AGENT_NAME='win11_lab'
```

L’agent è stato configurato per comunicare con il server Wazuh utilizzando:

* **Manager IP:** 192.168.56.101
* **Porta:** 1514
* **Protocollo:** TCP

Una volta completata l’installazione, la macchina Windows ha iniziato a inviare i log di sicurezza al SIEM.

---

# Simulazione Attacco – Brute Force

Per generare eventi di sicurezza e testare il funzionamento del SIEM ho utilizzato una **macchina Kali Linux** per simulare un attacco.

L’attacco simulato è stato un **tentativo di brute-force su autenticazione Windows**.

Questo tipo di attacco genera numerosi tentativi di login falliti, che vengono registrati nei log di sicurezza di Windows.

---

# Tool utilizzato: crackmapexec

Per eseguire l’attacco ho utilizzato **crackmapexec**, un tool molto utilizzato nei penetration test per effettuare attacchi brute-force su diversi protocolli.

Esempio di comando utilizzato durante il laboratorio:

```
crackmapexec smb 192.168.56.102 -u admin -p passwords.txt
```
### Significato dei parametri

| Parametro      | Descrizione                              |
| -------------- | ---------------------------------------- |
| crackmapexec   | tool utilizzato                          |
| smb            | protocollo target                        |
| 192.168.56.102 | indirizzo IP della macchina Windows      |
| -u             | utente utilizzato per i tentativi di autenticazione | 
| admin          | nome dell'account utilizzato per login   |
| -p             | lista di password passata per brute force|
| passwords-txt  | file contenten le password da provare    |

Ho deciso di sfruttare le vulnerabilità del protocollo smb in quanto la macchina attaccata (Windows 11 HOME), non essendo professional non ha la possibilità di attivare l'RDP

Il Siem rileva anche attacchi all'RDP, di seguito un comando di test con il tool hydra

```
hydra -l administrator -P passwords.txt rdp://192.168.56.102
```

### Significato dei parametri

| Parametro      | Descrizione                              |
| -------------- | ---------------------------------------- |
| -l             | utente utilizzato nei tentativi di login |
| -P             | wordlist di password                     |
| rdp://         | protocollo target                        |
| 192.168.56.102 | indirizzo IP della macchina Windows      |

---

# Eventi di sicurezza generati

Durante l’attacco sono stati generati diversi eventi di sicurezza nei log di Windows, tra cui:

* **Event ID 4625 – Failed Logon**
* tentativi di autenticazione ripetuti
* numerosi login falliti nello stesso intervallo di tempo

Questi eventi sono stati raccolti dal **Wazuh agent** e inviati al server SIEM.

---

# Rilevamento dell’attacco nel SIEM

Una volta ricevuti dal server, i log sono stati analizzati da Wazuh.

Gli indicatori che hanno permesso di identificare il comportamento sospetto includono:

* elevato numero di **login falliti**
* tentativi di accesso ripetuti dallo stesso IP
* numerosi eventi di autenticazione in breve tempo
---

# Problemi incontrati durante il laboratorio

Durante la configurazione del laboratorio ho incontrato diversi problemi tecnici che ho dovuto risolvere.

## Agent con stato "Never Connected"

### Problema

L’agent appariva nella console Wazuh ma risultava **Never Connected**.

### Soluzione

Ho verificato:

* la connettività tra le macchine tramite `ping`
* l’indirizzo IP del server nel file `ossec.conf`
* il riavvio del servizio Wazuh agent su Windows

---

## Errore "Duplicate Agent Name"

### Problema

Durante l’enrollment dell’agent veniva generato l’errore:

```
Duplicate agent name
```

### Causa

Era stato creato più di un agent con lo stesso nome durante i tentativi di registrazione.

### Soluzione

Ho eliminato gli agent duplicati dal server Wazuh e registrato nuovamente l’agent.

---

## Problemi di networking VirtualBox

### Problema

Le macchine non riuscivano a comunicare tra loro.

### Soluzione

Ho configurato le schede di rete delle macchine virtuali utilizzando **Host-Only Adapter** e verificato la comunicazione con il comando:

```
ping 192.168.56.101
```

---

# Competenze dimostrate

Questo progetto dimostra diverse competenze rilevanti per un ruolo di **Cyber Security Analyst / SOC Analyst**.

### SIEM Deployment

Installazione e configurazione della piattaforma Wazuh.

### Endpoint Monitoring

Integrazione di una macchina Windows con il SIEM.

### Log Analysis

Raccolta e analisi dei log di sicurezza di Windows.

### Attack Simulation

Simulazione di attacco brute-force tramite Kali Linux.

### Troubleshooting

Risoluzione di problemi di:

* connettività agent
* configurazione SIEM
* networking tra macchine virtuali

---

# Struttura della repository

```
wazuh-soc-home-lab
│
├── README.md
│  
│
├── config
│   └── ossec.conf
│
├── architecture
│   └── soc-lab-diagram.png
│
└── lab.evidence

```

---

# Conclusione

Questo laboratorio mi ha permesso di creare un ambiente di monitoraggio simile a quello utilizzato nei **Security Operations Center**.

Attraverso questo progetto ho acquisito esperienza pratica nella configurazione di un **SIEM**, nel monitoraggio degli endpoint e nella rilevazione di attività sospette generate da un attacco simulato.
