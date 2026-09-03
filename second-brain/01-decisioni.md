# 01 · Log delle decisioni

Ordine cronologico inverso (la più recente in alto). Ogni voce: **cosa**, **perché**,
**cosa comporta**.

---

## 2026-09-03 · La visuale si allarga fino al display centrale, ricostruito a mano

**Scelta**: per ospitare un'app nel display centrale la scena passa da **1147** a
**1720** unità di larghezza (altezza invariata, 791). La foto resta **intatta e
ancorata a sinistra**, pixel per pixel nella posizione di oggi; tutto ciò che sta a
destra di x 1147 — cruscotto, trim in alluminio, bocchetta, pannello in carbonio,
cornice e schermo — è **disegnato in CSS/SVG**, non fotografico. Il display centrale
è inclinato di **9°** verso il guidatore. Prima di integrare: mockup statico
`mockup-plancia-estesa.html` (fuori dal repo) per giudicarlo a occhio.

**Perché**: la foto di riferimento è 1147 × 641 e a destra del cluster **non esistono
pixel**. Le alternative erano cambiare foto di base (il cockpit Jeep: cluster piccolo,
inclinato, molto meno leggibile — cioè cambiare proprio la visuale che vogliamo
tenere) o ricomporre un'altra foto della stessa plancia, che non abbiamo. La
ricostruzione vettoriale è anche l'unica che dà uno **schermo vivo**: l'app dev'essere
leggibile e animata, non un'immagine incollata. I 9° sono il compromesso fra la
profondità della foto Jeep e la leggibilità dei numeri a schermo.

**Come si ottiene la continuità dei colori**: l'ultima colonna della foto (20 px)
viene stirata a destra e sfocata. Continua da sola le bande orizzontali — colline,
guard-rail, cruscotto, carbonio — con gli stessi identici colori, e su quella base si
disegna la geometria.

**Comporta**: (a) la strada procedurale va allargata a 1720 e il punto di fuga
**non** si sposta (resta quello della foto, x ≈ 573); (b) l'array `edge` della
maschera del parabrezza va prolungato oltre x 1147 fino al nuovo bordo;
(c) `--sw`, `max-width` e il `padding-top` che appoggia il cockpit al fondo pagina
sono tutti tarati su 1147 e vanno rifatti su 1720; (d) su schermi stretti tutto
scala insieme, quindi **il cluster diventa più piccolo di oggi a parità di finestra**:
è il prezzo dell'inquadratura più larga.

---

## 2026-09-03 · Una sola storia, e il push come unico canale di pubblicazione

**Scelta**: il repo locale e quello su GitHub, fino a oggi separati, diventano una
storia sola. Da qui in avanti si pubblica **solo** con `git push`: niente più upload
di `index.html` dalla pagina web di GitHub. Questo stato è marcato **`v1.0`**.

**Perché**: erano due progetti che si credevano lo stesso. Su GitHub un repo
costruito trascinando il file nel browser — quindici salvataggi tutti chiamati
"Add files via upload", solo `index.html` e la foto. Sul PC un `git init` separato,
con i messaggi veri e tutto il `second-brain/`. Nessun collegamento fra i due
(`git remote` era vuoto): le copie combaciavano per disciplina manuale, non per
meccanismo, e bastava una dimenticanza per farle divergere in silenzio — con la
versione online che è quella che il pubblico vede.

Al momento dell'unione i due `index.html` avevano contenuto **identico** (differivano
solo per i fine riga), quindi il merge non ha perso nulla. La storia degli upload da
browser resta sotto, innestata nella nostra.

**Comporta**: un upload dal web adesso crea un commit che il PC non ha, e il `push`
successivo viene rifiutato — con conflitto da ricucire a mano dentro un file da
oltre centomila caratteri. Il canale è uno solo, alla volta. Il link pubblico non
cambia mai: GitHub Pages serve sempre l'`index.html` in cima a `main`, quindi ogni
push aggiorna la demo allo stesso indirizzo. Dettagli operativi in
[10-pubblicazione.md](10-pubblicazione.md).

**Comporta anche**: il repo è **pubblico**, quindi da oggi queste note sono leggibili
da chiunque abbia il link. Vedi la voce corrispondente in
[08-domande-aperte.md](08-domande-aperte.md).

---

## 2026-09-02 · I pedali comandano la velocità reale, e il rilascio è un cruise

