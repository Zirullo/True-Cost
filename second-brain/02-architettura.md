# 02 · Architettura di `index.html`

File unico, nessuna dipendenza esterna a parte la foto e una chiamata HTTP ai prezzi.
Funziona aprendolo con doppio click.

## I layer della scena

```
#shell  (wrapper, serve solo ad ancorare i pedali al bordo basso della scena)
 ├─ #regia  la barra di regia: lo switch ICE / BEV, e nient'altro.
 │          Sta FUORI dallo stage di proposito - vedi sotto
#stage  (aspect-ratio --scenew/--stageh = 1639.78/767.2, max-width 1720px)
 ├─ z1  #windshield   <canvas id="road">  la strada, su TUTTA la scena
 ├─ z3  #interior     la lastra Green_Screen.jpeg, intera, col vetro ritagliato
 │                    via CSS mask (z2 e z4 erano #ext e #dashext: non esistono più)
 ├─ z5  #cluster      <svg viewBox="0 0 1147 641">  il quadro strumenti live
 ├─ z6  #stack        il gruppo display centrale (HTML, inclinato 9°)
 └─ z7  #refuel-prompt
```

Tutto scala insieme perché lo stage ha un aspect-ratio fisso e ogni layer usa lo
stesso sistema di coordinate: **1 unità = 1 pixel della foto originale**
(`True-Cost project picture.jpg`, 1147 × 641). Quella foto non è più sullo
schermo — dal 2026-09-08 c'è la lastra intera — ma resta il **righello**: la
lastra è posata dentro quello spazio a `0.93`, e il cluster è rimasto largo 1147
e ancorato a sinistra. Vedi [11-plancia-estesa.md](11-plancia-estesa.md).

### `--scenew`, la larghezza della scena

`--scenew:1639.78` nel foglio di stile è l'unica fonte di verità (= 1525/0.93, la
lastra intera in unità di cockpit): la usano
`aspect-ratio`, la posizione dei layer nuovi, e il canvas della strada che la
rilegge in JS (`SW`). Il **punto di fuga resta VPX 575**, quello della foto: la
carreggiata è decentrata a sinistra nell'inquadratura larga, ed è giusto così.

> Fino al 2026-09-09 c'era anche un `padding-top` calcolato che spingeva tutto
> verso il fondo della pagina su finestre più alte del contenuto. Tolto: ora il
> cockpit comincia sempre dall'alto e lo spazio che avanza resta sotto, non
> sopra — vedi il log del 2026-09-09 in
> [01-decisioni.md](01-decisioni.md).

### `#regia`, la barra che non è il cockpit

Sopra la scena, alta `--regiah` (42px). Contiene lo switch del veicolo e il link
**Feedback** — un CTA ambrato, deliberatamente più vistoso del resto della barra
perché va notato e cliccato, non solo letto — e sta lì proprio perché nessuno dei
due appartiene all'auto: nessuna vettura ha un bottone che trasforma un serbatoio
in una batteria, né un modulo per lasciare un commento. Il resto della barra
resta grigio e piatto, fuori dal vetro, come modo di non mentire sul confine fra
la demo e il prodotto.

La **tariffa di casa** ci stava, e ne è stata tolta: è un prezzo che il guidatore
paga davvero, non un comando di regia, e la sua casa è il pannello *Chargers*
sul display — accanto ai prezzi contro cui va confrontata. Vedi
[06-archivio-bev.md](06-archivio-bev.md).

### `PT`, i profili di powertrain

Un oggetto per motorizzazione (`PT.ice`, `PT.hev`, `PT.bev`, il PHEV verrà), e
`car()` che restituisce quello attivo secondo `state.vehicle`. Dentro un profilo
sta tutto ciò che i veicoli non condividono: **curva di consumo** (`per100`),
unità ed etichette del cluster, capacità e livello di partenza, prezzo iniziale,
consumo da fermo (`idleRate`), quanto costa **guadagnare velocità** (`accCost`),
e per chi ce l'ha i parametri della rigenerazione e del tampone.

**Due bandiere, e vogliono dire cose diverse.** `liquid` riguarda i **libri**: le
unità sono litri e si fa il pieno a una pompa. `hasGears` riguarda la
**trasmissione**: ci sono rapporti discreti fra cui saltare. Erano la stessa
domanda finché le auto erano due; il full hybrid le separa, perché è un e-CVT che
compra benzina — [12-full-hybrid.md](12-full-hybrid.md).

Il punto è che `update()` e `render()` **non si biforcano**: calcolano `stepU`,
l'energia del passo nell'unità del profilo, e da lì discendono costo del viaggio,
livello e €/km per tutti. I due slot sotto al tachimetro non hanno più un
ternario: ogni profilo porta `subVal(state)` e `gearVal(state)` e risponde per sé
— giri e marcia, giri e modo ibrido, o kW firmati e `D`. Un terzo veicolo è un
terzo oggetto, non un terzo ramo, ed è andata davvero così.

`garage` conserva lo stato di ciascun veicolo mentre non lo si guida (`CARRY`
elenca i campi che viaggiano), così lo switch è una porta fra due auto e non un
interruttore su una sola. Al boot lo stato ICE di apertura — quello scenografico
della foto, 128 km/h e un viaggio già di 48 km — viene riposto **com'è** invece
di essere ricostruito da zero.

### Layer 1 — `#windshield`

Contiene un solo `<canvas id="road">`, ridisegnato a ogni frame. La maschera del
layer sopra lo ritaglia nella forma del vetro, quindi il canvas puo' disegnare
liberamente oltre i bordi. Dettaglio del modulo: [09-strada.md](09-strada.md).

