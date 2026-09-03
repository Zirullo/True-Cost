# 09 · La strada nel parabrezza

Il modulo `road` in `index.html` (IIFE dentro lo script principale, subito prima della
sezione *physics*). Disegna su `<canvas id="road">` dentro `#windshield`.
Chiamato una volta per frame: `road.frame(state.currentSpeed, dt)`.

Decisioni e alternative scartate: [01-decisioni.md](01-decisioni.md).

## Il modello prospettico

Il canvas lavora nelle **stesse coordinate della foto** (1 unità = 1 px), come l'SVG
del cluster — ma **è largo quanto la scena, non quanto la foto**: `SW` viene riletto
da `--scenew` (oggi 1720) e vale per tutti i riempimenti a tutta larghezza. Il punto
di fuga resta quello della foto: [11-plancia-estesa.md](11-plancia-estesa.md). Un punto del mondo — `X` metri a lato, `Y` metri di altezza, `z` metri avanti —
finisce a:

```
sx = VPX + (X + curve(z) - sway) · F / z
sy = HZ  + (CAMH - Y)            · F / z
```

| Costante | Valore | Cos'è |
|---|---|---|
| `VPX`, `HZ` | 575, **38** | punto di fuga. La x è quella della foto; la y è stata **abbassata** da 60 per far entrare l'asfalto vicino (vedi *La velocità percepita*) |
| `F` | **900 px** | focale — ~65° di campo orizzontale, cioè l'obiettivo con cui si scattano davvero gli interni |
| `CAMH` | 1.2 m | altezza occhi del guidatore |
| `K = F·CAMH` | **1080** | il suolo sta a `y = HZ + K/z`: è l'inversa che dà la profondità di ogni riga |
| `YMAX` | 176 | ultima riga disegnata: sotto c'è la plancia, non si vede |
| `LANE` | 3.6 m | 3 corsie, camera in quella centrale |
| `DASH` | 12 m (4.5 dipinti) | strisce di corsia all'italiana |

Il vetro è una striscia bassa: si disegnano ~138 righe, da y 39 a y 176, ma la
maschera del parabrezza ne mostra solo fino a **y ≈ 122 al centro** — cioè
**12.9 m davanti al cofano**. Quella è la riga che conta: è la cosa più veloce
sullo schermo.

## La velocità percepita

> Questo è il punto in cui la strada era sbagliata fino al 2026-09-02: sembrava
> andare **molto** più piano del tachimetro.

L'occhio non legge la velocità dai numeri, la legge dal **suolo più vicino che
vede**. Il suo scorrimento su una riga dello schermo vale

```
dy/dt = v · Δy² / (F · CAMH)          Δy = riga − HZ
```

quindi la sensazione dipende **solo** da `F·CAMH` e da quanto sotto l'orizzonte la
plancia taglia la vista. Non da `travel`, che è sempre stato in metri veri.

| | prima | ora | auto vera |
|---|---|---|---|
| `F·CAMH` | 1800 | 1080 | — |
| taglio della plancia al centro | 29 m | **12.9 m** | 8–10 m |
| velocità angolare a 128 km/h | 2.9 °/s | **14.8 °/s** | 24–38 °/s |

Era ~8× troppo lento. La correzione è tutta nella **camera** — focale più corta e
orizzonte più basso sullo schermo — e **non tocca la fisica**: le strisce restano a
12 m, i pali a 4 m e `travel` avanza sempre in metri veri. Il resto della differenza
con un'auto vera è nel ritaglio della foto, che non ha finestrini laterali: la
periferia, dove il guidatore prende metà della sua sensazione di velocità, qui non
c'è.

**Se un giorno sembrasse di nuovo lenta**, prima di toccare `F` verifica i frame: il
loop limita `dt` a 50 ms, quindi sotto i 20 fps la strada rallenta davvero mentre il
tachimetro no.

## Perché la velocità è credibile

`travel += (km/h) / 3.6 × dt` — avanzamento in **metri veri**. Non c'è nessun fattore
di "sensazione" da tarare: strisce ogni 12 m e pali ogni 4 m passano al ritmo giusto
perché la geometria è giusta. A 0 km/h `travel` non avanza e la scena si ferma.

Il `dt` arriva dal loop (limitato a 50 ms, così un ritorno da scheda in background non
teletrasporta la strada di 200 m).