**Scelta**: tre stati e solo tre. **↑ tenuto** accelera finché è tenuto, **↓ tenuto**
rallenta finché è tenuto, **niente premuto mantiene** la velocità raggiunta. Un pedale
premuto non rampa più la velocità *richiesta*: chiede semplicemente `PEDAL_LEAD` = 8 km/h
oltre (o sotto) quella **reale**, e al rilascio `setPedal()` aggancia `targetSpeed` a
`currentSpeed`.

**Perché**: i pedali spostavano `targetSpeed` di +55 / −95 km/h al secondo, mentre la
vettura ne può guadagnare al massimo ~13. Una pressione di un secondo lasciava quindi il
target **40 km/h davanti** al veicolo, che continuava ad accelerare da solo per parecchi
secondi dopo che il tasto era già su. Il comando non era né un pedale né un cruise: era
una richiesta che si scaricava con ritardo, e in demo si finiva per inseguire il numero
invece di guidarlo.

Con il divario fisso a 8 km/h il divario non si chiude mai mentre tieni premuto — quindi
si accelera per tutto il tempo, alla rampa vera limitata da `ACC_MAX` e dal cambio — e si
azzera nell'istante del rilascio.

**Il freno ora frena**: la decelerazione con il pedale giù è `BRAKE_MAX` 20 + 5% della
velocità (~26 km/h/s a 130), circa il doppio del rilascio motore `DEC_COAST`. Prima
l'unico modo di rallentare era il freno motore, e fermarsi richiedeva un tempo
implausibile. Il ramo di puro rilascio resta, e si raggiunge quando è **lo slider** a
chiedere meno.

**Comporta**: con entrambi premuti **vince il freno** (valutato per secondo in
`applyPedals()`). Lo slider continua a funzionare come prima ed è l'unico modo di
ottenere una decelerazione in solo rilascio motore. `PEDAL_UP` e `PEDAL_DOWN` non
esistono più.

---

## 2026-09-02 · L'alone di richiamo dei pedali diventa rosso

**Scelta**: l'anello che pulsa attorno ai pedali finché non se ne tiene premuto uno
(`@keyframes pedalCall`) passa da 4 px di azzurro ad alfa **.18** a 7 px di **rosso
`#ff5f4d`** ad alfa **.55**, più un bagliore esterno di 30 px, il bordo che pulsa
insieme, e un anello sottile che resta anche nella fase bassa.

**Perché**: su una plancia scura l'azzurro al 18% non si vedeva. La demo restava ferma
ad aspettare un input che nessuno capiva di dover dare — esattamente il rischio che i
tre livelli di discoverability dovevano coprire. Il minimo dell'animazione non torna
più a zero perché un alone che si annulla del tutto per un secondo lascia proprio la
finestra in cui chi guarda non vede alcun invito.

**Perché quel rosso**: `#ff5f4d` è già il colore del pedale del freno premuto.
Introdurne un quarto per un solo elemento avrebbe sporcato la palette.

**Comporta**: mentre l'animazione gira, `border-color` è controllato dai keyframe,
quindi l'`:hover` non tinge più il bordo **prima** del primo utilizzo. Dopo torna
normale.

---

## 2026-09-02 · Un UFO nel cielo, come easter egg

**Scelta**: in fondo a sinistra, basso sulle colline, ogni **5 minuti** compare un
piccolo UFO che scende, apre un raggio, **tira su una mucca in silhouette** e risale
(`ufo()` in `index.html`). Passaggio di 10,5 s; il tasto **U** lo chiama a comando.

**Perché**: la demo si guarda a lungo e a velocità costante. Una sorpresa rara
premia chi resta a guardare senza rubare un solo pixel al cluster, che è quello
che deve leggere il pubblico. Raro e piccolo è la condizione perché resti una
sorpresa e non diventi arredamento.

**Comporta**: è l'unica cosa nella scena che **non** scorre con `travel` e non ha
parallasse — di proposito. Vola basso perché la mucca deve poter salire **da dietro
la boscaglia**, e questo lo fa passare dietro agli alberi di sinistra ogni tanto. Se un giorno desse fastidio in una presentazione,
basta alzare `UFO_EVERY`. Dettagli in [09-strada.md](09-strada.md).

---

## 2026-09-02 · Abitacolo = la foto, non una ricostruzione

**Scelta**: l'abitacolo (plancia, cornice, volante) è la foto di riferimento usata
come sfondo; sopra ci va il cluster live.

