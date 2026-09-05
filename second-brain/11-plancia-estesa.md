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

### Le viste, e perché sono bottoni

Fino al 2026-09-04 le viste erano due e si sceglievano con delle **linguette
unite**, attaccate fra loro e al pannello sotto. Era un oggetto solo: si leggeva
come una barra di titolo, non come due comandi, e in demo nessuno pensava di
toccarle. Ora sono **bottoni staccati** — gap fra loro, angoli tondi su tutti
e quattro i lati, una luce sul bordo alto e un'ombra sotto — e **si abbassano di
2 px quando si premono**. Quello aperto resta abbassato e acceso di blu.

Dal 2026-09-04 la demo **si apre su PUMP MAP, sul chip «lungo questa strada»**,
non più su Cost history: è l'unica vista che si può verificare guardando fuori
dal parabrezza, e chi apre la pagina vede subito che la lista e i cartelloni
dicono lo stesso prezzo. Le altre restano a un tocco.

Dal 2026-09-04 sono **quattro**: si è aggiunto REPORT. A quattro, «Price history»
non ci stava più nel suo bottone: l'etichetta scende a **8.5 px** e perde quasi
tutto l'`letter-spacing` (da 1.15 a 0.7 px), l'icona da 14 a 13, il gap fra i
bottoni da 9 a 7. Nessuna etichetta è stata accorciata: un bottone che dice
«Prices» non dice cosa fa.

Dal 2026-09-05 sono **cinque**: si è aggiunto TRIPS, e a cinque quella frase non
si può più mantenere. 454 px meno 22 di padding meno quattro gap da 7 fanno
**80.8 px a bottone**, icona compresa: nessuna etichetta di due parole ci entra a
nessuna dimensione leggibile. Quindi si accorciano **tutte e cinque** — *Costs,
Pumps, Prices, Report, Trips* — e l'icona resta a portare il resto. È il prezzo
della quinta vista, ed è stato pagato con gli occhi aperti: meglio cinque nomi
corti e un'icona ciascuno che quattro nomi lunghi e una vista in meno.

> **Trappola**: `#app button{font:inherit}` ha specificità `(1,0,1)` e **batte**
> qualunque regola di classe. Ogni regola che tocca un bottone dentro l'app deve
> portarsi dietro `#app` (`#app .ap-tab`, `#app .sc`, …), altrimenti il
> `font-size` torna a quello ereditato e le etichette vanno a capo. `#ap-back` ci
> era già cascato prima.

La fila dei bottoni è alta **46 px** invece di 34: il corpo delle viste scende da
318 a 306 righe, e ci sta lo stesso tutto quello che c'era.

### Vista 1 — Cost history (quella aperta all'avvio)

| Blocco | Cosa mostra | Da dove |
|---|---|---|
| Testata | costo del viaggio in euro, km, litri | `state.tripCost / tripKm / tripLiters` |
| Quattro celle | €/km ora, €/km medi, L/100 km, €/L nel serbatoio | `state` — gli stessi numeri del cluster |
| Grafico a barre | **una barra ogni 200 m** di strada vera, 25 barre = ultimi 5 km | campionato in `sampleTrip()` dai delta di `tripKm`/`tripCost` |
| Viaggi precedenti | **18 righe** fittizie **cliccabili** e scorrevoli, con freccia verde/rossa rispetto alla media | `TRIPS` nel modulo, **date calcolate da oggi** |

Le 25 barre sono **precaricate** a valori plausibili: il contachilometri parte già
da 48 km, un grafico vuoto direbbe che l'auto non ha mai camminato. La riga
tratteggiata verde è la media del viaggio, l'ultima barra (bianca) è l'adesso.
Se il viaggio viene azzerato con RESET, `sampleTrip()` vede un `dKm` negativo e
riparte senza sporcare la serie.

#### Aprire un viaggio passato