> **Nota**: il *Time Warp* dei comandi **non** accelera la strada. Comprime il
> viaggio (km, litri, euro), non il tempo fisico: la vettura resta a quella velocità.

## Le curve

La mezzeria segue `curveC(s) = 25·sin(s/420) + 6·sin(s/173)` metri. Ma non basta
usarla così: quello che si vede è la strada **rispetto a dove punta il muso**, quindi
si sottrae posizione e direzione attuali:

```
curveAt(z) = curveC(travel + z) − curveC(travel) − z · curveC'(travel)
```

Il termine lineare è l'assetto: tolto quello, resta una deviazione che cresce come
`z²` e sullo schermo diventa una piega che si allarga con la distanza — cioè una curva.
Raggi ~5–8 km: sono curve da autostrada, si notano appena, ed è quello che serve.

`sway` aggiunge un ondeggio laterale di ±13 cm: senza, la scena è innaturalmente
rigida.

## Cosa c'è nella scena, in ordine di disegno

Tutto quello che segue è **ridisegnato ogni frame** dentro un clip a `y < YMAX`.

1. **Cielo** — gradiente a tre stop (`skyTop` → `skyMid` → `skyLow`), un alone
   caldo di sole basso a destra e nuvole (**7 × SW/1147**, oggi ~10) su parallasse lentissima
   (`travel × 0.045`), posizionate da `hash()` così restano sempre le stesse
1bis. **UFO** (`ufo()`) — l'easter egg: compare in fondo a sinistra ogni 5 minuti,
   scende, apre un raggio, **tira su una mucca** e risale. Non scorre e non ha
   parallasse: vedi *L'UFO* sotto
2. **Tre strati di fondale** (`layer()`): collina lontana, collina vicina, boscaglia.
   Ognuno ha la **sua** parallasse (0.00035 / 0.0009 / 0.0022): è quella differenza,
   non il colore, a dare la distanza
3. **Suolo, riga per riga** — vedi sotto
3bis. **Alberi** su entrambi i lati (`trees()`), disegnati **prima** delle barriere
   così il new jersey e il guard-rail ne coprono la base — vedi sotto
3ter. **Cartelloni** ogni km a sinistra (`kmSigns()`), dal km 1 — vedi sotto
4. **New jersey** a sinistra: faccia, fascia alta, cappello, sporco alla base,
   **giunti ogni 6 m**, **pannelli antiabbagliamento ogni 2.5 m** e catarifrangenti
   ogni 12 m
5. **Guard-rail** a destra: pali ogni 4 m, nastro a doppia flangia con scanalatura,
   delineatori ambra ogni 25 m. È l'unica cosa disegnata **più vicina di `Z_NEAR`**
   (`Z_RAIL`, ~4.95 m): vedi sotto
6. **Cartelli autostradali** ogni 500 m sulla destra: entrano piccoli, si gonfiano ed
   escono sopra il tetto in circa un secondo. Niente legge "veloce" quanto loro.
   Lo slot che cade su una stazione viene saltato, altrimenti il cartello verde
   finisce esattamente sopra il cartellone del prezzo
6bis. **Stazione di servizio** ogni 2 km sulla destra (`stations()`) — vedi sotto
6ter. **Cavalcavia** ogni km (`overpasses()`), disegnato **dopo** le stazioni e
   **prima** del traffico: l'impalcato sta in alto dove i veicoli lontani non
   arrivano mai, e così un'auto vicina passa comunque davanti a un ponte lontano
7. **Traffico** — fino a 9 veicoli, disegnati dal più lontano
8. **Foschia** — gradiente verticale sopra tutto

### Il suolo, una riga alla volta

Ogni riga dello schermo è **sempre** la stessa profondità, quindi la sua nebbia, la
sua scala e **tutti i suoi colori sono costanti**: sono precalcolati una volta
nell'array `rows`, e il ciclo per frame si limita a piazzare rettangoli.

Ogni materiale è dichiarato **due volte** in `P`, vicino e lontano (`asphN`/`asphF`,
`grassN`/`grassF`, …), e ogni riga è la miscela delle due secondo
`f = 1 − e^(−z/190)`. Il contrasto cala con la distanza come nella realtà, invece di
essere una tinta piatta corretta solo dalla foschia finale.

Sopra l'asfalto, in ordine: bande di sfalcio sul prato, **rotaie di rotolamento**
(le uniche marcature che **non** scorrono), **grana** — 3 chiazze per riga scelte da
`hash()` sul **metro di strada**, così la texture viaggia col mondo invece di
ribollire sul posto — un **giunto di catrame** trasversale ogni 20 m, e infine le
strisce, la cui opacità sfuma vicino all'orizzonte (`row.fade`) perché lì una linea
netta striscerebbe.

