# 14 · Daily commute — la tratta che torna

La **sesta vista** del display centrale, aggiunta il **2026-09-11** e rifatta il
**2026-09-15**. È l'unica che non parla di *un* viaggio: parla di un'**abitudine**.

Le altre cinque rispondono a «quanto è costato questo viaggio». Questa risponde
alla domanda che un guidatore si ripete trecento volte l'anno e a cui nessun
cruscotto gli ha mai dato un numero: **quanto mi costa andare al lavoro** — e,
mentre lo sta facendo, **se oggi sta spendendo più o meno del solito**.

## Le tre cose che dovevano essere vere

### 1 · Si imposta, non si indovina

Un *commute* qui è **un giorno, un'ora e una distanza** — che è tutto ciò su cui
un'auto può davvero contare — e le scrive il guidatore. Nessuna mappa, nessun
percorso: una mappa avrebbe promesso una precisione che questa demo non ha.

**Se ne possono impostare più di uno**, perché quasi nessuno ne ha uno solo: il
tragitto per l'ufficio, la corsa all'altro stabilimento, il lunedì in un'altra
città. L'editor (`#cm-set`, sopra il pannello) ha nome, da/a, distanza a tratta,
ora di partenza, tipo di strada (autostrada · misto · città), i sette giorni
della settimana e «**drives back too**», che raddoppia i chilometri del giorno.
Salvare rigenera lo storico di quel commute; l'**ultimo non si può cancellare**,
perché una pagina sui commute senza nessun commute non avrebbe niente da dire.

I tre di partenza sono finti come i diciotto viaggi del registro, e **datati da
oggi** per la stessa ragione:

| | tratta | a tratta | strada | quando |
|---|---|---|---|---|
| `Torino → Milano` | Torino → Milano Portello | 142.5 km | autostrada | una volta a settimana, 06:20 |
| `Home → Office` | Rivoli → Mirafiori | 23.5 km | misto | lun–ven 07:40 |
| `Office → Plant` | Mirafiori → Grugliasco | 8.5 km | città | lun/mer/ven 13:10 |

Il primo della lista è **la tratta lunga**, perché è quella che il simulatore sta
guidando: centoquaranta chilometri di autostrada, non un salto in città. È anche
quella **ancorata al viaggio aperto** (§ note di taratura, punto 6).

Sono **mezzi chilometri** perché lo slider dell'editor va a passi di 0.5: un
pannello che apre su 23.4 km e mostra 23.5 appena si preme EDIT starebbe
litigando con i propri dati.

Coprono le tre forme di strada di proposito, perché è lì che il confronto fra le
motorizzazioni smette di essere prevedibile: la tratta urbana e quella
autostradale **non sono d'accordo** su quale auto convenga.

### 2 · Si riconosce, e dice se siamo sopra o sotto la media

Con quei tre dati scritti, l'app può guardare il viaggio in corso e dire **quale
commute è**, e **quanto ne è sicura** — e, cosa che conta di più, dire «nessuno»
quando non è nessuno. La fascia sotto la testata è quella frase.

Tre segnali, e il punteggio dice quanto ha preso da ciascuno:

- **L'ora** — pieno entro un quarto d'ora dalla partenza abituale, zero oltre
  un'ora e mezza. Pesa 0.4.
- **La strada** — quanto la distanza percorsa è vicina a **una tratta** o, per
  un viaggio mai tagliato all'arrivo, a **tutte e due**. Pesa 0.6. La curva sale
  fino alla tratta e poi cade: un terzo oltre, non è più quel commute.
- **Il giorno** — un segnale **morbido**, non un filtro: `× 0.55` se oggi non è
  un giorno in cui quel commute si fa. Un tragitto fatto di domenica è sempre
  quel tragitto, è solo meno probabile, e la fascia lo dice invece di rifiutarsi
  di vederlo.

Soglie: **≥ 0.75 «Recognised»** (verde), **≥ 0.48 «Looks like … N % sure»**
(blu), sotto «No commute recognised» (grigio). Il bottone **It's this one** fissa
a mano il commute a schermo; un nuovo viaggio scioglie il fermo, perché il
prossimo tragitto se l'etichetta se la deve guadagnare da solo.

