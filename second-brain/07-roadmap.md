# 07 · Roadmap

## Poi

- **Fare il pieno**: oggi superare una stazione è solo scenografia. Un rifornimento
  che aggiorna *Last Refuel Price* col prezzo di **quella** stazione chiuderebbe il
  cerchio: la scelta di dove fermarsi si vedrebbe negli €/km di tutto il viaggio dopo
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

- **Il debito dei nomi**: `tripLiters`, `tankLiters`, `instantL100`, `t.l100`
  contengono kWh sotto il profilo elettrico. Resta **tutto** del PHEV: il full
  hybrid non lo ha forzato, perché compra solo litri e il suo tampone sta fuori
  dai libri — [06-archivio-bev.md](06-archivio-bev.md),
  [12-full-hybrid.md](12-full-hybrid.md)
- **Il PHEV**, quarto profilo in `PT` — ed è quello che forzerà la mano sui nomi,
  perché brucia benzina **e** kWh nello stesso viaggio
- **Il taglio in rilascio**: in coasting senza freno tutti e tre pagano ancora il
  consumo di crociera, mentre un motore vero taglia l'iniezione. È il simmetrico
  del costo dell'accelerazione, che è stato fatto —
  [12-full-hybrid.md](12-full-hybrid.md)
- **Il pieno visto dall'app**: oggi premere «R» aggiorna il cluster ma l'app non se
  ne accorge, se non nel prezzo del serbatoio. Un evento nel grafico («qui hai fatto
  il pieno a 1.78») legherebbe la scelta della stazione al costo dei km successivi
- **Portare la mappa sul percorso**: pin coerenti con le stazioni che si incontrano
  davvero nel parabrezza — la scelta contraria, e il perché, in
  [01-decisioni.md](01-decisioni.md)

## Fatto

- ✅ **Il full hybrid, e un'auto che il risparmio se lo guadagna** (2026-09-07):
  terza motorizzazione in `PT`, barra di regia **ICE · HEV · BEV** e «V» che
  cicla. La curva è **solo il motore**; sopra c'è un tampone da 0.50 kWh che si
  riempie in frenata e si svuota in ripresa, con la barretta che lo mostra nel
  pannello. E-CVT: i giri seguono la potenza e cadono a **zero** in elettrico,
  con **EV · HV · CHG** al posto della marcia. Da fermo il motore è spento — zero
  contro 0.7 L/h. A 128 costanti −10%, in stop-and-go −59%, e **il km si ferma a
  zero, mai sotto**. Con esso `liquid` separato da `hasGears`, e
  **l'accelerazione che si paga su tutti e tre**, che ha messo d'accordo il
  modello vivo con i suoi stessi diciotto viaggi —
  [12-full-hybrid.md](12-full-hybrid.md), [01-decisioni.md](01-decisioni.md)

- ✅ **Un cartello, due prezzi** (2026-09-07): il cartellone delle stazioni
  porta il litro **e** il kWh, una riga per uno e dello stesso peso, con icona
  pompa e icona batteria, doppio marchio (le pompe e le colonnine dello stesso
  piazzale) e il luogo in testa. Niente sul pannello legge il veicolo: dice la
  stessa cosa sotto un serbatoio e sotto una batteria, ed è il confronto fra le
  due righe la cosa da vedere. Il *Local Price* del cluster continua a mostrare
  solo l'energia che quella macchina compra — [09-strada.md](09-strada.md),
  [01-decisioni.md](01-decisioni.md)

- ✅ **Lo stesso mese, guidato due volte** (2026-09-06): ogni viaggio porta
  `t.veh`, il log e il Report mostrano solo la metà della vettura in uso, e i
  diciotto viaggi hanno un gemello elettrico sulle stesse tratte col consumo
  derivato dalla curva viva. Sullo stesso mese e sugli stessi 1194 km:
  **€ 151.61 contro € 131.84**, con l'ordine dei viaggi che si rovescia fra
  autostrada e città. Con essi una serie storica dei prezzi elettrici
  **derivata e dichiarata tale**, e una velatura di colore sul cockpit che si
  ferma al vetro — [01-decisioni.md](01-decisioni.md)

- ✅ **Il BEV, e uno switch fra due veicoli** (2026-09-06): `index.html` ha due
  motorizzazioni e una barra di regia sopra la scena per passare dall'una
  all'altra («V»). Profili di powertrain in `PT`, `garage` che tiene i due mondi
  separati, giri → kW con segno, serbatoio → carica, totem in €/kWh, tariffa di
  casa regolabile — e in frenata **il costo del chilometro va sotto zero, in
  verde**. L'app resta da adattare — [06-archivio-bev.md](06-archivio-bev.md),
  [01-decisioni.md](01-decisioni.md)

- ✅ **Trip management, e il costo di stare fermi accesi** (2026-09-05): la quinta
  vista decide **dove finisce un viaggio** — chiave, motore spento per N minuti,
  fermo per N minuti, o solo a mano — con il conto alla rovescia che si guarda
  girare, KEEP OPEN, SPLIT HERE e la cartolina di fine viaggio (business /
  privato / commute, MERGE, DISCARD). Il viaggio salvato **entra davvero** in
  Cost history e nel Report. Con essa il simulatore ha un motore da spegnere e
  un consumo al minimo — [11-plancia-estesa.md](11-plancia-estesa.md),
  [01-decisioni.md](01-decisioni.md)

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