### Il beccheggio

`bob` sposta verticalmente tutta la scena di una frazione di pixel, in proporzione
alla velocità. Non si "vede": si legge come sforzo.

## Il traffico

Ogni veicolo ha una velocità **assoluta**; la distanza evolve con
`z += (sua − nostra) · dt`. Quindi il sorpasso è una conseguenza, non un'animazione.

- corsia sinistra: auto a 34–42 m/s (~125–150 km/h)
- corsia destra: auto a 27–32, camion a 22–25 m/s
- i veicoli nascono a una distanza qualsiasi fra 90 e 420 m, così la scena è già
  popolata al primo secondo invece di riempirsi in un minuto
- se andiamo **sotto** i ~120 km/h, alcuni compaiono da dietro (`z` piccolo) e ci
  superano; sopra, li raggiungiamo noi da lontano

A vettura ferma il traffico si allontana e sparisce: corretto, non è un bug.

## Le stazioni di servizio

Una ogni `ST_EVERY` metri sulla destra, oltre il guard-rail. È l'unico elemento
della scena che il **cluster legge**: `road.station()` restituisce
`{ idx, dist, price }` della prossima, e la riga *Local Price* mostra prezzo e
distanza che scala (metri sotto il chilometro).

| Costante | Valore | Cos'è |
|---|---|---|
| `ST_EVERY` | **2000 m** | passo fra le stazioni |
| `ST_SPREAD` | 0.15 € | scarto massimo attorno a `state.marketPrice` |
| `ST_TOTEM` | `RAIL + 3.6` | posizione laterale del cartellone |
| `ST_X` | `RAIL + 6` | bordo vicino del piazzale |

**Il prezzo è funzione dell'indice della stazione**, non del tempo:
`marketPrice + (hash(i·97+13)·2−1)·0.15`. Deciso una volta, non tremola mentre ti
avvicini, e cambia a ogni stazione. `marketPrice` è la media live MIMIT (1.85 di
fallback) — [05-dati-esterni.md](05-dati-esterni.md).

### Perché il cartellone è basso

Prima versione: totem da 9 m, come quelli veri. Sbagliato — il parabrezza mostra
una **striscia** che parte dall'orizzonte, quindi un oggetto alto `Y` entra nel
fotogramma solo se `z > 23.7·(Y − 1.2)`: il totem usciva dal bordo alto proprio
mentre si avvicinava abbastanza da poterlo leggere.

Il cartellone attuale sta fra **1.4 e 4.4 m**: resta dentro il vetro da ~200 m
fino a ~40 m, cioè per tutto l'avvicinamento, ed è leggibile (`h > 12 px`) da
~180 m. Stesso motivo per la pensilina, abbassata da 5 a 4.4 m.

> Regola generale, se aggiungi altri oggetti a bordo strada: **più sono alti,
> prima escono dall'inquadratura**. `y = HZ + (CAMH − Y)·F/z`.

### Cosa viene disegnato

Piazzale in cemento (22 m di strada), negozio arretrato, pensilina bianca su
quattro pilastri con fascia rossa sul bordo vicino, due pompe, e il cartellone:
pannello blu, banda verde in alto, prezzo bianco e `EURO / L` sotto (quest'ultimo
solo se il pannello supera i 22 px). Tutto con `quad()`, che proietta quattro
angoli `[X, Y, z]` — l'unico helper nuovo, riusabile per qualsiasi altro edificio.

## I cavalcavia

Uno ogni `OVP_EVERY` metri, il primo a `OVP_FIRST`. È **l'unica cosa della scena che
passa sopra la telecamera**: scende nell'inquadratura, si gonfia ed esce dal bordo
alto nell'ultimo secondo. Niente altro dice così chiaramente che la strada ha una
terza dimensione.