**Perché**: massima fedeltà visiva a costo quasi zero. Una ricostruzione in CSS/SVG
sarebbe stata più pulita ma meno fotorealistica, e il valore della demo sta proprio
nel sembrare una vettura vera.

**Comporta**: il progetto **dipende dal file** `True-Cost project picture.jpg`.
Se l'immagine cambia, vanno ricalcolate la maschera del parabrezza e le coordinate
del cluster ([03-mappa-design.md](03-mappa-design.md)).

---

## 2026-09-02 · Il parabrezza è un layer vuoto sotto la foto

**Scelta**: l'area del vetro viene **ritagliata dalla foto** con una mask SVG; sotto
c'è un `<div id="windshield">` vuoto.

**Perché**: l'effetto velocità deve andare *dentro* il parabrezza, non sopra la
plancia. Ritagliando il vetro, qualsiasi cosa metteremo nel div sarà automaticamente
mascherata dalla forma corretta del vetro senza altro lavoro.

**Comporta**: il bordo del ritaglio è un poligono tracciato a mano
(array `edge` in `index.html`). È l'unico punto da ritoccare se la giunzione tra
vetro e plancia stona.

---

## 2026-09-02 · Cluster in SVG, non più su canvas

**Scelta**: il quadro strumenti è un unico `<svg viewBox="0 0 1147 641">`; il canvas
2D della versione precedente è stato abbandonato.

**Perché**: testo nitido a ogni risoluzione, gradienti e glow più vicini alla foto,
e soprattutto **le coordinate SVG coincidono con i pixel della foto** — posizionare
un elemento significa leggerne la posizione sull'immagine.

**Comporta**: niente `requestAnimationFrame` per ridisegnare tutto; il loop aggiorna
solo i nodi di testo e le classi delle tacche. Più semplice ed economico.

---

## 2026-09-02 · Solo ICE: pannello BEV e switcher rimossi

**Scelta**: la demo mostra solo la vettura a benzina, come nella foto.

**Perché**: la foto di riferimento è ICE; mantenere due pannelli avrebbe raddoppiato
il lavoro di restyling e diluito il messaggio.

**Comporta**: la logica BEV (regen, SoC, kWh) **non è persa**, è documentata in
[06-archivio-bev.md](06-archivio-bev.md) e vive ancora in `OLD index.html`.

---

## 2026-09-02 · Calibrazione sui numeri della foto

**Scelta**: la mappa giri→velocità è tarata perché **2250 rpm = 128 km/h**, esattamente
come nell'immagine di riferimento; i valori iniziali del viaggio sono quelli della
foto (48.09 km · 5.02 L · EUR 9.55).

**Perché**: all'apertura la schermata è indistinguibile dal riferimento — conta molto
in un pitch. Poi tutto evolve realisticamente.

**Comporta**: i numeri iniziali sono *scenografici* e non perfettamente coerenti fra
loro (48.09 km × 0.14 €/km ≠ 9.55 €). È un compromesso voluto, ereditato dalla foto.
Il RESET azzera tutto e da lì i numeri sono coerenti.

---

## 2026-09-02 · Il costo si calcola sul prezzo dell'ultimo pieno

**Scelta**: `€/km = consumo × Last Refuel Price`, non × prezzo locale.

**Perché**: è il carburante che hai **effettivamente** in serbatoio; è quello che hai
pagato. Il prezzo locale serve come confronto ("quanto costerebbe fare il pieno qui").

**Comporta**: due prezzi distinti a schermo, ed è una delle idee più forti del
progetto — vale la pena spiegarla nei pitch.

---

## 2026-09-02 · Strada procedurale nel parabrezza (canvas)

**Scelta**: l'effetto velocità è una **strada disegnata a runtime** su un `<canvas>`
dentro `#windshield` — non un video, non uno sfondo sfocato. Autostrada a 3 corsie
di giorno, guida a destra, con guard-rail, curve lente e traffico.

**Perché**: nessun asset esterno (il file resta apribile con doppio click), il ritmo è
legato **metricamente** alla velocità (`travel += km/h ÷ 3.6 × dt`) invece che a un
`playbackRate` approssimato, e a vettura ferma si ferma davvero. Un video reale
sarebbe più fotorealistico ma pesante, con loop percepibile e velocità non calibrata.

**Comporta**: la resa è stilizzata, non fotografica. Se un giorno si volesse il
fotorealismo, il modulo è isolato e sostituibile senza toccare il resto.

