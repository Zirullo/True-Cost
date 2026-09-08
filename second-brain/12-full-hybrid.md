# 12 · Il Full Hybrid

Terza motorizzazione, viva in `index.html` dal **2026-09-07**. Non si attacca mai
a una presa: compra benzina, al litro, alle stesse pompe e agli stessi prezzi
dell'ICE. Quello che la rende un profilo a sé è che **il risparmio se lo guadagna**,
chilometro per chilometro, invece di dichiararlo in un coefficiente.

## Perché non paga il debito dei nomi

Il roadmap assegnava al PHEV lo scioglimento di `tripLiters` / `instantL100` /
`t.l100`, che sotto il BEV portano kWh. **Il full hybrid non lo forza**: il suo
libro contabile è in litri e basta. L'energia che muove nel tampone vive in un
campo separato, `buffKwh`, e **non entra mai nel conto in euro**. Il debito è
rimasto intero ed è rimasto del PHEV, che l'ha poi saldato — [13](13-phev.md).

## `liquid` contro `hasGears`

Prima erano la stessa domanda, perché le auto erano due. Il full hybrid le separa:
è un **e-CVT** — nessun rapporto discreto — che però compra benzina al litro.

| profilo | `liquid` (i libri) | `hasGears` (la trasmissione) |
|---|---|---|
| `ice` | sì | sì |
| `hev` | sì | **no** |
| `bev` | no | no |

`liquid` decide unità, valuta, tema, riga della tariffa di casa, etichette del
Report: ~14 punti che prima leggevano `hasGears` per sbaglio. `hasGears` è rimasto
in tre soli posti veri: il blocco cambio, la scelta della marcia allo switch, e il
render.

E il render non si biforca più: ogni profilo porta `subVal(state)` e
`gearVal(state)`, e i due slot sotto al tachimetro chiedono al profilo.

## La curva: solo il motore

```js
per100: v => PT.ice.per100(v) * (0.90 - 0.10 * Math.exp(-v / 45))
```

È la curva benzina moltiplicata per il guadagno del ciclo **Atkinson**, più forte
a basso carico dove un motore convenzionale spreca di più. Legge `PT.ice.per100`
invece di copiarne i coefficienti: ritarare il benzina ritara l'ibrido con lui.

**Dentro questo numero non c'è nessun recupero.** Il fattore va da 0.80 a fermo a
0.894 a 128.

| km/h | ICE | HEV solo motore |
|---|---|---|
| 30 | 4.37 | 3.71 |
| 90 | 5.43 | 4.81 |
| 128 | 7.14 | 6.38 |

## Il tampone

`buffCap: 0.50` kWh — e non è il pacco: è la **finestra di SOC** che un ibrido usa
davvero, una fetta in mezzo a una batteria più grande. È il motivo per cui
l'autonomia elettrica è di un paio di chilometri e non di cinquanta, ed è quello
che dà alla barretta nel pannello un movimento visibile invece di un tremolio.

Cambio fra le due energie: `kwhPerL: 3.10` — la benzina ha ~9.7 kWh/L e questo
motore ne mette in strada circa il 32%.

**In frenata** l'energia cinetica entra nel tampone (`regenEta 0.68`, tetto
`regenKwMax 40 kW`), e il termico è staccato: `stepU = 0`.

**In trazione** il motore elettrico può portare una quota del passo — tutta sotto i
35 km/h, sempre meno mentre l'aria comincia a costare, mai oltre `assistKwMax 25 kW`.
E **si smorza quando il tampone si svuota** (sotto il 35% di carica la quota cala in
proporzione): un ibrido che ha appena speso la sua carica guida come il benzina per
un po', ed è la ragione onesta per cui la stessa auto rende numeri diversi sulla
stessa strada.

**La ricarica dal motore** ha bisogno di un **latch**, non di una soglia: una volta
partita risale fino al 60% e solo lì molla. Un semplice «sotto il 30%» si sarebbe
fermato appena risalito, parcheggiando l'auto sul fondo del suo tampone per sempre.
Larga com'è, evita anche di far ciclare energia per niente in autostrada, dove ogni
andata e ritorno attraverso la batteria ne perde un decimo (`chargeEta 0.90`).

**Da fermo** il motore è **spento**. Si accende solo se il tampone è sotto il 30%,
e allora gira a `idleCharge 1.6 L/h` mettendo quello che brucia nel tampone, non
nella strada. Modo `CHG`.

## Il quadrante: e-CVT

I giri non seguono la velocità, seguono la **potenza richiesta**:

```js
kw        = stepU/dt · 3600 · kwhPerL
rpmTarget = motore acceso ? clamp(1000 + kw·45, 1000, 4200) : 0
```