| Costante | Valore | Cos'è |
|---|---|---|
| `OVP_EVERY` | **1000 m** | passo fra i cavalcavia |
| `OVP_FIRST` | **500 m** | il primo, mezzo chilometro dopo la partenza |
| `OVP_CLEAR` | 5.4 m | altezza dell'intradosso: franco autostradale |
| `OVP_DECK` / `OVP_PARA` | 1.15 / 1.05 m | trave dell'impalcato · parapetto |
| `OVP_L`, `OVP_R` | −25.5, +13.5 m | dove finisce la **campata piana** |
| `OVP_PL`, `OVP_PR` | −21.5, +10.2 m | le pile, fuori da entrambe le carreggiate |
| `OVP_W` | 9 m | quanto è largo il ponte, in `z` |
| `OVP_RUN` | 48 m | la rampa, dalla fine della campata al suolo |
| `OVP_GAP` | 24 m | banchina lasciata libera dagli alberi |

È **alto**, e le cose alte escono presto: `y = HZ + (CAMH − Y)·F/z` porta
l'intradosso fuori dal bordo dello stage a `z ≈ 21 m`. È voluto — deve sparire
sopra il tetto, non rimpicciolirsi in lontananza.

Tutto è disegnato in **un solo piano a `z` costante** (rampe, pile, trave,
parapetto), più l'intradosso — un `quad` da `z` a `z + OVP_W` che diventa un
soffitto solo sotto i 70 m — e una banda d'ombra sull'asfalto.

### Le rampe

Un ponte che finisce con la propria campata sembra un portale. Quello che dice
*strada che scavalca la strada* è il **rilevato** ai due lati che scende nei campi.
`ramp()` è un cuneo in quattro `quad` lungo un unico bordo inclinato: scarpata
erbosa, trave di bordo, parapetto, corrimano.

La scarpata è il punto. Una faccia in cemento a tutta altezza fino a 60 m murava
tutto l'orizzonte; quello che si costruisce davvero è un **rilevato in terra** con
la trave appoggiata sopra, e il verde si fonde con la banchina invece di tagliarla.

48 m di rampa per 6.5 m di quota è molto più ripido del vero (servirebbero ~110 m,
fuori inquadratura a qualsiasi distanza utile): a 130 km/h quello che si registra
è la **sagoma**, non la pendenza.

### Chi deve togliersi di mezzo

Tre cose stavano dove passa il ponte e ora saltano lo slot:

- gli **alberi**, entro `OVP_GAP` da un cavalcavia — crescerebbero dentro
  l'impalcato, e il rilevato li coprirebbe anche quando sono più vicini di lui
- i **cartelli verdi**, che cadono ogni 500 m e quindi **esattamente** su ogni
  cavalcavia: il cartello finiva dentro la trave
- i **cartelloni**, entro 12 m: sono alti 5.6 m contro un intradosso a 5.4

## I cartelloni

Uno ogni `KM_EVERY` = **1000 m** a sinistra, oltre la carreggiata opposta (la destra
è già occupata da cartelli verdi e stazioni, si darebbero fastidio). `travel` parte
da 0, quindi il primo sta **al km 1** e gli altri cadono sui km tondi. Il pannello sta
fra 2.4 e 5.6 m — resta dentro il vetro da ~340 m fino a ~21 m, quindi spazza in
alto e sopra il tetto invece di sparire ancora lontano.

I **cavalcavia stanno sul mezzo chilometro** (`OVP_FIRST` 500, `OVP_EVERY` 1000):
è per questo che un cartellone non ne trova mai uno addosso, nonostante il passo
sia lo stesso. Se cambi uno dei due, controlla lo scarto.

Si **alternano due grafiche**, quindi ognuna torna ogni **2 km** (~56 s a 130 km/h):

1. **il marchio Stellantis** — bianco sul navy corporate, con la banda ciano al piede
2. **il manifesto True Cost** — stampato al vivo su tutto il pannello: STELLANTIS in
   testa, il tondo TC verde acqua, `TRUE COST`, `TRANSPARENCY THAT COUNT$`, le tre
   righe di corpo minore e la fascia `POWERED BY STELLANTIS` in fondo

Le due grafiche sono dipinte **una volta sola** in canvas fuori schermo (`LOGO` e
`BOARD_TC`) e poi timbrate con `drawImage`. Ridisegnare sessanta pallini e dieci
righe di testo per cartellone per frame costerebbe di più e — peggio — **tremolerebbe**
mentre il cartellone scala; una bitmap rimpicciolita sta zitta.

`BOARD_TC` è impaginato sul **2.625:1 del cartellone**, non sul formato del bozzetto:
così non si stira nulla quando viene timbrato. Il corpo minore è illeggibile oltre i
~60 m ed è lì apposta — a velocità è **texture**, ed è la texture che dice all'occhio
che sta guardando un manifesto stampato e non un rettangolo colorato.

