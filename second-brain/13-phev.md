# 13 · Il Plug-in Hybrid

Quarta motorizzazione, viva in `index.html` dal **2026-09-07**. È il full hybrid
di [12-full-hybrid.md](12-full-hybrid.md) con un pacco che si riempie **a casa**, ed
è la prima auto qui dentro che **spende due energie nello stesso viaggio**. Per
questo è lei che ha saldato il debito dei nomi, come il roadmap aveva previsto.

## Il debito dei nomi, saldato

Prima del veicolo, in un commit a sé: `tripLiters` → `tripUnits`, `tankLiters` →
`tankUnits`, `instantL100` → `instantPer100`, `t.l100` → `t.per100`, e con loro i
contatori del registratore e i totali del Report. Nessuna cifra si è mossa —
apertura e crociera ICE lette identiche prima e dopo.

Poi la colonna nuova: **`tripKwh` accanto a `tripUnits`**, e `tripCost` che è la
somma di due acquisti, ognuno al prezzo del posto da cui viene. Tre auto su quattro
hanno il secondo termine a zero e non se ne accorgono.

## Le due semplificazioni, dichiarate

Sono scelte di prodotto, non modello. Vanno lette come tali.

**1. L'autonomia è una distanza, non una curva.** `evRangeBase 38` ± `evRangeVar
8.5` km, estratta a ogni ricarica, e il pacco si svuota **uniformemente lungo di
essa** comunque si guidi. Quindi il km a spina **costa uguale a 30 e a 128**, che è
l'unica cifra del simulatore a non curarsi di come si guida.

Conseguenza da tenere a mente, ed è visibile in demo: nello stop-and-go il PHEV a
spina fa **0.0708 €/km contro i 0.1960 del BEV**, che paga l'accelerazione mentre
lui no. In città il plug-in risulta *più economico dell'elettrico*, e non perché lo
sia: perché il suo consumo è a forfait. È il prezzo dei «38 km fissi».

**2. Il km si ferma a zero anche in elettrico.** Sul pacco, fisicamente, dovrebbe
andare sotto: quella batteria la frenata la prende davvero, come sul BEV. Il PHEV
tiene **un solo invariante da capo a fondo**, quello dell'ibrido. Zero frame
negativi, misurati su un ciclo intero. È il punto in cui questo profilo è
consapevolmente più gentile con il racconto che con la fisica.

Il pacco resta l'unico numero libero, ed è quello vero: **11.4 kWh su 38 km = 30
kWh/100 km**, cioè quello che restituisce un plug-in pesante con un termico morto a
bordo. Chi controlla la cifra la trova giusta.

## Sotto la spina è l'ibrido, più pesante

```js
per100: v => PT.hev.per100(v) * 1.06
mass: 1750          // contro i 1450 dell'HEV: il pacco pesa anche da scarico
```

Legge `PT.hev` invece di copiarlo, come `hev` fa con `ice`: ritarare uno sposta
tutta la linea. Tampone, modi `EV`/`HV`/`CHG`, e-CVT, motore spento da fermo:
tutto identico all'ibrido.

Il 1.06 sulla curva non è però tutto il costo del peso. Nello stop-and-go il PHEV
scarico fa **0.4692 €/km contro 0.3748** dell'ibrido, cioè **+25%**, perché il
termine di accelerazione (`accCost`, [12](12-full-hybrid.md)) vede i 300 kg in più.
Un plug-in scarico è un ibrido che si porta dietro la sua stessa promessa.

## La spina: `P`, e due cose che non finge

`P` riempie il pacco alla tariffa dello slider di casa. **Non c'è ricarica al
totem**: quest'auto compra elettricità a casa e benzina in strada, ed è quella riga
netta che rende le due cifre sul cartellone fuori un *argomento* invece che un
menu.

Due onestà nella ricevuta:

- **funziona anche in marcia** — chiesto esplicitamente, per poter mettere le due
  metà dell'auto una accanto all'altra in demo senza accostare. La ricevuta dice
  *«Charged overnight at home»*, così lo schermo non finge che sia successo lì
- **l'autonomia restituita è un'estrazione**, non i 38 della brochure. Sei estrazioni
  misurate: `45.3 · 45.2 · 38.2 · 31.3 · 44.1 · 40.9`. Attaccare due volte la stessa
  auto non dà lo stesso numero, che è la cosa più vera che un plug-in abbia da dire

## Il fotogramma di apertura: si sale a batteria vuota

Gli stessi 48.09 km di tutti, spesi come li spende questa macchina: **38 elettrici
e 10.09 ibridi**. Quindi arriva **scarica**, che è lo stato richiesto — si tocca il
pedale ed è un ibrido, finché qualcuno non la attacca.

L'apertura usa i **38 dichiarati e non un'estrazione**: il fotogramma di
riferimento è l'unico posto dove un dado costerebbe più di quanto vale, e ogni
altra auto apre riproducibile. La lotteria comincia dalla prima ricarica.

Sui 48.09 km della foto sono **0.68 L + 11.40 kWh = € 4.25**, e si infila fra
ibrido ed elettrico senza che sia stato tipato niente: è la curva a dirlo. La
tabella dei quattro sta in [04-modello-costi.md](04-modello-costi.md).

## Il quadrante segue l'energia, non l'auto

`relabel()` riscrive i due slot che rispondono all'**energia** e non alla vettura:
il consumo del viaggio e il prezzo di quello che si sta spendendo. Sulle altre tre
si scrivono una volta sola alla porta; qui **si ribaltano in mezzo a una guidata**,
senza che nient'altro sul pannello si muova. È la cosa più eloquente che
quest'auto abbia da mostrare, ed è la ragione per cui è una funzione e non sei
righe dentro `applyProfile()`.

`lastPlug` nel render evita di riscrivere cinque nodi DOM sessanta volte al secondo
per dire la stessa cosa.

**La barra livello resta il serbatoio**, sempre, su ogni auto che ne ha uno.

Sotto, il plug-in ha **due gauge affiancate** e nessuna delle due se ne va mai
(gruppo `vs-pack`, che sostituisce il `vs-buff` a tutta larghezza dell'ibrido):

| | larghezza | colore | cos'è |
|---|---|---|---|
| **PLUG-IN** | 74 px | viola `#a58cf0` | il pacco, **comprato** a un muro al prezzo dello slider |
| **BUFFER** | 36 px | ciano `#5fdcff` | la fetta che l'auto si **guadagna** frenando |