Cadono a **zero** appena l'auto va in elettrico mentre è ancora in movimento, e
schizzano su una stoccata di gas senza che il tachimetro si muova. È il tratto più
riconoscibile di un full hybrid vero. Lo slot che sull'ICE porta la marcia qui
porta il **modo**: `EV` · `HV` · `CHG` · `OFF`.

## La velatura in cabina, e la trappola che nascondeva

L'ibrido è l'unica delle tre cabine **di due colori insieme**: teal in alto, verso
il vetro, che scende a neutro e diventa ambra sulla console, dove il motore c'è
ancora. Teal e non l'azzurro cielo del BEV, così un'occhiata alla parte alta non
può scambiarli; e mai **verde**, che su questo cockpit significa una cosa sola.

La prima versione **usciva identica al benzina**, e per due ragioni che vale la
pena aver scritte:

1. **`#tint` copre tutta la scena, ma `--dashmask` ne scopre solo l'abitacolo**:
   la banda y 0.31 → 1.00. Il mio stop freddo stava a 0%, cioè su nel parabrezza,
   e sulla plancia arrivava solo il tratto tan→marrone — la stessa famiglia di
   tinta dell'ICE
2. **L'angolo**: a 168deg, su una scena larga 2.17 volte la sua altezza, il
   gradiente corre in diagonale. Spostati gli stop dentro la banda, il risultato
   era freddo *a sinistra* e identico al benzina al centro plancia — non «freddo
   sopra, caldo sotto» ma «freddo a sinistra», che di un ibrido non dice niente

ICE e BEV se lo permettono perché sono di una tinta sola da capo a fondo. Chi deve
cambiare **attraverso** l'abitacolo va a **180deg**, e allora gli stop sono
frazioni dell'altezza:

```css
opacity:.30;
background:linear-gradient(180deg,
           #6ec9c9 32%, #adb5b2 60%, #c79a58 78%, #8a5a2a 100%);
```

Lo stop di mezzo è quasi neutro apposta: interpolando dritto da teal ad ambra si
passa per il verde, e `mix-blend-mode: color` lo metterebbe sul cruscotto a piena
forza. Lasciando cadere la saturazione, la transizione legge come luce che cambia.

Misurato sul gradiente risolto, la differenza di tinta rispetto all'ICE è **147°
a sinistra e 148° a destra** — uniforme, come dev'essere — e il **37%**
dell'abitacolo visibile porta la tinta fredda.

Il quadro strumenti invece resta **ambra**: è un'auto che va a una pompa, e
vestirlo di ciano direbbe una bugia che il pannello dei prezzi poi smentisce.
Il suo segno distintivo è la barretta del tampone.

## Il km si ferma a zero

È la differenza dichiarata rispetto al BEV, e il codice non deve mai confonderla.
Un'auto a batteria spende subito l'energia della frenata e **il km costa meno di
niente**. Un ibrido può solo **parcheggiarla**, e spenderla dopo come benzina che
non brucia. Quindi il costo del km ibrido **si ferma a zero e non va sotto**:
alzare il piede ferma il carburante, ma niente può restituire un litro già bruciato.

## L'accelerazione si paga — ed è stato l'ibrido a scoprirlo

È il full hybrid che ha fatto emergere un difetto vecchio: `per100` è una curva di
**crociera** e nessuno pagava per *raggiungere* una velocità. Finché niente
restituiva energia non si vedeva; la frenata la restituisce, e un'auto accreditata
di energia cinetica mai addebitata riceve carburante gratis. È saltato fuori
subito: dieci fermate in città venivano **zero**.

Il termine `accCost` che ne è uscito vale su **tutti e quattro** i profili e vive
in [04-modello-costi.md §3b](04-modello-costi.md) — formula, coefficienti e il
motivo per cui a velocità costante è esattamente zero. Qui basta la conseguenza:
il modello vivo adesso dice quello che il log diceva già, cioè che **in città si
spende più che in autostrada**. Prima diceva il contrario, e contraddiceva i suoi
stessi diciotto viaggi.

### Due cose che si vedono solo qui

I readout smorzati sono in [04 §3c](04-modello-costi.md). Quello che quella nota
non può mostrare è la **forma della salita**.