### L'anello di pallini

Il logo Stellantis è il lettering più l'anello di pallini sopra la **A** (`dotRing()`,
centrato sul `aCx` che `wordmark()` restituisce). Non è un cerchio punteggiato: i
pallini cambiano raggio lungo la circonferenza — più grossi in basso a sinistra — e
stanno su una **banda** larga, non su un raggio esatto. È questo che lo fa leggere
come una costellazione invece che come uno stencil.

Due tarature scoperte a video, non a tavolino:

- **il pavimento del raggio** (`0.024·R`): portando il lato chiaro dell'anello fino a
  zero, alla distanza di un cartellone il cerchio smette di chiudersi e quel che
  resta è una sbavatura
- **il peso 400**: il lettering vero è più sottile, ma a 15 px di altezza un 300
  sparisce

L'anello va alzato di `0.10·FS`: le lettere sono disegnate con `textBaseline
'middle'`, e il centro delle maiuscole sta **più in alto** di quella linea.

## Gli alberi

Due file, una per lato, disegnate **prima** di new jersey e guard-rail (che devono
coprirne la base) e **dal fondo verso di noi**, così i vicini si sovrappongono ai
lontani.

| Costante | Valore | Cos'è |
|---|---|---|
| `TREE_STEP` | 11 m | passo degli slot |
| soglia slot vuoto | 0.42 | il 42% degli slot resta vuoto |
| jitter | ±3.5 m | spostamento dello slot lungo la strada |
| `TREE_FAR` | 300 m | oltre, ci pensa la boscaglia del fondale |
| `TREE_GAP` | 26 m | spazio lasciato libero attorno a una stazione |
| offset laterali | sinistra 19.5–31.5 m · destra 9.5–22.5 m | |

**L'offset laterale è il punto.** Un albero a 9.5 m esce dall'inquadratura a 15 m
davanti al cofano, quindi attraversa tutta la larghezza del vetro nell'ultimo mezzo
secondo: *quello* è il senso di velocità. Gli alberi lontani riempiono e basta.
Allontanare la fila dalla strada per "vederli meglio" li rende inutili.

**Nessuno uguale a un altro**: dallo slot `n`, via `hash()`, escono specie, altezza,
larghezza, quanti bozzi ha la chioma, il verde e la distanza dalla strada. Stesso
metro di strada, stesso albero — non tremolano mai, ma non se ne ripete uno.

Quattro specie, con le soglie in `drawTree()`:

| Specie | Quota | Forma |
|---|---|---|
| Cipresso | 24% | fiamma stretta, 7–12 m: il primo a uscire dal bordo alto |
| Pino marittimo | 22% | fusto nudo + chioma piatta e larga |
| Latifoglia | 28% | 2–3 ellissi sovrapposte, mai le stesse due |
| Cespuglio | 26% | basso e largo, riempie i buchi |

Ogni albero ha la sua **ombra** a terra (senza, galleggia) e il suo verde è mescolato
verso `P.hillFar` secondo `f`, come tutto il resto della scena.

### Una trappola trovata qui

`fit()` riassegna `cv.width`, e **assegnare `width` azzera il canvas**. Siccome è
chiamata anche dal `ResizeObserver`, che scatta a ogni assestamento del layout, la
strada spariva per un frame a momenti casuali (visibilissimo negli screenshot
headless, un lampo dal vivo). Ora `fit()` ridimensiona **solo se la dimensione in
pixel è davvero cambiata**.

## Il dirigibile

`blimp()`, l'unica cosa a spigoli vivi nel cielo: le nuvole sono gradienti che si
leggono come atmosfera, e ci vuole **un oggetto solido** che passi davanti a loro
perché il cielo abbia una profondità. Attraversa da sinistra a destra a `BL_SPD`
7.5 px/s **suoi**, quindi si muove anche a vettura ferma, e la corsa lo tira
indietro di `BL_PAR` 0.016 — un terzo di quello che prendono le nuvole, perché è
più lontano di loro. È quella **differenza** a essere la parallasse.

Sulla fiancata porta la scritta **STELLANTIS**, disegnata con lo stesso `wordmark()`
dei cartelloni e nello stesso navy `#2b3a8f`, a corpo 7 px e con alfa 0.62: a un
chilometro un logo non si legge, si **riconosce**, e la foschia deve stare anche su
di lui o diventa un adesivo.

