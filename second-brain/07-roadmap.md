# 07 · Roadmap

## Poi

- **Fare il pieno**: oggi superare una stazione è solo scenografia. Un rifornimento
  che aggiorna *Last Refuel Price* col prezzo di **quella** stazione chiuderebbe il
  cerchio: la scelta di dove fermarsi si vedrebbe negli €/km di tutto il viaggio dopo
- **Reintrodurre il BEV** nel nuovo design ([06-archivio-bev.md](06-archivio-bev.md))
- **Robustezza prezzi live**: cache dell'ultimo valore noto, così una demo dal vivo
  non dipende da un servizio gratuito che dorme
- **Geolocalizzazione** al posto delle coordinate fisse su Roma
- **Storico viaggi**: oggi appare/scompare per opacità; nel cluster reale comparirebbe
  solo a veicolo fermo. Valutare se portarlo *dentro* lo schermo
- **Scenari da pitch**: pulsanti preset ("autostrada", "città", "traffico") che
  muovono lo slider da soli, per raccontare la storia senza toccare i comandi
- **Confronto esplicito** ICE vs BEV sullo stesso percorso: è il numero che colpisce
  di più chi guarda

## Prossimo passo

- **Il pieno visto dall'app**: oggi premere «R» aggiorna il cluster ma l'app non se
  ne accorge, se non nel prezzo del serbatoio. Un evento nel grafico («qui hai fatto
  il pieno a 1.78») legherebbe la scelta della stazione al costo dei km successivi
- **Portare la mappa sul percorso**: pin coerenti con le stazioni che si incontrano
  davvero nel parabrezza — la scelta contraria, e il perché, in
  [01-decisioni.md](01-decisioni.md)

## Fatto

- ✅ **La pump map e il parabrezza dicono lo stesso prezzo** (2026-09-04): sotto
  «lungo questa strada» ci sono i cartelloni veri, non stazioni inventate, e la
  demo si apre su quella vista — [11-plancia-estesa.md](11-plancia-estesa.md)

- ✅ **L'app True Cost sul display centrale** (2026-09-04): quattro viste
  cliccabili, *Cost history* (viaggio in corso dal vivo + un mese di viaggi
  passati), *Pump map* (stazioni colorate per prezzo, verde la più conveniente,
  entro 5 / 10 / 20 km o lungo la strada), *Price history* e **Report** (nota spese
  con divisione business / privato e IVA recuperabile, invio simulato verso email,
  CSV, portale di flotta, expense o ride-hailing); via i tasti clima per fare
  spazio — [11-plancia-estesa.md](11-plancia-estesa.md)

- ✅ Visuale allargata a 1720 fino al display centrale (2026-09-04) —
  [11-plancia-estesa.md](11-plancia-estesa.md)

- ✅ Alberi a bordo strada, quattro specie e nessuno uguale a un altro, per la
  sensazione di velocità (2026-09-02) — [09-strada.md](09-strada.md)

- ✅ Stazioni di servizio ogni 2 km con prezzo sul cartellone, e *Local Price* che
  mostra prezzo e distanza della prossima (2026-09-02) — [09-strada.md](09-strada.md)

- ✅ Pedali Accelerate/Brake sul bordo del cockpit, tenuti premuti o con le frecce
  (2026-09-02) — [01-decisioni.md](01-decisioni.md)
- ✅ Passata grafica sulla strada (2026-09-02): fondale a tre parallassi, nuvole,
  colore per profondità, grana e giunti sull'asfalto, new jersey con giunti e
  pannelli antiabbagliamento, guard-rail a W, cartelli, veicoli con sagoma e
  foschia — [09-strada.md](09-strada.md)
- ✅ Strada procedurale nel parabrezza (2026-09-02) — [09-strada.md](09-strada.md)
- ✅ Restyling fedele alla foto di riferimento (2026-09-02)
- ✅ Parabrezza isolato come layer vuoto e mascherato
- ✅ Cluster ricostruito in SVG con coordinate sull'immagine
- ✅ Consumi guidati dalla velocità, calibrati sui numeri della foto
- ✅ Prezzi carburante live con fallback