Cliccando una riga (`openTrip()`) **tutta la metà alta cambia soggetto**: totale,
sottotitolo, le quattro celle — che cambiando significato cambiano anche
**etichetta** — il grafico, il titolo e gli estremi dell'asse. La riga si accende
di blu, compare il pillolo `◀ LIVE` accanto al titolo, e un secondo clic sulla
stessa riga o sul pillolo riporta al viaggio in corso.

Sotto non succede niente: `state` non viene toccato, il campionamento continua,
e la media del viaggio in corso al ritorno è quella giusta. Per questo il ridisegno
è **a evento e non a 4 Hz** (`paintHistory()` smista, e il tick veloce salta la
vista quando `selTrip` è aperto): un viaggio finito è una fotografia.

Ogni viaggio porta anche `min` (durata) e `paid` (€/L pagati al pieno): senza quei
due numeri una scheda di dettaglio non direbbe niente di più della riga. Da lì
escono litri, L/100 km e velocità media. Le **25 barre di ogni viaggio** sono
generate una volta al boot con una forma legata al carattere del percorso —
`road` piatta e bassa, `city` alta e a denti — e poi **normalizzate perché la loro
media sia esattamente gli €/km del viaggio**: così la riga tratteggiata cade dove
la riga della lista dice che deve cadere.

### Vista 2 — Pump map

Lista ordinata per prezzo a sinistra, mini-mappa a destra, e in fondo la riga del
risparmio. Il colore è **una sola scala** verde → ambra → rosso (`priceColor()`)
calcolata sull'intervallo dei prezzi presenti: la più conveniente è verde con
l'alone pulsante, la più cara è rossa. La barra colorata a sinistra di ogni riga e
il pin sulla mappa usano lo stesso valore, quindi lista e mappa non possono
contraddirsi. **L'alone e il pin selezionato si disegnano per ultimi**: due pompe a
pochi metri l'una dall'altra seppellivano il pin di cui parla tutta la vista.

#### Due specie di stazione, e una sola è inventata

Fino al 2026-09-04 le stazioni della mappa erano **un mondo a sé**, diverso da
quelle che si incontrano nel parabrezza. Era la bugia più visibile della demo:
si passava accanto a un cartellone da 2.05 e la riga «lungo questa strada» ne
diceva un'altra. Adesso la mappa ne tiene **due specie**, e la differenza è il
senso stesso del pannello.

| | **lungo la strada** | **in città** |
|---|---|---|
| chi sono | i cartelloni veri di `road`, uno ogni 2 km | pompe inventate, sparse intorno |
| prezzo | `road.priceAt(idx)` — **lo stesso numero scritto sul cartello** | `PRICES.ago(0) + offset` |
| dove stanno | nel corridoio (a 70 m dall’asse, alternate ai due lati) | **sempre fuori** dal corridoio |
| come si muovono | non si muovono: si **richiede** a `road` dove sono | scorrono coi metri veri e rinascono davanti |

Le stazioni della strada **non sono copiate**: l'app tiene 18 caselle, ognuna
porta solo un **indice**, e a ogni ripittura chiede a `road` prezzo, insegna e
distanza di quell'indice. Due copie possono divergere, una domanda no. Poiché i
cartelloni stanno ogni `road.every` metri, la casella *n* porta sempre l'unico
indice congruo a *n*: la stazione scelta **resta la stessa** mentre si guida, e
sparisce dalla lista nel momento esatto in cui la si supera davvero.

Le pompe di città sono **42**, in un mondo largo ±13 000 m e lungo da 7000 m
dietro a 34 000 m davanti, e nascono **sempre oltre il corridoio**: una pompa
inventata così vicina alla carreggiata direbbe di essere uno dei cartelloni, e
non lo è. Le prime 3 nascono vicine (fra −800 e +3200 m), così la demo si apre
su una lista piena anche a 5 km.

Al boot la ricerca trova **7 stazioni a 5 km, 15 a 10 km, 29 a 20 km e 17 lungo
la strada**. Se un chip o `NST` cambiano, questi numeri vanno rimisurati: due
chip che rispondono lo stesso numero sono due chip che non servono a niente.