`BL_PHASE` lo mette già in quadro al caricamento: senza, il primo passaggio sono
venticinque secondi di cielo vuoto, che è esattamente il modo in cui si finisce per
crederlo rotto.

## L'UFO (easter egg)

`ufo()` in `index.html`, disegnato in `backdrop()` **subito dopo il dirigibile**:
così i tre strati di fondale gli passano davanti e il raggio finisce dietro la
collina invece di appoggiarsi sul nulla.

| Costante | Valore | Cos'è |
|---|---|---|
| `UFO_X` | 152 | fondo a sinistra: l'unica fascia di cielo che nessuno strumento o cartellone attraversa mai |
| `yLow` | −6 | quota di stazionamento, **44 px sopra l'orizzonte**: basso sulle colline |
| `UFO_R` | 15 px | semilarghezza del disco — **30 px su 1147**, di proposito piccolo |
| `UFO_DUR` | 10.5 s | durata di un passaggio, ingresso e uscita compresi |
| `UFO_EVERY` | 300 s | da un arrivo al successivo: **5 minuti**, primo compreso |
| `COW_L` | 1.05 | scala della mucca — tutto il resto del disegno è in lunghezze-mucca |

**La discesa è tutto lo scherzo.** Tre battute — scende, sta, risale — con uno
`smoothstep` sulle due in movimento; `glide` (0→1) le guida tutte, e il raggio
è semplicemente `max(0, (glide − 0.74) / 0.26)`, quindi nasce e muore con la
sosta senza un timer suo. Parte da `y = −RISE − 34`, cioè **sopra** il bordo
alto del palco, e arriva a `y = −26`: entra ed esce scivolando, senza pop.

