Documentazione e Manuale Utente: Setup di Grafana, Loki e InfluxDB

Scopo: Fornire un ambiente per la raccolta, archiviazione e visualizzazione di dati di monitoraggio e log utilizzando InfluxDB (database time-series), Loki (aggregatore di log) e Grafana (dashboard).
1. Panoramica del Sistema
Questo setup automatizza l’installazione e la configurazione di tre strumenti open-source:
InfluxDB 1.11.8: Un database time-series per archiviare metriche (es. dati di sensori, performance di sistema).

Loki 3.4.3: Un sistema di aggregazione di log leggero, integrato con Grafana.

Grafana 8.5.27: Una piattaforma di visualizzazione per creare dashboard interattive basate su dati da InfluxDB e Loki.

Il tutto è gestito tramite un playbook Ansible che scarica, installa e configura i servizi su una macchina Ubuntu.
2. Prerequisiti
Sistema operativo: Ubuntu 24.04 (Noble Numbat) su una VM o un server fisico (architettura AMD64).

Accesso root: Privilegi di sudo per l’utente che esegue il playbook.

Connettività Internet: Necessaria per scaricare i pacchetti e i binari.

Spazio su disco: Almeno 2 GB liberi per i binari e i dati.

Strumenti richiesti:
ansible (installabile con sudo apt install ansible).

Pacchetti base: wget, tar, unzip.

3. Installazione
3.1. Preparazione dell’ambiente
Accedi alla tua VM o server come utente con privilegi sudo (es. ubuntu).

Aggiorna il sistema:
```
sudo apt update && sudo apt upgrade -y
```
Installa Ansible:
```
sudo apt install ansible -y
```
Crea una directory di lavoro:
```
mkdir -p /home/ubuntu/grafana
cd /home/ubuntu/grafana
```
Installa dipendenze da Requirements.yml
```
ansible-galaxy install -r requirements.yml -p /home/ubuntu/.ansible/collections
```
3.2. Esecuzione del Playbook
Esegui il playbook:
```
cd /home/ubuntu/grafana
ansible-playbook play.yml -v
```
Attendi il completamento (circa 5-10 minuti, a seconda della connessione e della macchina).

3.3. Verifica dell’installazione
Controlla lo stato dei servizi:
Grafana:
```
sudo systemctl status grafana-server
```
Loki:
```
sudo systemctl status loki
```
InfluxDB:
```
sudo systemctl status influxdb
```
Tutti i servizi dovrebbero essere Active: active (running).
4. Utilizzo del Sistema
4.1. Accesso a Grafana
Apri un browser e vai a:
```
http://<IP_DEL_SERVER>:3000
```
Sostituisci <IP_DEL_SERVER> con l’indirizzo IP della tua VM (es. 172.31.23.59).

Accedi con:
Username: admin

Password: ciao

Cambia la password al primo accesso per sicurezza.

4.2. Configurazione delle sorgenti dati in Grafana
Vai su Configuration > Data Sources.

Aggiungi InfluxDB:
Name: InfluxDB

URL: http://localhost:8086

Database: Lascia vuoto (InfluxDB 1.x usa il default).

Salva e testa la connessione.

Aggiungi Loki:
Name: Loki

URL: http://localhost:3100

Salva e testa la connessione.

4.3. Creazione di una Dashboard
Vai su Create > Dashboard.

Aggiungi un pannello:
Seleziona la sorgente dati (InfluxDB o Loki).

Scrivi una query (es. SHOW MEASUREMENTS per InfluxDB o {job="system"} per Loki).

Personalizza il grafico e salva la dashboard.

4.4. Invio di dati a InfluxDB
Usa il client CLI:
```

influx
CREATE DATABASE mydb
USE mydb
INSERT cpu,host=server01 value=0.64
```
O invia dati tramite HTTP:
bash
```
curl -i -XPOST 'http://localhost:8086/write?db=mydb' --data-binary 'cpu,host=server01 value=0.64'
```
4.5. Invio di log a Loki
Configura un client come promtail (non incluso nel playbook) per inviare log a http://localhost:3100/loki/api/v1/push.

5. Manutenzione
5.1. Avvio/Arresto dei Servizi
InfluxDB:
```

sudo systemctl start influxdb   # Avvia
sudo systemctl stop influxdb    # Ferma
sudo systemctl restart influxdb # Riavvia
```
Grafana:
```

sudo systemctl start grafana-server
sudo systemctl stop grafana-server
sudo systemctl restart grafana-server
```
Loki:
```

sudo systemctl start loki
sudo systemctl stop loki
sudo systemctl restart loki
```
5.2. Visualizzazione dei Log
InfluxDB:
```

journalctl -u influxdb
```
Grafana:
```

journalctl -u grafana-server
```
Loki:
```

journalctl -u loki
```
5.3. Backup dei Dati
InfluxDB:
```

sudo influxd backup -portable /path/to/backup
```
Grafana: Esporta le dashboard manualmente da interfaccia web o backuppa /usr/local/grafana/data.

Loki: Backuppa /usr/local/bin/loki-data (se configurato).

5.4. Aggiornamenti
Per aggiornare i componenti:
Modifica le versioni nel playbook (grafana_version, loki_version, ecc.).

Riesegui il playbook:
```

ansible-playbook play.yml -v
```
6. Risoluzione dei Problemi
6.1. InfluxDB non si avvia
Sintomo: systemctl status influxdb mostra Failed.

Soluzione:
Controlla i log:
```

journalctl -u influxdb
```
Verifica i permessi di /var/lib/influxdb:
```

sudo chown -R influxdb:influxdb /var/lib/influxdb
sudo chmod -R 0750 /var/lib/influxdb
```
Riavvia:
```

sudo systemctl restart influxdb
```
6.2. Grafana non accessibile su <IP>:3000
Sintomo: Il browser restituisce "Connection refused".

Soluzione:
Verifica che il servizio sia attivo:
```

sudo systemctl status grafana-server
```
Controlla la porta:
```

sudo netstat -tuln | grep 3000
```
Apri la porta nel firewall (se attivo):
```

sudo ufw allow 3000/tcp
```
Su AWS EC2, verifica il Security Group (aggiungi regola TCP 3000).

6.3. Loki non raccoglie log
Sintomo: Nessun dato visibile in Grafana.

Soluzione:
Verifica il servizio:
```

sudo systemctl status loki
```
Controlla la configurazione in /usr/local/bin/loki-config.yml.

Assicurati che un client (es. Promtail) stia inviando log a http://localhost:3100.

6.4. Errore durante l’installazione
Sintomo: Il playbook fallisce su InfluxDB.

Soluzione:
Controlla l’output per errori specifici.

Verifica la connessione a Internet:
```
ping download.influxdata.com
```
Esegui una pulizia e riprova (vedi sezione 3.3).

7. Configurazione Avanzata (Opzionale)
7.1. Modifica della porta di Grafana
Modifica /usr/local/grafana/conf/defaults.ini:
```
[server]
http_port = 3001
```
Riavvia Grafana:
```

sudo systemctl restart grafana-server
```
7.2. Personalizzazione di InfluxDB
Modifica /etc/influxdb/influxdb.conf:
```
[http]
bind-address = ":8086"
```
Riavvia InfluxDB:
```

sudo systemctl restart influxdb
```
8. Note Finali
Questo setup è ottimizzato per un ambiente di test o sviluppo. Per un uso in produzione:
Configura un firewall (es. ufw) e limita l’accesso alle porte.

Usa password più sicure per Grafana.

Configura backup automatici e monitoraggio dei servizi.