Accanto alla fascia c'è la **pastiglia**: `▲ 34 %` in rosso o `▼ 12 %` in verde,
sopra la scritta `VS USUAL`. È lo scarto fra gli **€/km del viaggio in corso** e
gli €/km medi di quel commute in quella macchina — per chilometro e non per
tratta, perché mezza tratta non ha ancora speso i soldi di una tratta intera e il
confronto deve reggere dal primo chilometro.

La pastiglia **non si misura sulla media del grafico**. Quella media è prezzata
su sei mesi di listini, e un pieno fatto questa settimana a un prezzo da cui il
mercato si è già mosso sarebbe letto come guida sbagliata. Il metro è questo
commute **riprezzato al pompa di oggi**, sul tratto già percorso, con traffico
normale (`cmToday()`): quello che resta nella pastiglia è **la guida** — la
velocità, la coda, il minimo — che è l'unica parte su cui il guidatore può fare
qualcosa. La barra bianca del grafico porta la stessa deviazione su una giornata
intera: «se il resto della strada va come il tratto alle spalle, la giornata
finisce qui».

> La pastiglia parla **solo** del commute che è a schermo, e **solo** finché è
> quello che si sta guidando. Si può stare leggendo *Office → Plant* mentre si
> guida *Home → Office*: in quel caso la pastiglia resta un trattino e la fascia
> scrive «*not the one on screen*». Una freccia che confrontasse questa mattina
> con una strada su cui non siamo sarebbe peggio di nessuna freccia.

### 3 · Una macchina alla volta, e il confronto si fa cambiandola

**Il pannello mostra una sola motorizzazione: quella sotto di te.**

Fino al 2026-09-15 questa vista metteva le quattro motorizzazioni una sotto
l'altra, con i loro litri e i loro euro affiancati. Roberto l'ha fermata, e aveva
ragione: **non è uno schermo reale**. Nessuna auto su una strada sa quanto
avrebbero speso le altre tre. Era un grafico da presentazione travestito da
cruscotto, e questa demo vive esattamente del contrario.

Il confronto si fa nel modo in cui lo fa il resto dell'app: **si cambia
motorizzazione sulla barra di regia in alto a sinistra e si rilegge lo stesso
commute**. Cambia il numero grande, cambiano le barre, cambia la media, e cambia
la testata dello storico, che è l'unico posto rimasto in cui il pannello nomina
l'auto: `18 RUNS · PETROL · AVG € 31.90`.

Sotto al grafico c'era una nota che diceva di chi erano quelle cifre e come si
passava all'auto successiva. **Non c'è più**, in due tempi: prima è caduta la
parte che insegnava il tasto — l'interfaccia cambia da sola quando si tocca la
barra di regia, e una didascalia che spiega un gesto che il pannello fa già da sé
è un manuale d'uso dentro un cruscotto — e poi anche il resto, perché una
didascalia che ripete un'etichetta già a schermo non vale i quarantuno pixel che
teneva. **Quei pixel sono andati al grafico**, che da 96 px passa a prendersi
tutta la colonna: la banda delle barre passa da 55 a **102 px**, e lo stesso
scarto dalla media si legge al doppio dell'altezza.

`cmPrice()` ha quindi **il veicolo nella chiave** insieme ai due prezzi, e
ricalcola solo per la macchina in corso: una alla volta non è un risparmio, è la
regola.

I commute però **restano gli stessi per tutte e quattro**: appartengono al
guidatore, non alla trasmissione. Stessa strada, stessa distanza, stessa ora.
Cambia solo il conto — e non è digitato:

```
CM_KMH = { road: 122, mix: 54, city: 26 }   km/h resi porta a porta
p      = PT[veh].per100(CM_KMH[char])       e ogni auto beve dalla SUA curva
```

