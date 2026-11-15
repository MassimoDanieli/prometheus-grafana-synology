# prometheus-grafana-synology

Monitoraggio di un NAS Synology con Prometheus e Grafana, pensato per girare su un Raspberry Pi (ma va anche su amd64).

| Servizio | Porta | |
|---|---|---|
| Prometheus | 9090 | raccoglie le metriche |
| Grafana | 3000 | dashboard `Synology-NAS` già configurata |
| Alertmanager | 9093 | notifiche (da configurare) |
| SNMP exporter | 9116 | legge il NAS via SNMP |
| node-exporter | 9100 | la macchina che ospita lo stack |
| cAdvisor | 8080 | i container |

## Avvio

```bash
cp grafana/config.monitoring.example grafana/config.monitoring   # password admin e secret key di Grafana
$EDITOR prometheus/prometheus.yml                                # IP del NAS e delle altre macchine
docker compose up -d
```

Poi Grafana su `http://<host>:3000` (utente `admin`, password quella messa in `config.monitoring`).

## Da sapere

- **SNMP**: sul NAS va attivato SNMP v2c (Pannello di controllo → Terminale e SNMP). La community è `public` in `snmp_exporter/snmp.yml`, sotto `auths:`. Meglio cambiarla, sia lì sia sul NAS.
- **Target**: gli indirizzi `192.168.0.x` in `prometheus/prometheus.yml` sono quelli di casa mia. Vanno cambiati.
- **Alert**: Alertmanager è collegato a Prometheus, ma i messaggi non vanno da nessuna parte finché non configuri un notifier in `alertmanager/config.yml` (c'è un esempio Slack commentato). Le regole di alert sono a parte.
- **Versioni**: le immagini sono fissate. Con `:latest` lo stack si rompeva quando snmp_exporter, node_exporter e Prometheus cambiavano formato di configurazione.

La CI controlla ogni modifica con gli stessi strumenti e le stesse versioni: `docker compose config`, `promtool`, `amtool`, `snmp_exporter --dry-run`.
