# 06 · Archivio BEV

Il pannello elettrico è stato rimosso dalla UI (vedi
[01-decisioni.md](01-decisioni.md)), ma il modello resta valido e **vive in
`OLD index.html`**. Questa nota serve a non doverlo ricostruire da zero quando lo
riprenderemo.

## Il concetto BEV

Stesso messaggio dell'ICE — quanto costa un km — con due differenze che lo rendono
più interessante, non meno:

1. Il costo si calcola sulla **tariffa domestica** impostata dall'utente
   (es. 0.25 €/kWh), non su un prezzo di mercato
2. In **frenata rigenerativa il costo per km diventa negativo**: l'auto ti sta
   restituendo energia. È l'effetto più efficace della demo

Il prezzo della colonnina fast più vicina resta visibile come confronto: rende
tangibile quanto costa ricaricare fuori casa (nel vecchio simulatore, 0.59 €/kWh
contro 0.25 domestici → +136%).

## Formule conservate

**In trazione** (potenza positiva), calibrata su Jeep Avenger BEV 54 kWh:

```js
kWh100 = 0.00055·v² + 0.030·v + 13.0        // kWh / 100 km
kwhKm  = (kWh100 / 100) × fattoreRegen
eurKm  = kwhKm × tariffaCasa
```

Fattore regen sul consumo in marcia: `off 1.00 · low 0.92 · high 0.82`.

**In rigenerazione** (potenza negativa):

```js
efficienza = { off: 0, low: 0.55, high: 0.72 }[modoRegen]
kwhKm = -(|kW| × efficienza / max(10, v)) × 0.5      // negativo
eurKm = kwhKm × tariffaCasa                          // negativo
```

Velocità: in trazione `clamp(|kW| × 1.12 + 30, 10, 160)`; in rigenerazione decelera
di 0.8 km/h per step (× Time Warp).

**Stato di carica**: batteria 54 kWh, partenza 78%, `ΔSoC = kWh consumati / 54 × 10`
(risale in rigenerazione).

## Se lo reintroduciamo nel nuovo design

Il layout della foto si mappa quasi 1:1:

| Elemento ICE | Equivalente BEV |
|---|---|
| Quadrante giri | potenza kW (con zona negativa = regen) |
| CONSUMED · litri | kWh consumati |
| Last Refuel Price | tariffa domestica €/kWh |
| Local Price | colonnina fast più vicina |
| Fuel Level E→F | stato di carica % |
| km/L | kWh/100 km |

Il numero grande €/km resta identico — e diventa **verde quando è negativo**.

Da decidere prima di rifarlo: switcher tra le due viste, oppure due pagine separate.
Vedi [08-domande-aperte.md](08-domande-aperte.md).
