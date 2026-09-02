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

## Fatto

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