**La mucca è il motivo per cui vola basso.** Sale nel raggio fra `p` 0.34 e 0.66,
da `y` locale 68 a 11. Sotto `y = 39` la scena viene ridipinta dal suolo e dai
fondali, che si disegnano **dopo** il backdrop: il suo primo secondo lo passa dietro
la boscaglia, quindi sembra **sollevata da terra vera** invece che comparire in aria.
In alto non tocca lo scafo — svanisce sotto di esso (una mucca che tocca il disco è
un tamponamento, una che sfuma è un'abduzione). È una **silhouette piena**: a nove
pixel di altezza non sopravvive nessuna ombreggiatura, conta solo il profilo — corpo,
muso, corno, mammella e quattro zampe che penzolano sfasate.

A questa quota il disco passa **dietro gli alberi e i cavalcavia** di sinistra, che
sono disegnati dopo: è voluto, e a 128 km/h l'occlusione dura mezzo secondo.

**Non scorre con `travel` e non ha parallasse**, a differenza di tutto il resto
del cielo. È voluto: le nuvole e il dirigibile sono scenografia trascinata via
dalla corsa, questo è fermo sulla verticale della strada. Quella differenza è
esattamente ciò che separa "lontano" da "non di qui".

**`U` lo chiama subito** (il listener è dentro il modulo, ignora i campi di testo):
un easter egg che non si può mostrare a comando è inutile in demo.

## `Z_NEAR` e il guard-rail

`Z_NEAR = K / (YMAX − HZ)` è la profondità della riga `YMAX`, l'ultima che il canvas
dipinge. Per tutto ciò che sta **sul piano stradale** è il punto giusto dove fermarsi:
più vicino di così lo copre la plancia.

Il guard-rail no. Sta **7.8 m di lato**, quindi a `Z_NEAR` è già a x 1472 — e finché
la scena era larga 1147 quel troncone stava al sicuro dietro la plancia. Nella scena
da 1720 è in mezzo al vetro, e il nastro **finiva a mezz'aria**.

Per questo `ribbon()`, `posts()` e `studs()` accettano un `zNear` opzionale, e il
guard-rail passa il suo:

```
Z_RAIL = min(Z_NEAR, 0.85 · RAIL · F / (SW + 60 − VPX))     ≈ 4.95 m
```

cioè: continua a disegnarlo finché non è **uscito dal bordo destro** della scena
(a x 1720 ci arriva a z ≈ 6.1 m, e lì è già quasi tutto dietro il cruscotto).
Lo 0.85 è il margine per le curve, che spostano il nastro di qualche metro di lato.

Alberi (z ≥ 6) e stazioni (z ≥ 4) erano già disegnati più vicini di `Z_NEAR`, quindi
non avevano il problema.

## Se qualcosa stona

| Sintomo | Dove mettere le mani |
|---|---|
| La strada non si incastra col cofano | `YMAX`, e i punti dell'array `edge` (maschera, [03-mappa-design.md](03-mappa-design.md)) |
| Sembra troppo grandangolare / troppo tele | `F` (e ricorda che `K = F·CAMH` cambia con lei, quindi cambia anche la velocità percepita) |
| Sembra più lenta del tachimetro | `F·CAMH` e `HZ` — vedi *La velocità percepita* |
| La grana dell'asfalto fa chiazze | la sua larghezza è in **metri** (`0.22 * s`): con una focale più corta un valore in pixel diventa enorme |
| Si vede troppo poca strada vicina | abbassa `CAMH` o alza `YMAX` |
| Le curve sono troppo marcate | le ampiezze 25 e 6 in `curveC` / `curveD` — vanno cambiate **insieme** |
| Troppo o troppo poco traffico | la soglia `traffic.length < 7` e la probabilità `0.03` in `moveTraffic` |
| Gli alberi sembrano una siepe | sono troppo fitti o troppo lontani: `TREE_STEP` (11 m), la soglia `0.42` degli slot vuoti, `TREE_FAR` (300 m) |
| Gli alberi non danno velocità | l'offset laterale della riga vicina in `trees()` (9.5 m a destra): più sono lontani dalla strada, più lento sembra tutto |
| Il cavalcavia sparisce troppo presto | è alto: `OVP_CLEAR`, e `y = HZ + (CAMH − Y)·F/z` |
| Le rampe murano l'orizzonte | la scarpata erbosa in `ramp()`, non la campata: `OVP_RUN` e lo spessore `B` della trave |
| Un albero spunta dentro l'impalcato | `OVP_GAP` |
| Il logo sul cartellone è una sbavatura | il pavimento del raggio in `dotRing()` e il peso del font in `wordmark()` — **non** la larghezza `w * 0.86` |
| Il manifesto True Cost è stirato | è impaginato 2.625:1: se cambi `KM_LO`/`KM_HI`/`KM_HALF` cambia anche `BOARD_TC` |
| Sempre lo stesso albero | le soglie di specie in `drawTree()` (0.24 / 0.46 / 0.74) |
| Troppo lattiginoso | gli stop del gradiente `hz` in fondo a `frame()` — e ricorda che la nebbia è contata **due volte**: lì e nel mix vicino/lontano di `P` |
| Asfalto slavato in lontananza | i colori `*F` in `P`, non la foschia |
| I pannelli antiabbagliamento sfarfallano | passo e sfumatura in `blades()` (oggi 2.5 m, spenti oltre 150 m) |
| Un veicolo lontano sembra un ritaglio | il velo di foschia in fondo a `drawVehicle()` |
| La strada resta vuota all'avvio | i veicoli nascono fra 90 e 420 m (`spawn()`); prima nascevano solo al limite lontano |
| Le stazioni si vedono troppo di rado | `ST_EVERY` (2000 m ≈ 55 s a 130 km/h; il valore autostradale vero sarebbe 5000) |
| Il prezzo sul cartellone non si legge | l'altezza del pannello `BRD_LO`/`BRD_HI` e la soglia `h > 12` — **non** la dimensione del font |
| Il cartellone esce dal bordo alto | è alto: vedi *Perché il cartellone è basso* |
| Un cartellone finisce dentro un cavalcavia | il passo è lo stesso (1 km): serve che `OVP_FIRST` resti sul mezzo chilometro |
| Un elemento laterale finisce a mezz'aria a destra | è fermo a `Z_NEAR`: gli serve un `zNear` suo, come `Z_RAIL` per il guard-rail |
| Il dirigibile sparisce prima del bordo | `BL_SPAN` e il cull, ora legati a `SW` |
| La scritta sul dirigibile sbava | il corpo 7 px e il peso 600 in `blimp()` — non allargarla, si allarga il dirigibile |
| L'UFO non si vede mai / si vede troppo | `UFO_EVERY` (cadenza) e `UFO_DUR` (durata); premi **U** per chiamarlo subito |
| Gli alberi coprono sempre l'UFO | vola basso di proposito: alza `yLow` (verso −30) o spostalo a sinistra con `UFO_X` |
| La mucca esce già in aria | sale troppo in fretta o parte troppo in alto: il 68 di `cy` in `ufo()` deve restare **sotto** y 39 |
