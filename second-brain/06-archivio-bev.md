# 06 · Il BEV

> Questa nota era un **archivio**: il pannello elettrico era stato tolto dalla UI
> e le formule aspettavano in `OLD index.html`. Dal **2026-09-06 il BEV è vivo**
> in `index.html`, come profilo di powertrain accanto all'ICE
> ([01-decisioni.md](01-decisioni.md)). Le formule qui sotto sono quelle che
> girano davvero, con le differenze dichiarate in fondo.

## Il concetto BEV

Stesso messaggio dell'ICE — quanto costa un km — con due differenze che lo rendono
più interessante, non meno:

1. Il costo si calcola sulla **tariffa domestica** impostata dall'utente
   (es. 0.25 €/kWh), non su un prezzo di mercato
2. In **frenata rigenerativa il costo per km diventa negativo**: l'auto ti sta
   restituendo energia. È l'effetto più efficace della demo

Il prezzo della colonnina fast più vicina resta visibile come confronto: rende
tangibile quanto costa ricaricare fuori casa (nel vecchio simulatore, 0.59 €/kWh
contro 0.25 domestici → +136%).

## Le formule che girano

**In trazione**, calibrata su Jeep Avenger BEV 54 kWh. La curva arrivata
dall'archivio è stata **ritarata il 2026-09-07** — vedi sotto — e adesso `PT.bev.per100` è:

```js
kWh100 = 0.00115·v² − 0.014·v + 7.0          // kWh / 100 km
```

A 128 km/h fa 24.1 kWh/100 km: a tariffa di casa sono **0.060 €/km**, contro i
**0.153 €/km** che l'ICE segna alla stessa velocità. È quello il confronto.

### Perché è stata ritarata

La curva vecchia portava una costante di **13.0 kWh/100**, che è un numero da
ciclo WLTP: **ha le accelerazioni già dentro**. Ma da quando esiste
[04 §3b](04-modello-costi.md) l'accelerazione la paga `accCost`, a parte. Il BEV
stava quindi **pagando due volte lo stesso kWh** — ed era l'unico dei quattro a
farlo, perché la costante dell'ICE sta legittimamente coprendo il crollo di
rendimento di un termico a basso carico, che un motore elettrico non ha.

Al suo posto c'è il carico stradale vero: `½·ρ·CdA·v³` per l'aria e `crr·m·g·v`
per le gomme, più 0.5 kW di ausiliari, diviso per un driveline all'87 %. **CdA
0.75, crr 0.011, 1600 kg** — un Avenger. Fit sui **70–150 km/h**, perché quello
che questa demo guida è un'autostrada.

| km/h | vecchia | nuova | fisica |
|---|---|---|---|
| 30 | 14.4 | **7.6** | 8.2 |
| 50 | 15.9 | **9.2** | 9.3 |
| 90 | 20.2 | **15.1** | 15.0 |
| 128 | 25.9 | **24.1** | 24.1 |
| 130 | 26.2 | **24.6** | 24.6 |

**In autostrada non si è mosso quasi niente** (−7 % a 128), ed è il punto: il
confronto di testa non era la cosa sbagliata. Sotto i 50 si dimezza, e quello è
il doppio conteggio che esce.

Il fotogramma d'apertura, che è **derivato** e non tipato
(`OPEN_KM × per100(128) / 100`), segue da sé: da **12.43 kWh / € 3.11** a
**11.57 kWh / € 2.89**. Verificato sul simulatore, non solo sulla carta.

**In rigenerazione** — e qui l'archivio è stato superato, vedi in fondo:

```js
recuperato = min( ½·m·(v₀² − v²) · η ,  P_max · dt )   // m 1600 kg, η 0.65, P_max 45 kW
stepU      = consumo_in_trazione − recuperato       // può essere negativo
eurKm      = (stepU / stepKm) × prezzo_della_carica  // negativo → verde
```

**Fermo e pronto**: `PT.bev.idleRate = 0.4` kW di ausiliari. Non zero — un
elettrico acceso e fermo consuma — ma un ordine di grandezza sotto i 0.7 L/h
dell'ICE, e Trips lo dice in euro.

**Stato di carica**: batteria 54 kWh, partenza 42.1 kWh (78%). Risale in
rigenerazione, perché `stepU` negativo rimette dentro invece di togliere.

## Com'è stato reintrodotto

Il layout della foto si mappa quasi 1:1, e così è stato fatto — **stesse caselle,
stesso quadrante, etichette diverse**:

| Elemento ICE | Equivalente BEV |
|---|---|
| Giri motore (`rpmTxt`) | potenza kW, **con segno**: negativa in rigenerazione |
| Marcia (`gearTxt`) | `D` — e `P` a vettura non pronta |
| CONSUMED · litri | kWh consumati |
| Last Refuel Price | Last Charge Price, €/kWh |
| Local Price | il totem della colonnina, 0.50–0.86 €/kWh |
| Fuel Level E→F | Charge 0→100 |
| L / 100 km | kWh / 100 km |

Il tachimetro **non cambia**: i km/h sono km/h. Il numero grande €/km resta lo
stesso ed è **verde quando è negativo**.

## Quello che è cambiato rispetto all'archivio

**La rigenerazione non è più un fattore, è fisica.** L'archivio la modellava con
tre efficienze (`off / low / high`) scelte a mano e un fattore moltiplicativo sul
consumo in marcia. Si è scelto invece il **freno automatico**
([08-domande-aperte.md](08-domande-aperte.md)): nessun selettore da spiegare, e
l'energia che torna è quella cinetica davvero persa nel passo,

```js
recuperato = min( ½·m·(v₀² − v²) · η ,  P_max·dt )     // m 1600 kg, η 0.65, P_max 45 kW
```