> **Il prezzo del cartellone non è più un mercato suo.** Valeva 2.10 € ± 25
> centesimi, inventati: contro le pompe di città (che stanno intorno alla media
> vera) la strada sarebbe sembrata sempre carissima. Adesso è
> `PRICES.ago(0) + PRICES.roadOffset(...)`, lo **stesso storico** di cluster,
> pieni e report — solo con una banda circa **tre volte più larga** di quella
> cittadina, altrimenti a 130 km/h i cartelli si somigliano tutti e non c'è
> niente da scegliere. Local Price nel cluster, cartello nel parabrezza e riga
> nella lista sono **un numero solo**.

E perché lo si veda senza doverci credere: sul cartellone c'è ora anche il
**marchio**, scritto sulla fascia verde quando è abbastanza vicino da leggersi.
Le insegne non sono sorteggiate ma **camminate** con passi coprimi con le due
liste (`i·5` sui luoghi, `i·3 + i/14` sui marchi): due stazioni in vista non
possono portare lo stesso nome, mentre con un hash ogni tanto la lista mostrava
due «Eni · Via Torino» a dieci chilometri di distanza.

#### Il raggio di ricerca

Quattro chip: **5 km · 10 km · 20 km · lungo questa strada** (fino al 2026-09-04
erano 1 / 2.5 / 5 km, su un mondo quattro volte più piccolo: a 20 km avrebbero
contenuto *tutte* le stazioni, e il terzo chip sarebbe stato una copia del
secondo). I primi tre sono un raggio (`distOf(s) ≤ scope`); il quarto **non è un
raggio** ma un corridoio — `|x| ≤ 1300 m`, da 1200 m dietro a 34 000 m davanti: le
stazioni che incontreresti senza uscire dal percorso. In quel caso la distanza
nella riga smette di essere una bussola (sarebbe sempre «N») e diventa
**«3.8 km ahead»**.

La ricerca **si vede sulla mappa**, altrimenti i chip cambierebbero la lista senza
un motivo visibile:

| | raggio | lungo la strada |
|---|---|---|
| l'auto sta | al centro (`cy = ch/2`) | in basso (`cy = 0.86·ch`) |
| si vede fin a | `scope × 1.14` | 21 080 m |
| disegnato | cerchio tratteggiato + velo blu | due linee tratteggiate ai lati |

Le stazioni **fuori ricerca restano disegnate**, ma grigie e piccole: la ricerca
decide cosa classificare, non dichiara che intorno non ci sia altro.

Con raggi fino a 20 km **due costanti non potevano più restare fisse**:

- **il passo della griglia** non è più 420 m ma un gradino di una scala tonda
  (420 / 1000 / 2000 / 4000 / 8000 m), scelto perché sullo schermo cadano sempre
  una dozzina di linee: a 420 m una vista da 20 km era carta millimetrata. La
  circonvallazione e la diagonale sono espresse **in isolati** (`3.57·G`, …), così
  tengono la proporzione a ogni zoom;
- **la barra di scala** prende il primo gradino di 250 m → 20 km che sullo schermo
  supera i 34 px, invece delle tre soglie scritte a mano di prima.

Anche il navigatore ha dovuto adeguarsi: una stazione a 18 km non ci si arriva in
città, quindi un percorso oltre **8 km** viene guidato a **25 m/s** invece di 12,5
e può avere **9 manovre** invece di 7. Senza questo, il chip più largo offriva solo
camminate da quaranta minuti.

Il risparmio in fondo è `(prezzo medio − migliore) × litri mancanti al pieno`: un
numero che **cresce col serbatoio che si svuota**, come nella realtà. Ma appena
selezioni una stazione **il piede della vista cambia mestiere**: da didascalia
diventa la porta del navigatore, con il bottone verde `NAVIGATE ▶`.

### Il navigatore — una demo dentro la demo