Fino al 2026-09-15 la benzina era **digitata** (`CM_P100 = { road: 6.1, … }`) e
le altre tre discendevano da lei con le costanti del registro (`HEV_F`,
`PHEV_W`, `BEV_C`). Il motivo era buono — un viaggio non si guida alla sua
velocità media — ma il prezzo era che **le due metà della pastiglia non erano
la stessa strada**: il commute costava un'autostrada percorsa a 88 km/h in
un'auto che ne stava facendo 128, e la pastiglia leggeva **+77%** sulla benzina
e **+33%** sull'ibrido senza che nessuno avesse guidato male.

Ora le litri e i kWh del commute escono **dalla curva dell'auto sotto di te**,
alla velocità che quella strada rende, e la riga autostrada dice **122** perché
è la strada su cui questa demo sta viaggiando. Il registro dei diciotto viaggi
tiene le sue costanti: quella è storia digitata, e non la si ritara.

#### Il traffico non costa uguale a tutti e quattro

È l'unico punto dell'app che lo dice. Ogni run porta un `tr` fra 0.97 e 1.03 —
il traffico che quella mattina ha trovato, estratto una volta e tenuto. Il tiro
è **stretto e centrato su 1**: un commute è la stessa strada alla stessa ora, e
quello che separa due sue mattine è una coda al casello, non un altro viaggio.
Tirato fra 0.90 e 1.20 produceva mattine più care del 18% della media — una
dispersione che appartiene a un mese di viaggi diversi, non a una strada rifatta
ventisei volte — e trascinava la media stessa cinque punti sopra una corsa
pulita. Adesso nessun run si stacca dalla media più di **±8%**, che è anche il
pavimento della scala del grafico: le barre usano tutta l'altezza senza che una
giornata normale sembri un disastro. L'auto a
benzina lo paga **tutto**: ogni fermata due volte, il carburante per riprendere
velocità e il calore per perderla. Chi la frenata la riprende ne restituisce una
parte:

```
CM_TRAF = { ice: 1, hev: 0.62, phev: 0.62, bev: 0.45 }
```

#### L'auto si carica a casa, una volta, di notte

È la regola che tiene insieme la metà elettrica, ed è quella che un pendolare
vive davvero. Il **plug-in ha un pacco al giorno** e l'**elettrica ha un pacco al
giorno**; tutto ciò che la giornata chiede oltre quel pacco è comprato **in
strada**, a prezzi di strada (`PRICES.chargeAgo`).

Su un tragitto da 23 km non se ne accorge nessuno. Sulla tratta per Milano e
ritorno è tutta la storia:

| | Home → Office (47 km/g) | Office → Plant (17 km/g) | Torino → Milano (285 km/g) |
|---|---|---|---|
| Petrol | 3.71 L · **€ 6.86** | 1.75 L · **€ 3.17** | 18.4 L · **€ 33.22** |
| Full hybrid | 2.64 L · **€ 4.89** | 0.99 L · **€ 1.80** | 15.9 L · **€ 28.58** |
| Plug-in | 0.53 L + 11.4 kWh · **€ 3.83** | 5.1 kWh, **EV only** · **€ 1.28** | 14.6 L + 11.4 kWh · **€ 29.10** |
| Electric | 6.6 kWh · **€ 1.65** | 2.1 kWh · **€ 0.52** | 62.6 kWh (8.6 fuori casa) · **€ 18.67** |

*(Questa tabella sta qui, nelle note, non sullo schermo: qui serve a chi ritara i
numeri, lì sarebbe la bugia che il 2026-09-15 è stata tolta.)*

Due cose cadono da sole dalla fisica e che nessuno ha sistemato a mano:

- **In città il plug-in non accende mai il motore.** La testata allora non parla
  di litri: la cella `L / 100 KM` diventa `KWH / 100 KM`, il sottotitolo smette
  di scrivere «0.00 L +» e le righe dello storico dicono **EV only**. È l'unico
  posto dell'app in cui l'etichetta segue il **viaggio** e non il veicolo.