Il tetto dei 45 kW è quello che rende la cosa onesta: una frenata forte chiede
molta più potenza di quanta il motore ne assorba, e il resto se lo prendono i
freni ad attrito. Non si ripaga tutto, e non deve.

**Il prezzo di partenza è la tariffa di casa.** La carica in batteria viene dal
muro, a `state.homeTariff` (0.25 €/kWh, regolabile **dentro il pannello
*Chargers*** sul display). È il
contrasto che fa la demo: ogni cartellone lungo la strada ne chiede almeno il
doppio. Attaccarsi a una colonnina riscrive quel prezzo — e da quel momento
`chargedOut` impedisce allo slider di riscriverlo di nuovo, perché quei kWh li
hai già pagati a quel prezzo lì.

**Ricarica solo alla colonnina.** Niente gesto «attacca a casa»: si carica dove ci
si ferma, come si fa il pieno, con lo stesso tasto «R».

## La tariffa sta dove si decide

Lo slider vive nel pannello che sotto il termico si chiama *Pumps* e sotto
l'elettrico **Chargers**, in una riga propria sotto `SEARCH`, e compare solo in
BEV. Accanto al valore c'è la frase che è tutto il progetto in una riga:
«cheapest in range 0.48 · 1.9× quello che paghi a casa». Metterlo qui invece che
sulla barra sopra la scena vuol dire metterlo **accanto ai prezzi contro cui
va confrontato**, e togliere dalla regia una cosa che non è regia.

## La velocità attraversa lo switch

È l'unica cosa che passa la porta fra i due veicoli. I soldi no — viaggio,
serbatoio o batteria, ultimo prezzo pagato restano di ciascuno — ma i km/h sì:
a 128 premi BEV e sei ancora a 128, e gli €/km passano da **0.15 a 0.06** sotto
i tuoi occhi senza toccare nient'altro. È il confronto più forte che la demo
possa fare, e prima non era possibile perché si ripartiva da fermi.

Perché funzioni, l'auto in cui atterri deve saper *reggere* quella velocità
subito: entrando nell'ICE si sceglie la marcia il cui rapporto tiene i giri in
banda (invece di ritrovarsi in prima a 128), entrando nel BEV si parte dalla
potenza che quella andatura costa davvero, così i kW si assestano invece di
salire da zero.

## Il log, il Report e i prezzi

Ogni viaggio porta `t.veh`, e le viste mostrano solo la metà della vettura in
uso. I diciotto viaggi hanno un gemello elettrico sulle stesse tratte, col
consumo derivato da `PT.bev.per100` sulla velocità media — e l'ordine del mese
si rovescia, perché in autostrada l'elettrico è caro e in città è economico.
Stesso mese, stessi 1194 km: **€ 151.61 contro € 131.33**.

### `BEV_C`: la media non è l'andatura

Con la costante da WLTP tolta, leggere la curva **solo** sulla velocità media è
diventato sbagliato in modo visibile, e va detto perché.

`per100` è una curva di crociera, `kmh` è una **media**. Un viaggio non si guida
alla propria media: l'ora Torino–Milano è 120 in autostrada e 30 sulle rampe, e
siccome il consumo è **convesso** nella velocità, leggere la media **sotto-stima**
il viaggio. È la disuguaglianza di Jensen, in autostrada vale circa un quarto, e
la vecchia costante di 13.0 la stava nascondendo.

Quindi lo stesso mestiere di `HEV_F`, nell'altro verso: **un fattore per
carattere**, e `char` già sa quale. Non sono inventati nemmeno questi — ognuno è
il rapporto fra un profilo di velocità plausibile per quel carattere, fatto
passare **dentro questa curva** e addebitato delle proprie fermate, e quello che
la curva dice alla media che quel profilo produce:

| | profilo | kWh/100 reali | media | `BEV_C` |
|---|---|---|---|---|
| **road** | 120 con rampe a 31, un rallentamento ogni 12 km | 19.8 | 81.7 | **1.46** |
| **mix** | 95 / 62 / 30, una fermata sì e una no | 14.8 | 59.6 | **1.45** |
| **city** | 46 e 24, due fermate e mezza al chilometro | 12.4 | 33.5 | **1.60** |

Sono le cifre che un Avenger restituisce in quelle tre giornate. Il totale del
mese quasi non si muove — **218.6 → 217.7 kWh** — ma la **distribuzione** sì:
l'autostrada rincara, la città cala del 14 %. Che è la forma giusta, e quella che
la curva vecchia appiattiva.

### Una semplificazione dichiarata

È la ragione per cui questi numeri sono **tipati qui** invece che guidati dal vivo.
Le fermate qui sopra sono addebitate al recupero di un'auto vera, **65 % della
cinetica**. Il modello vivo rigenera **solo col pedale del freno premuto**, e il
pedale qui dentro toglie **20 km/h al secondo**: così forte che il tetto dei 45 kW
manda in calore la maggior parte di una frenata.

**In autostrada i due coincidono**, ed è lì che questa demo si guida. In città il
log è il più onesto dei due, e chiudere quel divario vuol dire dare all'auto viva
la **rigenerazione in rilascio** — vedi [08-domande-aperte.md](08-domande-aperte.md).

La serie storica dei prezzi elettrici è **derivata** da quella dei carburanti,
smorzata al 55 %, e la vista lo dichiara (`EST. MARKET`). Dettagli e ragioni in
[01-decisioni.md](01-decisioni.md).

## Il debito che resta

`tripLiters`, `tankLiters`, `instantL100` e `t.l100` contengono kWh sotto il
profilo elettrico. Lo forzerà il PHEV, che brucia entrambi e non può fingere
che sia un contatore solo.