Si apre **sopra** la mappa (`#ap-nav`, dentro `#pane-map`) ed è **volutamente
scollegato da tutto il resto**: ha la sua strada, la sua velocità (`NAV_V`,
12.5 m/s ≈ 45 km/h) e **non legge né scrive `state`**. Il viaggio sotto continua,
il parabrezza pure, e `✕ END` non lascia niente dietro di sé.

`buildRoute()` prende la distanza in linea d'aria e la **allunga del 22–50 %** —
è quello che fanno le strade — poi la taglia in gambe da almeno 140 m, ognuna che
finisce in una manovra e in una via. Ogni passo è `{t, m, st}`: la manovra alla
**fine** della gamba, la sua lunghezza, e la via in cui si entra dopo.

Sopra: la freccia grande (un `path` in un box 64, uno per manovra), i metri che
scalano, la via, e «then …» con la manovra dopo. Sotto: il **resto
dell'itinerario** — la fila che prima era un buco vuoto alto 200 px — ridisegnata
**solo quando si supera un passo**, non a ogni frame. In fondo quattro celle:
arrivo, km, minuti, €/L là.

Sotto i 70 m dalla manovra la velocità scende a 5.5 m/s: è quello che rende
credibile il conto alla rovescia. Arrivato, il riquadro diventa verde e resta
fermo su `ARRIVED`.

Il navigatore **continua a girare anche cambiando vista**: `navFrame(dt)` sta nel
`frame()` dell'app, non nella mappa. La mappa invece non si ridisegna mentre il
navigatore è aperto (`view === 'map' && !navOn`): sta sotto, coperta.

### Vista 3 — Price history

Due anni di benzina, e dove ci siamo fermati dentro quei due anni.

**La linea** è la media nazionale italiana, benzina self-service, in €/L. Le
ancore sono **mensili** (`ANCH`, 25 valori dal mese di due anni fa a questo) e la
serie giornaliera (`DAYS = 731`) le interpola con una *smoothstep* più un rumore
deterministico — una deriva settimanale più una giornaliera — così il passo
**DAY** somiglia a un mercato e non a una curva liscia.

> Le ancore sono i valori pubblicati reali fin dove arrivano; la coda è una
> continuazione plausibile, **non un dato**. Comunque vada, **gli ultimi 60 giorni
> vengono piegati sul prezzo live** (`state.marketPrice`) con una rampa e non con
> un gradino: il grafico finisce dove sta il cluster. Si ricostruisce da solo
> quando cambia `marketPrice` **o** `refuelPrice`.

**I pallini** sono i nostri rifornimenti: uno ogni 20–40 giorni, sempre gli stessi
(seed), ciascuno un po' sopra o sotto il mercato di quel giorno — che è tutto il
punto della vista. **Verde sotto la media, rosso sopra.** L'ultimo, nove giorni fa,
**è la benzina che abbiamo nel serbatoio**: vale `state.refuelPrice`.

**I due controlli** fanno esattamente quello che dicono: `RANGE` (3M · 6M · 1Y ·
2Y) sceglie la finestra fino a due anni indietro, `STEP` (DAY · WEEK · MONTH)
ricampiona la stessa serie facendo la **media** di ogni blocco di giorni. Sotto,
quattro numeri (mercato ora, minimo, massimo, la nostra media pagata) e una riga
che dice **di quanto abbiamo pagato sopra o sotto** la media nazionale nella
finestra.

**Toccando un pallino** quella riga diventa la scheda del rifornimento: data,
insegna, litri, €/L, totale, e lo scarto dal mercato di quel giorno.

> **Trappola pagata due volte.**
> 1. `.pv-foot` era `display:flex`: un contenitore flex **butta via gli spazi**
>    fra i nodi di testo, e la frase usciva «13 stopsin this window». È un blocco.
> 2. Il gruppo display è ruotato di 9° in 3D: `getBoundingClientRect()` dà il
>    **riquadro proiettato** e il click sul canvas cadeva storto. `ev.offsetX/Y`
>    sono nello spazio locale non trasformato dell'elemento — cioè esattamente
>    quello in cui il grafico è disegnato.

