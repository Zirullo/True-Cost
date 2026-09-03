# 08 · Domande aperte

Da chiarire quando serve. Non bloccano il lavoro, ma cambiano le scelte.

## Prodotto

- **Effetto velocità**: procedurale, video, o ibrido? → è la prossima decisione,
  opzioni in [07-roadmap.md](07-roadmap.md)
- **BEV**: quando torna, e in che forma — switcher, seconda pagina, o vista unica
  che si adatta al tipo di vettura?
- ~~**Pubblico della demo**: portatile o link?~~ → **risolta il 2026-09-03**: è
  distribuita come link (https://zirullo.github.io/True-Cost/). Quindi gira su
  macchine che non controlliamo: ogni dipendenza da un servizio esterno va sempre
  con il suo fallback (vedi [05-dati-esterni.md](05-dati-esterni.md))
- **Il repo è pubblico**, e con esso questo second-brain. Va bene, o le note vanno
  spostate in un repo privato? Da decidere prima di scriverci dentro numeri
  Stellantis non ancora comunicabili
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
- **Consumo al minimo**: oggi a vettura ferma il costo istantaneo è 0. Un motore
  acceso e fermo consuma ~0.6–0.8 L/h — mostrarlo sarebbe un dettaglio molto
  "True Cost" (il costo di stare fermi accesi)
- **Curva consumo**: ereditata dal simulatore precedente. Su quale vettura vogliamo
  tararla davvero?

## Fedeltà visiva

- La foto ha **due "160"** sul quadrante (artefatto dell'immagine originale); noi
  usiamo la scala corretta 0–240 a passo 20. Confermato che va bene?
- I valori iniziali scenografici della foto non sono coerenti fra loro. Teniamo
  l'aggancio visivo al riferimento o partiamo da un viaggio a zero?