Le larghezze sono le taglie: 11.4 kWh contro 0.50. E i due colori non sono
decorazione — dicono la differenza fra le due metà di questa macchina: energia
**pagata** contro energia **guadagnata**.

**Il pacco vuoto resta in vista.** Una prima versione gli faceva passare la mano al
tampone quando finiva, e su una macchina in cui *«la batteria è finita»* è metà di
quello che c'è da vedere quella era la metà sbagliata da nascondere. Adesso la
scatola vuota **è** la lettura, e a scendere è la scritta: `#b9a6f0` con carica,
`#5c5175` a zero. Così «scarico» si legge invece di dover essere notato.

Misurate: 0/70 all'apertura con l'etichetta già spenta, 70/70 appena attaccato,
35/70 a metà, e 0/70 con il tampone a 32/32 quando la carica è finita e l'ibrido
ha ripreso a respirare. HEV e ICE non vedono questo gruppo affatto.

## Il tema: viola, e per esclusione

Costruzione identica all'ibrido — `180deg`, perché deve cambiare **attraverso**
l'abitacolo, con la stessa fermata neutra in mezzo (interpolando viola→ambra si
passa per un marrone fangoso, e `mix-blend-mode: color` lo metterebbe intero sul
cruscotto).

```css
#stage.phev #tint{
  opacity:.30;
  background:linear-gradient(180deg,
             #9a86e0 32%, #b0adb4 60%, #c79a58 78%, #8a5a2a 100%);
}
```

Viola e non un teal più freddo perché **queste due auto non devono confondersi a
colpo d'occhio**, e il viola è l'unica tinta fredda che questo cockpit non abbia
mai usato: non il cielo del BEV, non il teal dell'ibrido, e lontanissima dal verde,
che qui significa una cosa sola. Il quadro strumenti resta **ambra**: va a una
pompa. Il pulsante nella barra porta lo stesso gradiente dell'ibrido con la metà
fredda spostata su viola.

