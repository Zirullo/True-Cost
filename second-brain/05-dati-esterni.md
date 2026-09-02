# 05 · Dati esterni

## Prezzi benzina (in uso nel simulatore)

```
https://prezzi-carburante.onrender.com/api/distributori
  ?latitude=41.9028&longitude=12.4964&distance=50&fuel=benzina&results=100
```

- Coordinate attualmente **fisse su Roma** (41.9028, 12.4964), raggio 50 km
- Risposta: array di distributori, campo `prezzo`
- Il simulatore filtra i valori fuori range (< 0.8 o > 4 €/L), scarta se restano
  meno di 3 stazioni, e fa la **media**
- Timeout 12 s, con fallback silenzioso ai prezzi demo (`refuelPrice` 1.91,
  `marketPrice` 1.85 €/L)
- Verificato funzionante il 2026-09-02: 50 stazioni, media 1.850 €/L

**Come vengono usati i prezzi:**

| Valore | Da dove | Uso |
|---|---|---|
| `state.marketPrice` | media live (fallback 1.85) | **non si vede a schermo**: è il centro attorno a cui si distribuiscono i prezzi delle stazioni sulla strada |
| Local Price | prezzo della **prossima stazione** = `marketPrice ± 0.15` | confronto + il cartellone nel parabrezza |
| Last Refuel Price | media live − 0.19 €/L | **base del calcolo del costo** |

Dal 2026-09-02 *Local Price* non è più la media nuda: è il prezzo della stazione
che stai per incontrare, con la distanza che scala. La media live è diventata il
**centro della distribuzione** (`ST_SPREAD` = ±0.15 €), non un numero mostrato.
Dettagli sulle stazioni: [09-strada.md](09-strada.md).

Il −0.19 è una **simulazione**: finge che il pieno sia stato fatto giorni fa a un
prezzo più basso. In un veicolo reale questo dato arriverebbe dallo storico
rifornimenti dei Connected Services. È il numero da cambiare se vuoi raccontare lo
scenario inverso (pieno fatto quando costava di più).

⚠️ Servizio di terze parti su hosting gratuito: può essere lento al primo colpo
(cold start) o non rispondere. Il fallback esiste apposta. Se il progetto va in
demo dal vivo, valuta di **congelare gli ultimi prezzi noti** in un file.

## Fonti citate nel progetto (non ancora integrate)

- **MIMIT open data** — dati prezzi carburante ufficiali italiani, è la fonte a
  monte del servizio qui usato
- **TomTom Fuel Prices API** — nell'ecosistema Stellantis, aggiornamento ~10 min:
  è la strada "seria" per la produzione
- **TomTom EV Charging API** — equivalente per le colonnine, servirebbe per il BEV
- **Free2Move** — prezzi ricarica fast, usati come confronto nel pannello BEV
  ([06-archivio-bev.md](06-archivio-bev.md))

## Cosa servirebbe da un veicolo vero

| Dato | Da dove |
|---|---|
| Velocità, giri, consumo istantaneo | bus veicolo |
| Livello serbatoio | sensore serbatoio |
| Prezzo e data dell'ultimo pieno | storico rifornimenti / Connected Services |
| Posizione (per il prezzo locale) | GPS |
| Prezzi delle stazioni vicine | API prezzi carburante |

Nessuno di questi richiede hardware aggiuntivo: è il punto centrale del pitch.
