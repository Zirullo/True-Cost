# 02 · Architettura di `index.html`

File unico, nessuna dipendenza esterna a parte la foto e una chiamata HTTP ai prezzi.
Funziona aprendolo con doppio click.

## I layer della scena

```
#shell  (wrapper, serve solo ad ancorare i pedali al bordo basso della scena)
 ├─ #regia  la barra di regia: lo switch ICE / BEV, e nient'altro.
 │          Sta FUORI dallo stage di proposito - vedi sotto
#stage  (aspect-ratio --scenew/--stageh = 1720/791, max-width 1720px)
 ├─ z1  #windshield   <canvas id="road">  la strada, su TUTTI i 1720
 ├─ z2  #ext          x 1135→1720: l'ultima colonna della foto stirata e sfocata
 ├─ z3  #interior     la foto (x 0–1147), col vetro ritagliato via CSS mask
 ├─ z4  #dashext      <svg viewBox="1147 0 573 641">  la plancia disegnata
 ├─ z5  #cluster      <svg viewBox="0 0 1147 641">  il quadro strumenti live
 ├─ z6  #stack        il gruppo display centrale (HTML, inclinato 9°)
 └─ z7  #refuel-prompt
```

Tutto scala insieme perché lo stage ha un aspect-ratio fisso e ogni layer usa lo
stesso sistema di coordinate della foto: **1 unità = 1 pixel dell'immagine
originale**. La foto e il cluster restano **larghi 1147 e ancorati a sinistra**:
allargare la scena non li ha toccati.

### `--scenew`, la larghezza della scena

`--scenew:1720` nel foglio di stile è l'unica fonte di verità: la usano
`aspect-ratio`, il `padding-top` che appoggia il cockpit al fondo pagina, la
posizione dei layer nuovi, e il canvas della strada che la rilegge in JS
(`SW`). Il **punto di fuga resta VPX 575**, quello della foto: la carreggiata è
decentrata a sinistra nell'inquadratura larga, ed è giusto così.

### `#regia`, la barra che non è il cockpit

Sopra la scena, alta `--regiah` (42px, sottratta dal `padding-top` che appoggia
tutto al fondo pagina). Contiene **solo lo switch del veicolo**, e sta lì proprio
perché non appartiene all'auto: nessuna vettura ha un bottone che trasforma un
serbatoio in una batteria. Tenerla grigia e piatta, fuori dal vetro, è il modo di
non mentire sul confine fra la demo e il prodotto.

La **tariffa di casa** ci stava, e ne è stata tolta: è un prezzo che il guidatore
paga davvero, non un comando di regia, e la sua casa è il pannello *Chargers*
sul display — accanto ai prezzi contro cui va confrontata. Vedi
[06-archivio-bev.md](06-archivio-bev.md).

### `PT`, i profili di powertrain

Un oggetto per motorizzazione (`PT.ice`, `PT.bev`, il PHEV verrà), e `car()` che
restituisce quello attivo secondo `state.vehicle`. Dentro un profilo sta tutto
ciò che i due veicoli non condividono: **curva di consumo** (`per100`), unità ed
etichette del cluster, capacità e livello di partenza, prezzo iniziale, consumo
da fermo (`idleRate`), se ha un **cambio** (`hasGears`), e per l'elettrico i
parametri della rigenerazione.

Il punto è che `update()` e `render()` **non si biforcano**: calcolano `stepU`,
l'energia del passo nell'unità del profilo, e da lì discendono costo del viaggio,
livello e €/km per entrambi. Un terzo veicolo è un terzo oggetto, non un terzo
ramo.

`garage` conserva lo stato di ciascun veicolo mentre non lo si guida (`CARRY`
elenca i campi che viaggiano), così lo switch è una porta fra due auto e non un
interruttore su una sola. Al boot lo stato ICE di apertura — quello scenografico
della foto, 128 km/h e un viaggio già di 48 km — viene riposto **com'è** invece
di essere ricostruito da zero.

### Layer 1 — `#windshield`

Contiene un solo `<canvas id="road">`, ridisegnato a ogni frame. La maschera del
layer sopra lo ritaglia nella forma del vetro, quindi il canvas puo' disegnare
liberamente oltre i bordi. Dettaglio del modulo: [09-strada.md](09-strada.md).

### Layer 2b — `#ext` + `#dashext` + `#stack` (la plancia che la foto non ha)

Da x 1147 in poi non esistono pixel fotografici: quella metà destra è
ricostruita. Vedi [11-plancia-estesa.md](11-plancia-estesa.md).

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
| `#ext` | la base colore della plancia estesa (foto stirata + sfocata) |
| `#dashext` | l'SVG della plancia disegnata: cruscotto, trim, bocchetta, carbonio |
| `#stack` `#bezel` `#glass` | il gruppo display centrale (la fila clima `#hvac` non c'è più) |
| `#app` | l'app True Cost sul display: `.ap-tab`, `#pane-history`, `#pane-map`, `#ap-canvas` |
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