---

## 2026-09-02 · Camera nella corsia centrale

**Scelta**: il punto di vista sta nella **corsia centrale**, non nella corsia di destra
come imporrebbe il codice della strada.

**Perché**: il punto di fuga della foto è a x 575 su 1147, cioè quasi al centro. Con la
camera in corsia centrale la strada resta simmetrica attorno a quel punto e si
incastra con la prospettiva dell'abitacolo; dalla corsia destra la carreggiata
sbanderebbe tutta a sinistra.

**Comporta**: una piccola infedeltà al codice della strada, invisibile a chi guarda.
Il traffico compensa: si sorpassa a destra (mezzi lenti) e si viene sorpassati a
sinistra quando si va piano.

---

## 2026-09-02 · Pedali sul bordo del cockpit

**Scelta**: due pulsanti **Accelerate / Brake** in leggera sovrapposizione col bordo
basso della scena (`#pedals`, dentro un wrapper `#shell` che avvolge solo `#stage`),
che si comandano **tenendoli premuti** e sono l'esatto equivalente di **freccia su /
freccia giù**. Non muovono la velocità direttamente: fanno una rampa su
`state.targetSpeed`, cioè sulla stessa richiesta che fa lo slider — la fisica di
accelerazione e il cambio restano gli unici a decidere come ci si arriva.

**Perché**: lo slider Speed è preciso ma non si "guida", e chi vede la demo per la
prima volta non capisce di poter cambiare andatura. Un pedale tenuto premuto è
l'unico gesto che sembra guidare. La sovrapposizione col cockpit li lega alla vettura
invece che al pannello di debug.

