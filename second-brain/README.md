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
| [06-archivio-bev.md](06-archivio-bev.md) | Modello BEV rimosso dalla UI ma conservato | Quando reintrodurremo l'elettrico |
| [07-roadmap.md](07-roadmap.md) | Prossimi passi, a partire dall'effetto velocità | A inizio sessione |
| [08-domande-aperte.md](08-domande-aperte.md) | Cose non ancora decise / da verificare | Quando serve una decisione tua |
| [09-strada.md](09-strada.md) | La strada procedurale nel parabrezza: prospettiva, curve, traffico | Per ritoccare l'effetto velocità |
| [10-pubblicazione.md](10-pubblicazione.md) | Dove vive la demo, come si pubblica, come si torna indietro | Per aggiornare il link pubblico o recuperare una versione |

## File del progetto

```
d:\Cowork\Simulator\
├─ index.html                      ← il simulatore attuale (design della foto)
├─ True-Cost project picture.jpg   ← immagine di riferimento del design (1147×641)
├─ second-brain\                   ← queste note
├─ .gitignore                      ← tiene i backup manuali fuori dal repo
│
│  solo su disco, non su GitHub:
├─ OLD index.html                  ← versione precedente, doppio pannello ICE/BEV su canvas
├─ index - Copia*.html             ← backup manuali, sostituiti dalla cronologia git
└─ BCK\
```

## Dove vive

Demo live: **https://zirullo.github.io/True-Cost/** — sempre questo indirizzo, si
aggiorna a ogni `git push`. Repo pubblico: **https://github.com/Zirullo/True-Cost**.
Versione marcata corrente: **`v1.0`**. Tutto il resto in
[10-pubblicazione.md](10-pubblicazione.md).

> Il repo è pubblico: **queste note sono leggibili da chiunque abbia il link.**

## Stato in una riga

Simulatore ICE fedele alla foto di riferimento, con consumi in tempo reale guidati
dalla velocità, prezzi carburante live e una **strada procedurale nel parabrezza** che
scorre alla velocità reale della vettura (vedi [09-strada.md](09-strada.md)). Si guida
tenendo premuti i **pedali** sul bordo del cockpit (o le frecce), e ogni 2 km si
incontra una **stazione** il cui prezzo si legge sul cartellone e sul cluster.