### Layer 2b — `#stack` (il gruppo display)

L'unica cosa disegnata da noi sulla metà destra della plancia: il resto è
fotografia. `#ext` e `#dashext` — la spalmatura e la geometria SVG che ricostruivano
quella metà quando la foto finiva a x 1147 — sono state rimosse il 2026-09-08.
Vedi [11-plancia-estesa.md](11-plancia-estesa.md).

### Layer 2 — `#interior`

La lastra `Green_Screen.jpeg` (1525 × 688) come `background`, con una **mask SVG
generata a runtime** (primo IIFE dello script) che buca l'area del vetro. La
maschera è un rettangolo pieno con un foro `fill-rule="evenodd"`; il foro segue
l'array `edge`, **17 punti** che non sono tracciati a mano: sono l'ultima riga
verde di ogni colonna della lastra, semplificata e spinta 3 px dentro il cruscotto.

Se la giunzione stona, si toccano solo quei punti: `[x, y]` in **pixel della
lastra**, da sinistra verso destra.

### Layer 3 — `#cluster`

Un solo SVG, statico nella struttura, dinamico solo nei valori. Contiene, in ordine
di disegno: schermo (clip + gradiente + texture), barra STELLANTIS, cartello 130,
titolo TRUE COST e "?", tachimetro, colonna LIVE CONSUMPTION, colonna
DISTANCE/CONSUMED, box TOTAL TRIP COST + RESET, pannello VEHICLE STATUS.

Coordinate di ogni elemento: [03-mappa-design.md](03-mappa-design.md).

## Il loop

```
loop(now) → update() → render() → road.frame(speed, dt) → app.frame(dt) → requestAnimationFrame(loop)
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

- **`app.frame(dt)`** — il display centrale: campiona il costo ogni 200 m, muove le
  stazioni della mappa, e ridisegna testi (4 Hz) e canvas (~16 Hz) solo per la vista
  aperta — [11-plancia-estesa.md](11-plancia-estesa.md).

Nessun ridisegno completo: l'SVG resta lo stesso, cambiano solo testi e classi.

## Id da conoscere

| Id | Cosa |
|---|---|
| `#windshield` | l'area del parabrezza |
| `#road` | il canvas della strada procedurale |
| `#interior` | la lastra `Green_Screen.jpeg`, mascherata |
| `#cluster` | l'SVG del quadro strumenti |
| `#spd` `#rpmTxt` | velocità e giri sul quadrante |
| `#eurKm` | il numero grande €/km |
| `#dist` `#cons` `#total` | distanza, litri, costo totale |
| `#refuel` `#local` | i due prezzi in VEHICLE STATUS |
| `#localDist` | quanto manca alla prossima stazione, accanto a Local Price |
| `#fuelbar` | la barra livello carburante (cambia `width`) |
| `#tick-group` `#num-group` | tacche e numeri del quadrante, generati in JS |
| `#help-hit` `#reset-hit` | zone cliccabili dentro l'SVG ("?" e RESET) |
| `#stack` `#bezel` `#glass` | il gruppo display centrale (la fila clima `#hvac` non c'è più) |
| `#app` | l'app True Cost sul display: `.ap-tab`, `#pane-history`, `#pane-map`, `#ap-canvas` |
| `#hidden-controls` | lo slider della velocità, ritagliato a un pixel: lo scrivono pedali, frecce e cluster. È tutto ciò che resta del deck |
| `#pedals` | i due pedali, ancorati a `#shell` — non al deck, che non c'è più (`#pedal-gas`, `#pedal-brake`) |
| `#pedal-hint` | la scritta che invita a tenere premuto, sparisce al primo uso |

## Comandi (fuori dalla scena)

- **Slider Speed** → `state.targetSpeed` (0–240 km/h): la velocità *richiesta*,
  non quella reale — ci si arriva con la rampa di accelerazione e il cambio
- **Pedali ↑ / ↓** → finché sono tenuti premuti chiedono `PEDAL_LEAD` (8 km/h) in
  più o in meno della velocità **reale**, da mouse/touch o dalle frecce della
  tastiera: il divario non si chiude mai, quindi la vettura tira per tutto il
  tempo. **Al rilascio la velocità richiesta viene agganciata a quella raggiunta**,
  quindi non si tocca nulla e si mantiene l'andatura. Lo slider Speed segue.
- **Reset trip** → chiude il viaggio, lo scrive nello storico, azzera gli accumulatori.
  Dal 2026-09-07 **non ha più un bottone**: `hardReset()` è lì, il cablaggio lo cerca
  e non lo trova. La porta rimasta è nel pannello **Trips** sul display — *Split here*
  e *End trip*, che passano da `closeTrip()` e chiamano lo stesso `resetTrip()`
- **Tasti** → `E` motore, `V` veicolo, `R` rifornimento, `U` l'UFO. `E` è l'unico modo
  di spegnere il motore da quando la barra in fondo è sparita

## Fedeltà: come sono state ricavate le coordinate

Sovrapponendo alla foto una griglia da 50 px e leggendo la posizione di ogni
elemento. Il bordo del parabrezza è stato invece rilevato **a scansione**: per ogni
colonna di pixel, la prima riga in cui la luminanza scende sotto ~70 per 8 pixel
consecutivi = inizio plancia. Gli artefatti (auto scure sulla strada, riflessi sul
cruscotto) sono stati corretti a mano.
