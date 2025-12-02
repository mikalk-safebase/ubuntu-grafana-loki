
# Ubuntu Grafana Loki Stack

Lokal logg-stack for å lære Loki. Og logcli. 

## Kom i gang

**Start Grafana på http://localhost:3000/**
```bash
docker compose up -d
```


## Bruk av LogCLI

**Lag et logcli-alias:**
```bash
alias logcli='docker run --net=host --rm -i -v /etc/localtime:/etc/localtime:ro grafana/logcli:3.0.0 --addr="http://localhost:3100" --timezone=Local'
```


### Søk etter tekst (Som grep):
Bruk `|=` for å filtrere linjer som *må* inneholde teksten.

```bash
logcli query '{job="varlogs"} |= "error"'
```

### Søk etter tekst (Case insensitive):
|~ angir regex. (?i) gjør at den finner både "Error", "error" og "ERROR".
```bash
logcli query '{job="varlogs"} |~ "(?i)error"'
```

### Ekskluder tekst (Som grep -v):
Bruk `!=` for å fjerne støy.
```bash
logcli query '{job="varlogs"} != "debug" != "info"'
```

### Sanntids-logger (Som tail -f):
```bash
logcli query --tail '{job="varlogs"}'
```

### Siste 2 timer:
```bash
logcli query --since=2h '{job="varlogs"}'
```

### Spesifikt tidsrom (ISO 8601):
Bruk formatet `YYYY-MM-DDTHH:MM:SSZ` (UTC). 
Tips: Du kan bruke `date -u` i terminalen for å sjekke hva klokka er i UTC akkurat nå.
```bash
logcli query --from="2025-12-02T04:00:00Z" --to="2025-12-02T05:00:00Z" '{job="varlogs"}'
```

### Ren output (Raw)
Fjerner tidsstempel og labels for enklere piping til awk eller grep.
```bash
logcli query --output=raw '{job="varlogs"}' | grep "kernel" | awk '{print $5}'
```

### Statistikk (Instant Queries)
Du kan også spørre om *mengden* data i stedet for innholdet.
**Volum (Linjer siste 10 min):**

```bash
logcli instant-query 'count_over_time({job="varlogs"}[10m])'
```

**Feilrate (Antall feil per sekund akkurat nå):**
```bash
logcli instant-query 'rate({job="varlogs"} |= "error" [1m])'
```