### Vista 4 — Report

Le altre tre viste sono per chi guida. Questa è **per chi paga**: trasforma il
registro dei viaggi in una nota spese — cosa è stato percorso, quanto è costato,
quanta parte è dell'azienda — e la manda dove il carburante viene davvero
contabilizzato. È la vista che parla ai **fleet manager**, ed è il motivo per cui
esiste: senza di lei True Cost è un bel numero sul cruscotto e basta.

| Blocco | Cosa mostra |
|---|---|
| Chip periodo | `THIS WEEK` (≤ 7 giorni), `THIS MONTH` (≤ 30), `CUSTOM` |
| Lista a sinistra | i 18 viaggi del registro, con spunta, data, **B/P**, euro |
| Scheda a destra | totale €, km, litri, €/km, L/100 km, €/L pagati |
| Barra split | business / privato in euro e in percentuale |
| Riga IVA | IVA 22 % dentro la spesa business, e il **40 % recuperabile** |
| Chip destinazione | EMAIL · CSV · FLEET PORTAL · EXPENSE · RIDE-HAIL |
| Busta + SEND | l'anteprima **precompilata** di quella destinazione |

**Nessun euro è scritto a mano**, neanche qui: i totali escono dagli stessi
`priceTrips()` che disegnano le righe della cronologia, quindi il report non può
contraddire l'app da cui è stato esportato. Toccare una spunta porta i chip su
`CUSTOM` da solo: un chip che dice «questo mese» mentre la selezione è un'altra
sarebbe una bugia.

Perché il periodo abbia senso il registro è cresciuto da **4 a 18 viaggi su 30
giorni** (un report di flotta si chiede al mese, non al viaggio), e la lista della
cronologia adesso **scorre**: troncare il registro sarebbe stato mentire su cosa
l'app sa. Ogni viaggio porta un campo `biz`: senza la divisione fra aziendale e
privato una nota spese non è una nota spese, è una stima.

#### L'invio è simulato, e lo dice

Nessuna mail parte, nessuna richiesta esce dalla pagina — c'è scritto **sul piede
del pannello**, non nelle note del progetto. Quello che è vero è la **forma**: i
campi che un portale chiede davvero (org, targa, centro di costo, token), la
divisione che una nota spese pretende, la riga IVA che un amministrativo cerca per
prima. L'invio è una macchina a stadi guidata da `dt` come il navigatore — quattro
tappe da 0.9 s, ognuna che cade nel log con la sua spunta — e finisce in una
ricevuta verde con id del report, ora, viaggi, euro e IVA recuperabile.

Le cinque destinazioni non sono cinque bottoni uguali con un nome diverso: ognuna
**precompila una busta con i numeri della selezione** (l'indirizzo e gli allegati
per la mail, l'endpoint e il token per il portale, l'importo rimborsabile per la
nota spese, i €/km per il ride-hailing). Un fleet manager in demo deve vedere i
*suoi* numeri prima di premere qualcosa.

### Vista 5 — Trip management

Le altre quattro rispondono a **quanto costa**. Questa possiede la domanda che
sta sotto e che nessuno pronuncia: **dove finisce un viaggio**.

Non è una domanda di comodo. Se il viaggio finisce con la chiave, il pieno a metà
strada archivia Torino–Milano come **due** viaggi. Se non finisce mai, le due ore
ferme dal cliente stanno **dentro** la guida e ne abbassano la media. La regola
decide cosa dice la nota spese — e nei gestionali di flotta è un'impostazione
sepolta in un menu. Qui è la pagina, e il conto alla rovescia la fa **vedere
mentre lavora**.

Quattro regole, e non sono quattro sfumature della stessa:

