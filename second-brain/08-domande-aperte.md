# 08 · Domande aperte

Da chiarire quando serve. Non bloccano il lavoro, ma cambiano le scelte.

## Prodotto

- **Effetto velocità**: procedurale, video, o ibrido? → è la prossima decisione,
  opzioni in [07-roadmap.md](07-roadmap.md)
- ~~**BEV**: quando torna, e in che forma — switcher, seconda pagina, o vista unica
  che si adatta al tipo di vettura?~~ → **risolta il 2026-09-06**: **vista unica
  che si adatta**, con profili di powertrain in un solo `index.html` e uno switch
  in una barra di regia sopra la scena. Non due pagine: strada, app, mappa e
  report sarebbero andati tenuti allineati a mano su tre copie da 5000 righe.
  Con essa sono cadute altre tre scelte — la rigenerazione è **automatica sul
  freno** (nessun selettore da spiegare), la ricarica si fa **solo alla
  colonnina**, e i due veicoli tengono **viaggi e storici separati**: nessun
  viaggio è metà a benzina e metà elettrico
- ~~**Pubblico della demo**: portatile o link?~~ → **risolta il 2026-09-03**: è
  distribuita come link (https://zirullo.github.io/True-Cost/). Quindi gira su
  macchine che non controlliamo: ogni dipendenza da un servizio esterno va sempre
  con il suo fallback (vedi [05-dati-esterni.md](05-dati-esterni.md))
- ~~**Il repo è pubblico**, e con esso queste note: privato o no?~~ → **decisa il
  2026-09-03**: restano pubbliche. Non c'è niente di segreto — formule, coordinate,
  decisioni tecniche, prezzi da API aperte. Vale però la regola che ne discende:
  **queste note sono un documento pubblico**, quindi niente numeri, nomi o
  affermazioni su terzi che non siano già comunicabili. Un repo pubblico è
  scopribile senza condividerne il link (profilo, ricerca, feed eventi di GitHub),
  e la storia git è permanente: quello che entra in un commit non si toglie
  cancellandolo dopo
- **Lingua**: la UI è in inglese, queste note in italiano. Va bene così?

## Dati e affermazioni

- **"Stellantis Connect Fleet ha confermato interesse"** — presente nei testi del
  simulatore precedente. Da riverificare prima di ripeterlo in un pitch o in un
  documento pubblico
- **Prezzo dell'ultimo pieno**: oggi è simulato come `prezzo locale − 0.19 €/L`.
  Quale scenario vogliamo raccontare — pieno fatto quando costava meno (mostra un
  risparmio) o quando costava di più?
- **Posizione**: fissa su Roma. Geolocalizzare o lasciare un default controllato per
  avere numeri prevedibili in demo?

## Modello

- **Mappa giri → velocità lineare**: nella realtà dipende dalla marcia. Vale la pena
  simulare un cambio a 6 rapporti (più credibile per un pubblico tecnico) o la
  semplicità è meglio?
- ~~**Consumo al minimo**: oggi a vettura ferma il costo istantaneo è 0~~ →
  **risolta il 2026-09-05**: un motore acceso e fermo brucia `IDLE_LH = 0.7` L/h,
  il costo del viaggio lo porta, e Trip management ne tiene il conto in euro
  (`€ STANDING`) e lo dice sulla cartolina di fine viaggio: *«fermo con il motore
  acceso per 3:34, € 0.08 — l'1 % di questo viaggio, su nessuna distanza»*. Gli
  €/km istantanei restano 0 a vettura ferma: a 0 km/h non sono definiti
- **Curva consumo**: ereditata dal simulatore precedente. Su quale vettura vogliamo
  tararla davvero? — **il BEV è stato staccato dall'eredità il 2026-09-07** e adesso
  sta su carico stradale fisico ([06-archivio-bev.md](06-archivio-bev.md)). L'ICE e
  i due ibridi restano quelli della foto, e reggono bene il confronto con la realtà
- **Rigenerazione solo col pedale del freno**: `update()` recupera energia
  unicamente quando `held.brake` è vero, e il pedale toglie **20 km/h al secondo** —
  così forte che il tetto dei 45 kW manda in calore la maggior parte di una
  frenata. Un'auto elettrica vera recupera soprattutto **in rilascio**, dove qui
  non torna niente. In autostrada non si vede; in città fa una differenza grande, ed
  è il motivo per cui il gemello elettrico dei diciotto viaggi ha un `BEV_C` tipato
  invece di essere guidato dal vivo. Vale la pena dare regen al rilascio?

## Fedeltà visiva

- La foto ha **due "160"** sul quadrante (artefatto dell'immagine originale); noi
  usiamo la scala corretta 0–240 a passo 20. Confermato che va bene?
- I valori iniziali scenografici della foto non sono coerenti fra loro. Teniamo
  l'aggancio visivo al riferimento o partiamo da un viaggio a zero?
