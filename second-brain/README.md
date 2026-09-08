# True Cost — Second Brain

Base di conoscenza del progetto **True Cost** (Star\*Up 2026 · Stellantis).
Punto di ingresso: leggi questo file, poi salta alla nota che ti serve.

> Regola: ogni volta che prendiamo una decisione, cambiamo una formula o scopriamo
> un dato esterno, la nota corrispondente va aggiornata. Il codice dice *come*,
> queste note dicono *perché*.

## Mappa delle note

| Nota | Contenuto | Quando serve |
|---|---|---|
| [00-progetto.md](00-progetto.md) | Cos'è True Cost, il pitch, chi lo guarda, stato attuale | Onboarding, pitch, presentazioni |
| [01-decisioni.md](01-decisioni.md) | Log decisionale (cosa abbiamo scelto e perché) | Prima di rimettere in discussione una scelta |
| [02-architettura.md](02-architettura.md) | Come è fatto `index.html`: i 3 layer, gli id, il flusso | Prima di toccare il codice |
| [03-mappa-design.md](03-mappa-design.md) | Coordinate di ogni elemento del cluster sulla foto | Per spostare/aggiungere elementi grafici |
| [04-modello-costi.md](04-modello-costi.md) | Fisica e formule: giri → velocità → consumo → €/km | Per cambiare la calibrazione |
| [05-dati-esterni.md](05-dati-esterni.md) | API prezzi carburante, fonti dati, fallback | Quando i prezzi live non arrivano |
| [06-archivio-bev.md](06-archivio-bev.md) | L'elettrico: curva su carico stradale, rigenerazione, tariffa di casa | Per ritarare il BEV |
| [07-roadmap.md](07-roadmap.md) | Prossimi passi, a partire dall'effetto velocità | A inizio sessione |
| [08-domande-aperte.md](08-domande-aperte.md) | Cose non ancora decise / da verificare | Quando serve una decisione tua |
| [09-strada.md](09-strada.md) | La strada procedurale nel parabrezza: prospettiva, curve, traffico | Per ritoccare l'effetto velocità |
| [10-pubblicazione.md](10-pubblicazione.md) | Dove vive la demo, come si pubblica, come si torna indietro | Per aggiornare il link pubblico o recuperare una versione |
| [11-plancia-estesa.md](11-plancia-estesa.md) | La metà destra disegnata a mano e il display centrale | Per ritoccare la plancia estesa o lavorare sull'app |
| [12-full-hybrid.md](12-full-hybrid.md) | Il full hybrid: curva del solo motore, tampone, e-CVT | Per ritarare l'ibrido o capire perché il km si ferma a zero |
| [13-phev.md](13-phev.md) | Il plug-in: due energie in un viaggio, e le cifre del mese per tutti e quattro | Per ritarare il plug-in o citare un numero |

## File del progetto

```
d:\Cowork\Simulator\
├─ index.html                      ← il simulatore attuale
├─ Green_Screen.jpeg               ← **la lastra**: la plancia intera, parabrezza in
│                                    verde croma (1525×688). Serve alla demo: va nel repo
├─ True_Cost_cockpit_base.jpeg     ← il rendering di riferimento: da qui vengono le
│                                    coordinate del display centrale
├─ True-Cost project picture.jpg   ← la foto vecchia (1147×641). Non è più a schermo,
│                                    ma resta il **righello**: 1 unità = 1 suo pixel
├─ second-brain\                   ← queste note
├─ .gitignore                      ← tiene i backup manuali fuori dal repo
│
│  solo su disco, non su GitHub:
├─ OLD index.html                  ← versione precedente, doppio pannello ICE/BEV su canvas
├─ index - Copia*.html             ← backup manuali, sostituiti dalla cronologia git
├─ mockup-plancia-estesa.html      ← la prova statica con cui è stata approvata la plancia estesa
└─ BCK\
```

## Dove vive

Demo live: **https://zirullo.github.io/True-Cost/** — sempre questo indirizzo, si
aggiorna a ogni `git push`. Repo pubblico: **https://github.com/Zirullo/True-Cost**.
Versione marcata corrente: **`v1.0`**. Tutto il resto in
[10-pubblicazione.md](10-pubblicazione.md).

> Il repo è pubblico: **queste note sono leggibili da chiunque abbia il link.**

## Stato in una riga

Simulatore fedele alla foto di riferimento con **quattro motorizzazioni** —
**ICE · HEV · PHEV · BEV**, che si cambiano con «**V**» e tengono ciascuna il proprio
viaggio, serbatoio e ultimo prezzo pagato. Consumi in tempo reale guidati dalla
velocità, prezzi carburante live, e una **strada procedurale nel parabrezza** che
scorre alla velocità vera della vettura ([09-strada.md](09-strada.md)). Si guida
tenendo premuti i **pedali** sul bordo del cockpit (o le frecce), e ogni 2 km si
incontra una **stazione** il cui cartellone dice **due** prezzi, il litro e il kWh.

Dal **2026-09-08** la plancia è **una sola fotografia**, dal montante sinistro al
lato opposto: la metà destra non è più ricostruita a mano. Sopra ci sta il
**display centrale**, dove gira l'**app True Cost** in **cinque viste** —
*Cost history* (il viaggio in corso e un mese di viaggi passati, guidato da tutte
e quattro le auto), *Pump map* (le stazioni intorno, e quelle **lungo la strada
sono i cartelloni veri** del parabrezza; è la vista su cui la demo si apre),
*Price history*, *Report* (la nota spese per il fleet manager) e *Trip management*
(dove finisce un viaggio) — vedi [11-plancia-estesa.md](11-plancia-estesa.md).

Dal **2026-09-07** il progetto è considerato completo: da qui in poi rifiniture.