| | chiude quando | a cosa serve |
|---|---|---|
| `IGNITION` | il motore si spegne, subito | la chiave *è* il viaggio. Semplice, e sbagliata a ogni sosta |
| `SMART` | il motore **resta** spento per la soglia | il pieno resta dentro il viaggio, tornare a casa no |
| `DESTINATION` | si arriva dove il navigatore era stato puntato | il viaggio finisce dove il guidatore aveva detto che sarebbe finito |
| `MANUAL` | mai da sola | solo END TRIP |

La soglia (2 / 5 / 15 / 30 min) vale per `SMART` e `DESTINATION` — lì è il tempo
di conferma dell'arrivo, che distingue una sosta da un passaggio; sotto le altre
due le chip si **spengono**, perché una chip che non cambia niente non deve
somigliare a una che cambia qualcosa.

`DESTINATION` è l'unica che legge un dato **messo dal guidatore**, e quindi
l'unica che può rispondere onestamente «non lo so»: `arm()` è
`navOn && navLeft <= 0`, cioè il navigatore di Pump map — si sceglie una
stazione, si preme NAVIGATE, e il viaggio si chiude all'arrivo. Senza
destinazione la fascia lo dichiara (*«No destination set — nothing here will
close this trip»*), con una in corso conta i chilometri che mancano. Dire che
non si sa vale più che indovinare, e le altre tre regole restano lì per chi
quella risposta la vuole comunque.

> Al suo posto c'era `PARKED` (fermo per N minuti, motore acceso o spento). Si
> dimostrava bene, ma diceva quasi la stessa cosa di `SMART` con un sensore
> diverso, e non portava nel pannello nessuna informazione che l'app non avesse
> già. È stata tolta il 2026-09-05.

Sopra: lo stato (`RECORDING` / `PAUSED` / `CLOSING IN 3:34` / `ENGINE OFF ·
TRIP HELD OPEN`), il totale in euro e quattro celle — durata, km registrati,
€/km medi e **€ fermi**. Sotto: PAUSE, SPLIT HERE, END TRIP.

In mezzo la **fascia**, che cambia mestiere: in movimento dichiara la regola in
vigore e quanto costa un'ora di minimo; a vettura ferma diventa il conto alla
rovescia — anello ambrato che **si svuota**, «*Engine off for 1:24 — closing in
3:35*», e il bottone **KEEP OPEN**, che tiene aperto il viaggio finché la
condizione non si libera. *A fuel stop is not the end of a trip.*

#### I contatori sono suoi

Il modulo non legge solo `state.tripKm`: tiene `tKm`, `tLit`, `tEur`, `tSec`,
`idleSec`, `idleEur` **propri**, e li muove sui delta. Deve poterlo fare, perché
**PAUSE** significa esattamente questo — la vettura continua a guidare e il
cluster continua a contare, ma il registro sta fermo. Quei km non finiranno in
nessuna nota spese.

Al boot il modulo si **innesta sul viaggio già in corso**: la demo si apre con
48 km fatti, e una pagina che dicesse 0.0 km mentre Cost history ne dice 48
sarebbe la solita contraddizione.

#### Il motore, e il costo di stare fermi accesi

Una regola che chiude a motore spento non ha niente da mostrare senza un motore
da spegnere. Quindi il cockpit ha una **chiave**: `state.engineOn`, il bottone
ENGINE sul deck e il tasto «E». Spento, la vettura scende a zero da sola, i giri
cadono a zero e né pedale né slider possono chiedere velocità. Quello che **non**
fa è fermare il viaggio: quello lo decide la regola, ed è tutto il punto.

Con la chiave arriva il numero che il simulatore aveva sempre arrotondato via:
`IDLE_LH = 0.7` L/h. A vettura ferma con il motore acceso il costo del viaggio
**sale senza che salgano i km** — è l'unico costo di tutto il cruscotto che non
compra un metro. La cartolina di fine viaggio lo dice per esteso: *«fermo con il
motore acceso per 3:34, € 0.08 — l'1 % di questo viaggio, su nessuna distanza»*.
Chiude una domanda che era aperta in [08-domande-aperte.md](08-domande-aperte.md).

