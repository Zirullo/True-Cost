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

**In trazione**, calibrata su Jeep Avenger BEV 54 kWh — questa è arrivata intatta
dall'archivio ed è `PT.bev.per100`:

```js
kWh100 = 0.00055·v² + 0.030·v + 13.0        // kWh / 100 km
```

A 128 km/h fa 25.9 kWh/100 km: a tariffa di casa sono **0.065 €/km**, contro i
**0.153 €/km** che l'ICE segna alla stessa velocità. È quello il confronto.

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

## Quello che ancora non segue il veicolo

L'app sul display centrale tiene **un libro solo**, in litri: Cost history,
Price history e Report parlano ancora di carburante anche sotto il profilo BEV.
Seguono il veicolo solo le celle vive del viaggio in corso e il costo del fermo
in Trips. È il prossimo passo — [07-roadmap.md](07-roadmap.md).
