# 11 · La plancia estesa e il display centrale

Dal 2026-09-04 la scena è larga **1720** invece di 1147. Da x 1147 in poi non
esistono pixel fotografici: quella parte è **ricostruita**. Perché lo abbiamo fatto
così — e cosa avevamo scartato — sta in [01-decisioni.md](01-decisioni.md).

Coordinate: le stesse della foto (`True-Cost project picture.jpg`, 1147 × 641).
La verticale va sempre 0–641, anche nei layer nuovi, così `--rise` può cambiare
senza trascinarsi dietro niente.

## I tre pezzi

### `#ext` — la continuità dei colori

Un `div` con **la foto stessa come sfondo**, scalata ×28.65 e allineata a destra:
di fatto le sue ultime **20 colonne** stirate su tutta la larghezza nuova, più un
`blur(7px)`. Continua da sola le bande orizzontali — colline, guard-rail,
cruscotto, alluminio, carbonio — con i colori esatti dell'immagine, e su quella
base si disegna la geometria.

| Proprietà | Valore | Perché |
|---|---|---|
| `background-size` | `5735% 117.6%` | 5735 % = ×28.65 in orizzontale (20 px → 573); 117.6 % rimette la **scala verticale a 1:1** (il div è alto 545 righe di foto, l'immagine 641) |
| `background-position` | `100% 100%` | allinea il bordo destro e il bordo basso della foto: in cima resta esposta la riga 96 |
| `clip-path` | `polygon(0 2.2%, 100% 0, 100% 100%, 0 100%)` | il bordo del parabrezza continua: **y 108 a x 1147** (2.2 % di 545), **y 96** all'estremo destro |
| `::after` | gradiente scuro verso destra | nella foto la luce cala allontanandosi dal guidatore |
| `left` | **1135**, non 1147 | `blur()` sfuma a trasparente il bordo dell'elemento stesso: partendo 12 px più a sinistra la sfumatura finisce **sotto la foto** invece di lasciare una cucitura nera. Per questo la foto sta a z3 e `#ext` a z2 |

**Se stona la giunzione**: si tocca `clip-path` (il bordo del vetro) o la
percentuale di `background-size` verticale — le due cose che allineano l'estensione
alla foto. Non allargare la colonna sorgente oltre ~25 px: entrerebbe la cornice
del cluster e comparirebbe una fascia scura.

### `#dashext` — la geometria disegnata

Un SVG `viewBox="1147 0 573 641"`, `preserveAspectRatio="none"`.

| Elemento | Geometria | Note |
|---|---|---|
| Ombra sotto il vetro | `1147,108 → 1720,94`, alta 42 | sfocata, stacca vetro e plancia |
| Cruscotto (scamosciato) | `1147,112 → 1720,96 → 1720,242 → 1147,270` | gradiente `#cowlG` |
| Griglia sbrinatore | `1180,122 → 1700,104`, alta ~46 | pattern `#defrostP` |
| Fascia in alluminio | `1147,302 → 1210,286 → 1700,264 → 1720,262` e ritorno a `1210,346 / 1147,366` | raccoglie il trim della foto, gradiente `#trimG` |
| Pannello in carbonio | da `1147,410 → 1720,390` fino a 641 | pattern `#cfxP` + velatura |
| Bocchetta d'aria | rect `1156,180` · 42 × 104 | fra cluster e display, 5 alette |
| Ombra portata | ellisse `1450,556` rx 238 ry 32 | posa il gruppo display sulla plancia |
| Alone freddo | ellisse `1450,310` rx 308 ry 222 | la luce dello schermo sulla plancia |

### `#stack` — il gruppo display

HTML, non SVG: dentro ci gira **UI vera**. Un solo `transform` inclina insieme
cornice e vetro.

```
left   1212 / 1720        (coordinate della scena)
width   476 / 1720
bottom  100 / 791         ancorato in basso: indipendente da --rise
height  404 / 791
transform: perspective(1500px) rotateY(-9deg)   origine a sinistra
```

Dentro: `#bezel` (cornice piano black, `padding` 2.3 % — **tutta** l'altezza) →
`#glass` (il vetro, col riflesso obliquo in `::after`) → `#app`, l'applicazione.

**La fila clima non c'è più.** Fino al 2026-09-04 sotto lo schermo c'era `#hvac`,
sette tasti finti alti il 20 % del gruppo: decorazione che rubava spazio all'unica
cosa che in questa demo deve essere leggibile. Rimossa, il vetro passa da 274 a
**368 righe** di altezza (+34 %) senza toccare l'inquadratura. L'ombra portata è
scesa a `cy 556` e la bocchetta si è stretta a 42 px per far posto al bordo
sinistro del gruppo.

**I 9 gradi**: compromesso fra la profondità della foto Jeep di riferimento e la
leggibilità dei numeri. Se l'app risultasse faticosa da leggere in pitch, il
numero da abbassare è solo quello.

## L'app dentro `#app`

Dal 2026-09-04 `#app` non è più un segnaposto: è l'applicazione True Cost come
girerebbe sul display centrale. Modulo `app` in fondo allo script, chiamato dal
loop con `app.frame(dt)` subito dopo `road.frame()`.

### Lo spazio di disegno: 454 × 382

Tutta la UI è scritta in **px fissi** dentro uno spazio di 454 × 382, e `fit()`
la posa sul vetro con **un solo `scale()`** non uniforme su `#app`, ricalcolato da
un `ResizeObserver` su `#glass`. Così un `font-size: 11px` nel CSS vale sempre la
stessa frazione di schermo, a qualunque dimensione di finestra, e non serve
inseguire `cqw`/`vw` in ogni regola.

Il canvas della mappa disegna anch'esso in px di *design*: solo il suo backing
store segue i pixel veri (`pxRatio = scala × devicePixelRatio`).

> **Trappola già pagata due volte.**
> 1. Scrivere `cv.width` **cancella** il canvas, e `ResizeObserver` spara un
>    callback da solo appena inizia a osservare: la mappa si disegnava e veniva
>    subito ripulita. `sizeCanvas()` ora tocca il backing store **solo se è
>    davvero cambiato**, e ridisegna.
> 2. `.ap-pane` imposta `display:flex`, che **batte** `[hidden]` dello user-agent:
>    senza la regola `.ap-pane[hidden]{display:none}` le due viste si disegnavano
>    una sopra l'altra.

### Vista 1 — Cost history (quella aperta all'avvio)

| Blocco | Cosa mostra | Da dove |
|---|---|---|
| Testata | costo del viaggio in euro, km, litri | `state.tripCost / tripKm / tripLiters` |
| Quattro celle | €/km ora, €/km medi, L/100 km, €/L nel serbatoio | `state` — gli stessi numeri del cluster |
| Grafico a barre | **una barra ogni 200 m** di strada vera, 25 barre = ultimi 5 km | campionato in `sampleTrip()` dai delta di `tripKm`/`tripCost` |
| Viaggi precedenti | 4 righe fittizie, con freccia verde/rossa rispetto alla media | costanti nel modulo, **date calcolate da oggi** |

Le 25 barre sono **precaricate** a valori plausibili: il contachilometri parte già
da 48 km, un grafico vuoto direbbe che l'auto non ha mai camminato. La riga
tratteggiata verde è la media del viaggio, l'ultima barra (bianca) è l'adesso.
Se il viaggio viene azzerato con RESET, `sampleTrip()` vede un `dKm` negativo e
riparte senza sporcare la serie.

### Vista 2 — Local pump map

Lista ordinata per prezzo a sinistra, mini-mappa a destra, e in fondo la riga del
risparmio. Il colore è **una sola scala** verde → ambra → rosso (`priceColor()`)
calcolata sull'intervallo dei prezzi presenti: la più conveniente è verde con
l'alone pulsante, la più cara è rossa. La barra colorata a sinistra di ogni riga e
il pin sulla mappa usano lo stesso valore, quindi lista e mappa non possono
contraddirsi.

**Le stazioni della mappa sono un mondo a sé**, diverso da quelle che si incontrano
nel parabrezza (`road`, ogni 2 km). Sette stazioni entro ±1500 m di lato, da 1800 m
dietro a 2400 m davanti: scorrono verso l'auto con i **metri veri percorsi** e,
quando restano indietro, rinascono davanti con nome e prezzo nuovi. Il prezzo è
`state.marketPrice + offset`, quindi quando l'API dei prezzi risponde **mappa e
cluster si muovono insieme**.

Il mondo è ritagliato apposta su quello che la mappa inquadra (`MAPR` 2500 m): ogni
riga della lista ha un pin che si può indicare col dito durante una demo.

Il risparmio in fondo è `(prezzo medio − migliore) × litri mancanti al pieno`: un
numero che **cresce col serbatoio che si svuota**, come nella realtà.

### Ritmi

Testo dei numeri 4 volte al secondo, canvas ~16 volte, e il `€/km` in alto a ogni
frame: la mappa si ridisegna **solo se è la vista aperta**, ma i dati (barre,
posizione delle stazioni) si aggiornano sempre, così cambiando scheda non c'è mai
un buco nella storia.

## Cosa è cambiato nella strada

Il canvas copre ora tutti i 1720. Nel modulo `road`:

- `SW` viene **riletto da `--scenew`** e sostituisce il vecchio 1147 in tutti i
  riempimenti a tutta larghezza: cielo, sole, colline, erba, foschia, `clearRect`,
  `clip`, e il culling dei cartelli chilometrici
- le **nuvole** hanno una banda di ricircolo `SW + 460` e sono più numerose
  (`× SW/1147`), altrimenti il cielo nuovo restava vuoto sulla destra
- il **guard-rail** viene disegnato più vicino di `Z_NEAR` (`Z_RAIL`) finché non
  esce dal bordo destro: prima finiva a mezz'aria a x 1472, dove la vecchia
  inquadratura aveva la plancia — [09-strada.md](09-strada.md)
- il **dirigibile** ha wrap e cull legati a `SW`: spariva a tre quarti del cielo
- **`VPX` resta 575**: il punto di fuga è quello della foto, non dell'inquadratura.
  La carreggiata è quindi decentrata a sinistra — è la stessa cosa che si vede da un
  finestrino passeggero, non un errore da correggere

Tutto il resto — prospettiva, metri reali, curve, traffico, stazioni — è invariato:
vedi [09-strada.md](09-strada.md).

## Cosa NON è cambiato

Foto, maschera del parabrezza, cluster SVG, fisica, prezzi, pedali: **niente**. La
foto e il cluster sono larghi 1147 e ancorati a sinistra, con le stesse coordinate
di prima. I pedali e il prompt del rifornimento sono stati ricentrati **sulla foto**
(x 573.5), non sulla scena: appartengono al guidatore.

## Il prezzo pagato

A parità di finestra il cockpit è più piccolo di prima, perché lo stage è largo il
50 % in più e scala tutto insieme. È il costo dell'inquadratura larga.