Il €/km dell'ICE partendo da fermo, ogni 200 ms: `0.22 · 0.47 · 0.65 · 0.78 · 0.87
· 0.93 · 0.97 · 1.00 · 1.02`. Sull'ibrido si legge anche il gomito a ~1.8 s: è il
tampone che finisce e il termico che subentra.

**Una guardia che serve davvero.** Un frame di durata zero esiste — il primo lo è
sempre, perché il loop imposta `lastFrame` e poi si chiama subito, e un browser può
consegnare due callback sullo stesso timestamp — e rendeva `stepU/stepKm` uno
`0/0`. Era un NaN che il frame dopo sovrascriveva in silenzio. **Con i readout
smorzati un NaN non si laverebbe più via**, perché tutto ciò che gli si somma
resta NaN. Un frame che non ha coperto strada non dice niente sul consumo, quindi
la cifra sta ferma.

## I numeri, misurati

Finestre pulite, stato interno letto direttamente, non il display arrotondato:

| | ICE | HEV | rapporto |
|---|---|---|---|
| 128 km/h costanti | **7.14** L/100 | **6.43** L/100 | 90% |
| stop-and-go (7 ripartenze) | **11.48** | **4.72** | 41% |
| fermo acceso, 60 s | 0.0117 L · 0.025 € | **0.0000 L · 0.00 €** | 0 |

E gli invarianti, frame per frame su un ciclo urbano: i litri del viaggio non
scendono **mai**, il €/km non va **mai** sotto zero, il tampone resta dentro i suoi
limiti, e tutti e tre i modi compaiono.

## Il mese guidato una terza volta

I diciotto viaggi hanno un terzo gemello, e nasce **al contrario** di quello
elettrico. Il gemello BEV viene dalla curva, perché un'auto a batteria non ha un sé
benzina da cui scendere. L'ibrido ce l'ha: **è** quella macchina, con un recupero
sopra. Quindi il suo consumo è la cifra benzina di *quel* viaggio, scontata di
quanto quel viaggio era stop-and-go:

```js
const HEV_F = { road: 0.88, mix: 0.72, city: 0.58 };
```

Non sono inventati: sono quello che il **modello vivo** restituisce guidato alle
velocità medie di quei viaggi. Una curva di crociera non avrebbe potuto dirlo — le
cifre benzina tipate portano le code e i semafori dentro di sé, ed è proprio quelle
che l'ibrido restituisce.

E la freccia si rovescia: la Torino–Milano è il viaggio *conveniente* del mese per
il benzina e quello *caro* per ibrido ed elettrico, perché i loro mesi sono
dominati dai viaggi urbani. Le cifre dei quattro, viaggio per viaggio e sul mese,
stanno in [13-phev.md](13-phev.md) in copia unica.

## Il fotogramma di apertura

Ogni veicolo si sale con **lo stesso viaggio già alle spalle**: i 48.09 km della
foto, alla sua velocità di crociera. Quello che cambia è il conto — ed è la tesi
del prodotto messa sullo schermo prima che qualcuno tocchi un comando. La tabella
dei quattro sta in [04-modello-costi.md](04-modello-costi.md), in copia unica.

L'ICE tiene la coppia della foto, `openUnits` / `openCost`, incoerenza ereditata
compresa. Gli altri **non** possono permetterselo: dicono quello che la loro curva
dice di quel viaggio, così nessun numero sul loro pannello ne contraddice un altro.
Prima aprivano a zero, e al momento dello scambio non c'era niente da confrontare.

Due conseguenze da tenere a mente:

- **La tariffa di casa ri-prezza tutto il viaggio del BEV**, non solo la carica in
  batteria. Finché non si compra a un totem (`chargedOut`) ogni km di quell'auto
  viene da quella stessa presa a quello stesso prezzo, quindi gli euro sono
  semplicemente l'energia per lo slider. Salire sul BEV *dopo* aver mosso lo slider
  legge già la tariffa nuova, perché `fresh()` prezza lì l'apertura: sono gli stessi
  due gesti nell'altro ordine. Un confronto `tripKm === OPEN_KM` era stato provato
  e **non funziona**: la decelerazione è asintotica, l'auto non arriva mai
  esattamente a zero e il chilometraggio si scosta subito
- **Il campionatore delle barre va ribasato** su nuovo viaggio. `resetCounters()`
  ribasava già il registratore; `sampleTrip()` no, e la sua guardia `dKm < 0`
  copriva solo il caso in cui l'altra auto avesse percorso *meno*. Un salto
  positivo fra due contachilometri diventava una barra da 200 m che non è mai
  esistita. Ora `newTrip()` chiama anche `resetSampler()`

## Il debito che resta

- Il **coasting**: in rilascio senza freno tutti e quattro pagano ancora il consumo
  di crociera, mentre un motore vero taglia l'iniezione. È il simmetrico del termine
  di accelerazione, e non è stato fatto: il modello sovrastima il consumo, ma lo
  sovrastima allo stesso modo per tutti
- Il tampone in guida aggressiva vive fra il 9% e il 33%: realistico, ma la
  barretta respira meno di quanto potrebbe
