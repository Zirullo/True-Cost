# 02 · Architettura di `index.html`

File unico, nessuna dipendenza esterna a parte la foto e una chiamata HTTP ai prezzi.
Funziona aprendolo con doppio click.

## I tre layer della scena

```
#shell  (wrapper, serve solo ad ancorare i pedali al bordo basso della scena)
#stage  (aspect-ratio 1147/641, max-width 1147px)
 ├─ z1  #windshield   <canvas id="road">  la strada procedurale
 ├─ z2  #interior     la foto, col vetro ritagliato via CSS mask
 └─ z3  #cluster      <svg viewBox="0 0 1147 641">  il quadro strumenti live
```

Tutto scala insieme perché lo stage ha un aspect-ratio fisso e l'SVG usa lo stesso
sistema di coordinate della foto: **1 unità SVG = 1 pixel dell'immagine originale**.

### Layer 1 — `#windshield`

Contiene un solo `<canvas id="road">`, ridisegnato a ogni frame. La maschera del
layer sopra lo ritaglia nella forma del vetro, quindi il canvas puo' disegnare
liberamente oltre i bordi. Dettaglio del modulo: [09-strada.md](09-strada.md).

### Layer 2 — `#interior`

La foto come `background`, con una **mask SVG generata a runtime** (primo IIFE dello
script) che buca l'area del vetro. La maschera è un rettangolo pieno con un foro
`fill-rule="evenodd"`; il foro segue l'array `edge`, ~56 punti tracciati sul bordo
plancia/vetro dell'immagine.

Se la giunzione stona, si toccano solo quei punti: `[x, y]` in coordinate immagine,
da destra verso sinistra.

### Layer 3 — `#cluster`

Un solo SVG, statico nella struttura, dinamico solo nei valori. Contiene, in ordine
di disegno: schermo (clip + gradiente + texture), barra STELLANTIS, cartello 130,
titolo TRUE COST e "?", tachimetro, colonna LIVE CONSUMPTION, colonna
DISTANCE/CONSUMED, box TOTAL TRIP COST + RESET, pannello VEHICLE STATUS.

Coordinate di ogni elemento: [03-mappa-design.md](03-mappa-design.md).

## Il loop

```
loop(now) → update() → render() → road.frame(speed, dt) → requestAnimationFrame(loop)
```

Il `dt` reale (secondi fra un frame e l'altro, limitato a 50 ms per sopravvivere ai
cambi di scheda) serve **solo alla strada**: la fisica dei consumi usa ancora il suo
passo fisso di 0.016 s.

- **`update()`** — solo fisica e accumulatori: giri, velocità, consumo istantaneo,
  km/litri/euro del viaggio, livello serbatoio. Formule in
  [04-modello-costi.md](04-modello-costi.md).
- **`render()`** — scrive `textContent` sui nodi SVG, chiede a `road.station()`
  prezzo e distanza della prossima stazione, aggiorna la larghezza della
  barra carburante, accende/spegne le tacche del quadrante, attenua lo storico
  viaggi quando la vettura è in moto.

Nessun ridisegno completo: l'SVG resta lo stesso, cambiano solo testi e classi.

## Id da conoscere

| Id | Cosa |
|---|---|
| `#windshield` | l'area del parabrezza |
| `#road` | il canvas della strada procedurale |
| `#interior` | la foto mascherata |
| `#cluster` | l'SVG del quadro strumenti |
| `#spd` `#rpmTxt` | velocità e giri sul quadrante |
| `#eurKm` | il numero grande €/km |
| `#dist` `#cons` `#total` | distanza, litri, costo totale |
| `#refuel` `#local` | i due prezzi in VEHICLE STATUS |
| `#localDist` | quanto manca alla prossima stazione, accanto a Local Price |
| `#fuelbar` | la barra livello carburante (cambia `width`) |
| `#tick-group` `#num-group` | tacche e numeri del quadrante, generati in JS |
| `#help-hit` `#reset-hit` | zone cliccabili dentro l'SVG ("?" e RESET) |
| `#deck` `#history` | comandi e storico viaggi, HTML sotto la scena |
| `#pedals` | i due pedali a cavallo fra scena e deck (`#pedal-gas`, `#pedal-brake`) |
| `#pedal-hint` | la scritta che invita a tenere premuto, sparisce al primo uso |

## Comandi (fuori dalla scena)

- **Slider Speed** → `state.targetSpeed` (0–240 km/h): la velocità *richiesta*,
  non quella reale — ci si arriva con la rampa di accelerazione e il cambio
- **Pedali ↑ / ↓** → finché sono tenuti premuti chiedono `PEDAL_LEAD` (8 km/h) in
  più o in meno della velocità **reale**, da mouse/touch o dalle frecce della
  tastiera: il divario non si chiude mai, quindi la vettura tira per tutto il
  tempo. **Al rilascio la velocità richiesta viene agganciata a quella raggiunta**,
  quindi non si tocca nulla e si mantiene l'andatura. Lo slider Speed segue.
- **Reset trip** → chiude il viaggio, lo scrive nello storico, azzera gli accumulatori
  (stesso effetto del RESET dentro il cluster)

## Fedeltà: come sono state ricavate le coordinate

Sovrapponendo alla foto una griglia da 50 px e leggendo la posizione di ogni
elemento. Il bordo del parabrezza è stato invece rilevato **a scansione**: per ogni
colonna di pixel, la prima riga in cui la luminanza scende sotto ~70 per 8 pixel
consecutivi = inizio plancia. Gli artefatti (auto scure sulla strada, riflessi sul
cruscotto) sono stati corretti a mano.