Gli **€/km istantanei restano 0** a vettura ferma: a 0 km/h non sono definiti.
Il costo del minimo entra da `state.tripCost`, non da lì.

#### La cartolina di fine viaggio

Niente si archivia finché qualcuno non dice **di chi era**. Un viaggio che si
chiude apre una card sopra il pannello: percorso, orari, km, durata, euro, €/km,
la riga ambrata del minimo, e tre chip — BUSINESS / PRIVATE / COMMUTE — già
posate sulla risposta probabile (la politica scelta, o giorno feriale 07–20).

Tre uscite: **SAVE**, **MERGE WITH PREVIOUS** — che disfa l'ultimo taglio, perché
un taglio sbagliato è peggio di nessun taglio, e resta spento se il viaggio
precedente non è uno dei nostri — e **DISCARD**, l'unico posto dell'app dove una
cifra in euro si può buttare via.

Quando la regola scatta sotto una politica che **già sa** di chi è il viaggio, la
card non serve: si archivia da sé e lo dice sulla fascia.

Un viaggio da meno di 200 m (`MIN_KM`) non si registra affatto: una manovra in un
parcheggio non è un viaggio, e una regola che ne deposita uno a ogni spegnimento
riempirebbe la nota spese di niente.

#### Due specie di viaggio nello stesso array

Il viaggio salvato entra in cima a `TRIPS`, con la sua riga in Cost history e la
sua riga nel Report — e se le chip del Report descrivono un **insieme** (non una
scelta a mano) ci entra già spuntato: una nota spese «ultimi 30 giorni, business»
non può saltare la guida finita un minuto fa.

Ma porta `fixed: true`, e quella parola è tutta la differenza. I diciotto finti si
dichiarano **fisicamente** e lasciano che `PRICES` li trasformi in euro, perché un
euro scritto a mano invecchia appena l'API risponde con un mercato diverso. Questo
non ne ha bisogno: i suoi euro sono stati **misurati litro per litro** sul
carburante che era davvero nel serbatoio mentre guidava. Quindi `priceTrips()` lo
scavalca — il mercato si muove, i diciotto lo seguono, e il nostro resta esattamente
come è stato misurato.

Da qui la separazione fra `priceTrips()` (ridà il prezzo) e `paintRows()` (ridisegna
le righe): il registro cambia ora per **tre** motivi diversi — un prezzo nuovo, un
viaggio archiviato in cima, una fusione che ha fatto crescere quello che c'era già.

#### RESET è una cosa sola

Il tasto sul cluster e quello sul deck chiamavano `resetTrip()`. Ora chiamano
`hardReset()`, che azzera i contatori **e** riapre il viaggio registrato. Tenuti
separati, dal primo reset in poi le due pagine avrebbero raccontato due storie.

### Ritmi

Testo dei numeri 4 volte al secondo, canvas ~16 volte, e il `€/km` in alto a ogni
frame: la mappa si ridisegna **solo se è la vista aperta** (e solo se il navigatore
non la sta coprendo), ma i dati (barre, posizione delle stazioni) si aggiornano
sempre, così cambiando scheda non c'è mai un buco nella storia.

Le due viste «ferme» costano quasi niente: un viaggio passato è una fotografia, e
**il grafico dei prezzi è storia** — si ridisegna a evento, e nel tick lento solo
se `state.marketPrice` è cambiato, cioè quando l'API risponde. Il navigatore
invece gira sempre, anche mentre guardi un'altra vista, e così l'invio del report:
`sendFrame(dt)` sta accanto a `navFrame(dt)` nel battito dell'app.

Accanto a loro c'è ora **`tmFrame(dt)`**, e per la stessa ragione portata più in
là: un viaggio continua a registrarsi *e a chiudersi da solo* mentre stai
guardando la mappa. La regola non si mette in pausa perché hai cambiato scheda —
sarebbe l'unico modo di renderla inaffidabile. Il **disegno** invece sì:
`tmPaint()` gira solo se la vista aperta è `trips`, come tutte le altre.

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