- **In autostrada il plug-in costa più del full hybrid** (€ 29.10 contro
  € 28.58): il pacco è lo stesso su 285 km come su 17, e il resto del viaggio lo
  fa un ibrido che porta 11.4 kWh di peso morto. Lo si scopre premendo «V» due
  volte, che è esattamente il gesto che questa vista vuole insegnare.

## Il grafico: ogni run rispetto alla media

La colonna di sinistra del corpo. Una barra per run, **la più vecchia a
sinistra**, e la riga tratteggiata verde in mezzo è la **media**. Le barre
**pendono da quella riga** invece di poggiare sullo zero: rosse verso l'alto,
verdi verso il basso.

Disegnate da zero erano una fila di torri identiche che non diceva niente — ed è
stato il grafico del *plug-in su Office → Plant*, dove tutti i run stanno in tre
centesimi, a dimostrarlo. La domanda non è quanto è costato un run: è **se è
costato più del solito**.

- **Scala con un pavimento dell'8 %.** Senza, un commute i cui run stanno tutti
  entro un centesimo si vedrebbe gonfiare l'arrotondamento a piena altezza e
  sembrerebbe ballerino. **Una strada costante deve sembrare costante.**
- **L'ultima barra, bianca, è il viaggio in corso** — ed è una **proiezione**: i
  suoi €/km portati su una tratta intera. Mezzo commute messo accanto a
  ventisei commute interi si leggerebbe come la mattina più economica del mese.
  Compare solo dopo un quinto di strada, e l'asse dice qual è
  (*this run € 9.31*).
- L'asse porta il **minimo e il massimo in euro**: è l'unica scala assoluta del
  grafico, e serve perché le barre sono scarti.

Accanto, lo storico dei run con **freccia e percentuale** per ciascuno, misurata
in **euro** contro la stessa media che disegna la riga tratteggiata — così una
riga e la barra sopra di lei non possono mai dire cose diverse sulla stessa
mattina.

## Lo storico dei run, e perché è staccato dai viaggi

`CM_RUNS = 26` occorrenze camminate all'indietro **sui giorni in cui quel
commute si fa** — per questo il lunedì-Milano mostra diciassette lunedì e quello
feriale un mese di mattine. Un run è **una data più due estrazioni** (il traffico
di quella mattina, e quale pompa riempiva il serbatoio quella settimana) e niente
altro: ogni cifra con un euro davanti si calcola dopo, quindi un run sopravvive a
un cambio di prezzo di mercato esattamente come i diciotto viaggi.

Oggi conta **solo se l'ora è già passata**: un commute che parte alle 07:40 alle
06:00 non è ancora stato fatto.

> **Questa vista è slegata dal registro dei viaggi**, per scelta presa il
> 2026-09-15. Fino a quel giorno un viaggio chiuso e salvato dalla cartolina di
> *Trips* entrava in cima allo storico del commute riconosciuto, con un flag
> `real` e tutta la macchina che ne seguiva: il commute timbrato su `makeRecord`
> perché alla chiusura i contatori sono già azzerati, le medie che dovevano
> scavalcare i run misurati, le righe filtrate per veicolo. Funzionava, ed è
> stata tolta lo stesso: costava tre accoppiamenti fra due moduli per un
> guadagno che in demo non si vede. I due registri adesso sono due registri di
> due cose diverse, e nessuno dei due sa dell'altro.

## Il prezzo pagato: la sesta linguetta

454 px meno 22 di padding meno cinque gap fanno **67.8 px a bottone**. I gap
scendono da 7 a 5, l'icona da 13 a 12, l'etichetta da 8.5 a **8 px** con quasi
niente `letter-spacing`. Ci sta la più lunga delle sei, misurata: 66 px di
contenuto in 66 px di bottone, su tutte e sei.

**Una settima vista costerebbe una di quelle che ci sono.** Non è una previsione:
è il conto.

## Gli spazi, in px di design

Corpo del pannello 289 px utili. `cm-pick` 20 · testata 74 · fascia 34 · corpo
143, in due colonne da 212 con 8 di gap. A sinistra il **grafico, che si prende
tutta la colonna** (141, con la banda delle barre alta 102); a destra lo storico,
che scorre. L'editor sta in 306 senza scorrere.