**Discoverability** (il vero rischio: che l'utente non capisca di poter accelerare):
tre livelli ridondanti — (1) i pedali *respirano* con un alone **rosso** finché non ne
viene tenuto premuto uno la prima volta (`#pedals.untouched`), (2) sopra c'è la
scritta "Hold a pedal — or the ↑ ↓ arrow keys" che svanisce al primo uso, (3) il
popup di benvenuto lo dice e il suo bottone di chiusura è "Got it — hold ↑ to drive".

**Comporta**: le frecce su/giù non scrollano più la pagina (`preventDefault` globale)
e non pilotano più lo slider quando ha il focus; lo slider ora **segue** i pedali
invece di essere la sola sorgente. Un pedale non può restare incastrato: si rilascia
su `pointerup/cancel/leave`, su `blur` del bottone e su `blur` della finestra.

---

## 2026-09-02 · Stazioni di servizio sulla strada, e Local Price che le conta

**Scelta**: la riga *Local Price* del cluster non mostra più la media nuda dei
distributori: mostra il **prezzo della prossima stazione** e **quanto manca**. Le
stazioni stanno ogni **2 km** sulla destra della carreggiata, con un cartellone su
cui il prezzo è scritto e leggibile mentre ti avvicini.

**Perché**: il prezzo medio è un numero che non succede a nessuno. Un prezzo con un
posto e una distanza è una decisione: *fermarmi qui o no*. È esattamente la domanda
che True Cost vuole mettere in testa a chi guida, ed è gratis da raccontare perché
la strada c'era già.

**I tre numeri e perché quelli**:
- **2 km** invece dei 5 chiesti (e dei ~5 reali in autostrada): a 130 km/h sono ~55
  secondi. A 5 km una stazione passa ogni 2 minuti e mezzo ed è visibile 10 secondi —
  in una demo dal vivo non se ne vedrebbe mai una. La costante è `ST_EVERY`, una riga.
- **± 0.15 €** attorno alla **media live** MIMIT, non attorno a 1.85 fisso: 1.85 resta
  il centro solo quando l'API non risponde. Il dato reale è metà del pitch, buttarlo
  per una simulazione sarebbe stato un cattivo affare.
- Il prezzo dipende dall'**indice della stazione** via `hash()`, non dal tempo: deciso
  una volta, non tremola mentre ti avvicini, diverso a ogni stazione.

**Superare una stazione non fa niente**: nessun rifornimento automatico, il serbatoio
e *Last Refuel Price* non cambiano. Il rifornimento è una funzione a sé, non un
effetto collaterale del passaggio ([07-roadmap.md](07-roadmap.md)).

**Comporta**: il conteggio parte da `travel` della strada, non da `tripKm`, quindi
**RESET non teletrasporta la stazione** — azzera il viaggio, non la posizione nel
mondo. E il cartellone è basso (1.4–4.4 m) contro ogni realismo: vedi
[09-strada.md](09-strada.md), *Perché il cartellone è basso*.

---

## 2026-09-02 · Un cavalcavia ogni km, e i cartelloni diventano due

**Scelta**: un **cavalcavia ogni 1000 m a partire da 500 m**, con rampe di raccordo
su rilevato erboso ai due lati; e i cartelloni sulla sinistra alternano **il marchio
Stellantis** e **il manifesto True Cost**, quindi ognuno ogni 600 m.

**Perché il cavalcavia**: nella scena non c'era niente che passasse **sopra** la
telecamera. Alberi, guard-rail e cartelli spazzano di lato; un impalcato scende
nell'inquadratura, si gonfia ed esce dal tetto, ed è l'unica cosa che dichiara che
la strada ha una terza dimensione. Un chilometro sono ~28 secondi a 130 km/h:
abbastanza raro da restare un evento, abbastanza frequente da vederne uno in demo.

**Perché le rampe**: la prima versione era la sola campata, e sembrava un portale
autostradale. Il rilevato che scende nei campi è ciò che dice *strada che scavalca
la strada*. Prima provato in cemento a tutta altezza: murava l'orizzonte da un bordo
all'altro. La scarpata **in erba** con la trave sopra costa uguale e si fonde con la
banchina — vedi [09-strada.md](09-strada.md), *Le rampe*.

**Perché due grafiche**: il cartellone unico con la sola scritta `TRUE COST` sotto il
marchio era pubblicitariamente muto. Il manifesto porta la promessa
(*transparency that count\$*) e la famiglia dei brand, e alternandolo col marchio
nudo nessuno dei due diventa carta da parati. Entrambe le grafiche sono **canvas
fuori schermo disegnati una volta**: la strada non paga niente per frame e le scritte
non tremolano mentre il cartellone scala.

**Comporta**: cartelli verdi (ogni 500 m) e cartelloni (ogni 300 m) cadevano dentro
l'impalcato — ora lo slot che finisce su un cavalcavia viene saltato, come già si
faceva per le stazioni. E gli alberi lasciano 24 m liberi attorno a ogni ponte.

---

## 2026-09-04 · L'app sul display: due viste, e la mappa è un mondo a sé

**Scelta**: il display centrale mostra l'app True Cost con due schede —
**Cost history** (aperta all'avvio) e **Local pump map**. Per fare posto, la fila
di tasti clima finti sotto lo schermo è stata **eliminata** e quel nastro è
diventato vetro: +34 % di altezza utile senza toccare l'inquadratura.

**Perché aprire su Cost history**: è la tesi del progetto. Chi guarda la demo deve
vedere per primo *quanto sto spendendo*, non un navigatore — di navigatori ne ha
già visti. La mappa è la conseguenza («e allora dove conviene fermarsi?»), quindi
sta a un tocco di distanza, non davanti.

**Perché via i tasti clima**: erano decorazione, e rubavano il 20 % dell'unica
superficie che in questa demo deve essere leggibile a distanza in una sala. Un
display grande e nudo è anche più credibile di uno piccolo con finti comandi:
è dove sta andando l'industria.

**Perché le stazioni della mappa NON sono quelle della strada**: le due cose
rispondono a due domande diverse. La strada racconta *il viaggio* (una stazione
ogni 2 km, prezzo sul cartellone, ci si può fermare a fare il pieno); la mappa
racconta *il mercato intorno alla vettura* — sette insegne sparse a 360°, dietro e
davanti, che sulla carreggiata non si incontrerebbero mai. Legarle avrebbe voluto
dire o una mappa con tutti i pin in fila indiana sulla stessa retta, o riscrivere
il mondo di `road`. Il legame che conta lo abbiamo tenuto: **entrambe partono da
`state.marketPrice`**, il prezzo vero preso dall'API, quindi non si contraddicono
sul livello dei prezzi.

**Comporta**: se un giorno servisse la coerenza pin-per-pin («la mappa dice 1.78 e
fra 2 km sul cartellone leggo 1.78») va fatto generare a `road` anche le stazioni
fuori percorso, non il contrario. E il colore è **una scala sola**
(verde → ambra → rosso) calcolata sull'intervallo dei prezzi a schermo: se un
giorno i prezzi si appiattissero, verde e rosso resterebbero comunque agli
estremi — è una **classifica**, non una soglia assoluta.