## Il mese, guidato una quarta volta

Costruito **sopra** il gemello ibrido, perché è letteralmente quello che
quest'auto è a pacco scarico: cifra ibrida × 1.06 per quello che resta del viaggio
dopo la carica.

**Ogni viaggio apre a pacco pieno**, e non è generosità: i diciotto stanno su
diciotto **giorni diversi**, quindi un'auto caricata di notte li incontra tutti
pieni. Il giorno in cui un secondo viaggio cadrà su un giorno già guidato, è lì che
andrà letto quello che il primo ha lasciato — il commento nel codice lo dice.

L'autonomia è estratta **per viaggio** riusando `t.u`, il jitter stabile che il log
già usa per i prezzi: stesso viaggio, stessa autonomia a ogni caricamento, e due
viaggi non ne condividono una. Il mese deve contenere il commute che ce l'ha fatta
e quello che per poco no.

Sugli stessi 1194 km — **è l'unica copia di questa tabella nel second brain**, e i
numeri sono **misure**, non costanti: dipendono da `state.marketPrice`, quindi vanno
ri-letti dal Report dopo ogni ritaratura di una curva, non copiati altrove.
Rilevate il 2026-09-07, dopo la ritaratura del BEV:

| | totale | €/km | consumo |
|---|---|---|---|
| ICE | € 151.96 | 0.127 | 81.7 L |
| HEV | € 121.31 | 0.102 | 65.3 L |
| **PHEV** | **€ 107.92** | **0.090** | **32.7 L + 188.4 kWh di casa** |
| BEV | € 131.33 | 0.110 | 217.7 kWh |

Il plug-in vince **il mese** e perde **la città**, ed entrambe le cose sono
interessanti:

| | Torino–Milano, 142.6 km | City · Torino, 19.4 km |
|---|---|---|
| ICE | € 16.71 | € 3.54 |
| HEV | € 14.71 | € 2.05 |
| PHEV | € 14.06 · 5.74 L + 11.4 kWh | € 1.73 · **0.00 L** + 6.9 kWh |
| BEV | € 19.43 | € 1.59 |

> ⚠️ Queste due colonne sono **anteriori alla ritaratura del BEV del 2026-09-07**
> e non sono state ri-misurate. La riga BEV è quella che si è mossa di più: la
> Torino–Milano rincara (22.0 kWh/100 contro 20.2), la City · Torino cala
> (12.2 contro 14.4). **Il verso del confronto regge, le cifre no.**

In autostrada il pacco è l'8% del viaggio e il PHEV è appena meglio dell'ibrido. In
città non brucia **niente**. Il mese è dominato dai viaggi corti, ed è per questo
che vince — cioè per la stessa ragione per cui un plug-in vero vince o perde a
seconda di chi lo guida, che è esattamente il punto del prodotto.

## Il Report: la cella che smette di mentire

Sul plug-in la cella `L / 100 KM` diventa **`KWH FROM HOME`**. Non per nascondere
la cifra — sta su ogni riga sopra — ma perché un L/100 km mensile misto è il numero
**più fuorviante** che un plug-in produca, e la cella accanto alla spesa totale
deve portare quello su cui un amministrativo può agire: quanto di questo mese è
uscito dal muro invece che dalla pompa.

Le righe del viaggio invece portano **entrambe**: `142.6 km · 5.74 L + 11.4 kWh`.
Una delle due da sola è un'altra macchina.

## Il debito che resta

- **Il coasting**, ancora: in rilascio senza freno tutti e quattro pagano la
  crociera. Sovrastima uguale per tutti — [12](12-full-hybrid.md)
- **Un giorno, due viaggi**: il gemello storico non sa ancora dividere una carica
  fra due viaggi dello stesso giorno, perché il log non gliene offre l'occasione
- **Il forfait elettrico**: vedi la semplificazione 1. È la cosa da rifare per prima
  se qualcuno chiede perché in città il plug-in batte l'elettrico