Il grafico è l'unica cosa di questo pannello che **non ha un'altezza scritta**:
`flex:1`, e le barre si misurano a ogni ridisegno (`cm-bars.clientHeight`) invece
di fidarsi della costante. `CM_BARS_H = 55` è rimasta solo come ripiego per il
primo disegno, quando il pannello è ancora `hidden` e ogni misura dentro di lui
vale zero.

## Trappole pagate

1. **Il riquadro dei run è UNO SOLO.** La guardia che evita di riscriverlo
   quattro volte al secondo stava sull'oggetto *commute* (`c.key`): cambiando
   commute leggeva «queste righe sono già giuste» a proposito delle righe di un
   **altro** commute, e il pannello mostrava il mese precedente sotto la testata
   nuova. La guardia (`cmLogKey`) deve stare con la **scatola**, non con il dato.
2. **`cmPrice()` ha il veicolo nella chiave**, oltre ai due prezzi: `r.cur` è il
   run *come l'ha guidato questa macchina*, e uno switch che lo lasciasse con i
   litri dell'altra metterebbe una cifra a benzina sotto una batteria.
3. **I comandi a una lettera.** `E`, `V`, `P`, `R` e le frecce sono stati liberi
   finché l'app non ha avuto un campo in cui si scrive un nome: `typing(e)` ora
   li disinnesca quando il fuoco è in un `input`. Il gestore dell'UFO l'aveva già
   pagata una volta. (E `V` è diventato **il** gesto di questa vista, quindi
   deve funzionare sempre tranne lì.)
4. **Niente margine negativo** sulle righe dello storico, a differenza di
   `.ch-row`: questa lista è stretta abbastanza che otto px di sbordo ci mettono
   sotto una barra di scorrimento orizzontale.
5. **La fascia è larga 246 px**, non 432: la pastiglia e il bottone si prendono
   il resto. Sono circa **settanta caratteri**, e il settantunesimo è un
   ellissi. Le due righe sono scritte su quella misura — è già stata sforata
   due volte.
6. **Ora e giorno della prima tratta sono ancorati al viaggio aperto**, così la
   demo riconosce il tragitto sotto di sé appena la si apre. Il giorno serve
   quanto l'ora: una tratta che torna una volta a settimana, lasciata al lunedì,
   di martedì varrebbe `× 0.55` e il pannello aprirebbe dicendo che non sa cosa
   sia questo viaggio. L'etichetta segue lo spostamento e nomina il giorno su cui
   la tratta gira adesso — per questo il nome è `Torino → Milano` e non più
   `Monday · Milano`, che di martedì si contraddiceva da solo. Vale **solo fra le
   05:00 e le 21:00**: aperta alle due di notte annuncerebbe un commute che parte
   all'01:10, che è una bugia peggiore di quella che sta comprando; fuori da
   quella fascia tiene il suo lunedì 06:20, il segnale orario non prende niente e
   la fascia scende a «Looks like … 50 % sure» sulla sola distanza, che a
   quell'ora è la risposta onesta.

## Dove guardare nel codice

| | |
|---|---|
| CSS | blocco `── DAILY COMMUTE ──`, prima di *cluster typography* |
| Markup | `#pane-commute`, dopo `#pane-trips` |
| Modulo | blocco `══ DAILY COMMUTE ══` dentro `app`, prima di *the beat* |
| Costanti condivise | `HEV_F` · `PHEV_W` · `BEV_C`, sopra `TRIPS` |
| Funzioni | `cmAdd` `cmBuild` `cmEnergy` `cmCost` `cmPrice` `cmPaint` `cmBand` `cmScore` `cmDetect` `cmEpk` `cmNewTrip` · editor `csOpen` `csSync` |

Vedi anche [11-plancia-estesa.md](11-plancia-estesa.md) per le altre cinque
viste, [04-modello-costi.md](04-modello-costi.md) per le curve e
[13-phev.md](13-phev.md) per il plug-in.
