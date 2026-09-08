# 04 · Modello: dalla velocità all'euro

La catena è tutta qui. Ogni anello è una riga di `update()` in `index.html`.

```
slider Throttle → giri → velocità → consumo /100km → € / km → € totali
                                                   ↘ unità → livello serbatoio
```

## 1 · Giri motore

Il valore dello slider è un **target**; i giri reali lo inseguono con smorzamento e
una leggera oscillazione, così l'ago non "scatta":

```js
drift        = sin(driftAngle) * 12          // respiro del motore, ±12 rpm
currentRpm  += (targetRpm + drift - currentRpm) * 0.1
```

Range slider: **800 – 3520 rpm** (800 = minimo, vettura ferma).

## 2 · Velocità

```js
v = clamp( (rpm - 800) × 0.08828 , 0 , 240 )   // km/h
```

Il coefficiente **non è arbitrario**: è tarato perché 2250 rpm dia esattamente
128 km/h, i valori della foto di riferimento. Mappa lineare (in realtà un cambio ha
rapporti discreti — semplificazione consapevole, vedi
[08-domande-aperte.md](08-domande-aperte.md)).

A 800 rpm la vettura è **ferma**: consumo istantaneo azzerato (il consumo al minimo
non è modellato) e lo storico viaggi torna in evidenza.

## 3 · Consumo istantaneo

Curva a U, la firma di ogni motore termico. Il minimo sta a **28.9 km/h**
(`0.0163 / (2·0.000282)`) e la curva resta quasi piatta fino a 50: quello che si
vede in guida è la risalita aerodinamica dopo i 70.

```js
L100 = 0.000282·v² − 0.0163·v + 4.61        // litri / 100 km
```

Punti di controllo:

| Velocità | L/100 km | Nota |
|---|---|---|
| 30 km/h | 4.4 | traffico |
| 60 km/h | 4.6 | |
| 90 km/h | 5.4 | extraurbano |
| 128 km/h | 7.1 | il caso della foto |
| 160 km/h | 9.2 | |
| 240 km/h | 16.9 | l'aerodinamica domina |

Il termine quadratico è la resistenza aerodinamica, quello lineare l'efficienza
crescente del motore che sale di carico. È la curva ereditata dal simulatore
precedente, plausibile per una compatta a benzina.

### Le altre tre curve

```js
hev:  v => ice(v) · (0.90 − 0.10·e^(−v/45))           // L / 100 km, solo motore
phev: v => hev(v) · 1.06                              // L / 100 km, a pacco scarico
bev:  v => 0.00115·v² − 0.014·v + 7.0                 // kWh / 100 km
```

**Sono una catena, non quattro copie**: l'ibrido legge `ice`, il plug-in legge
`hev`. Ritarare il benzina sposta tutta la linea. Il fattore dell'ibrido è il
guadagno Atkinson, più forte a basso carico: 0.85 a 30, 0.894 a 128 — e dentro non
c'è nessun recupero, quello lo guadagna il tampone in `update()`
([12](12-full-hybrid.md)). L'1.06 del plug-in è il pacco che pesa anche da scarico
([13](13-phev.md)).

Il BEV è l'unico che sta per conto suo, e l'unico su **carico stradale fisico**
invece che sull'eredità della foto: perché, e con quali coefficienti, in
[06-archivio-bev.md](06-archivio-bev.md).

### 3b · L'accelerazione si paga

`per100` è una curva di **crociera**: dice cosa costa *tenere* una velocità, non
*raggiungerla*. Finché nessuno restituiva energia non si vedeva; con la frenata
rigenerativa l'auto veniva accreditata di energia cinetica mai addebitata, cioè
carburante gratis. Da qui, su tutti e quattro i profili:

```js
if (v > v0) stepU += ½·mass·((v/3.6)² − (v0/3.6)²)/3.6e6 · accCost
```

`accCost` è il cambio fra un kWh alla ruota e l'unità del pieno: **0.382 L/kWh**
ICE, **0.323** i due ibridi, **1.136 kWh/kWh** BEV. A velocità costante il termine è
esattamente zero, quindi la crociera di riferimento legge ancora 7.14 L/100 km.
Ma adesso **in città si spende più che in autostrada**, come dicevano già i
diciotto viaggi seminati e come il modello vivo negava.

### 3c · I readout si assestano

L'energia del passo cade tutta nel frame in cui la chiedi, e il **libro** la legge
grezza. Il **quadrante** la mostra smorzata, come ogni consumo istantaneo vero:

```js
instantPer100 += (target − instantPer100) · min(1, dt · INST_DAMP)  // 1.6, τ ≈ 0.6 s
instantEurKm  += (target − instantEurKm)  · min(1, dt · INST_DAMP)
```

Nessun totale cambia: `instantPer100` e `instantEurKm` sono letti solo dai tre posti
che li stampano, e le barre di costo campionano `tripKm` / `tripCost`. Se la salita
in accelerazione va resa più o meno progressiva, **`INST_DAMP` è l'unica manopola**.

## 4 · Costo per chilometro

```js
eurKm = (L100 / 100) × refuelPrice
```

Si usa il **prezzo dell'ultimo pieno**, non quello locale: è il carburante che stai
bruciando davvero. Vedi la decisione in [01-decisioni.md](01-decisioni.md).

Verifica sui numeri della foto: `7.14 / 100 × 1.91 = 0.136` → mostrato **0.14 €/km**.
Identico al riferimento.

## 5 · Accumulo del viaggio

Per ogni frame (~16 ms), moltiplicato per il Time Warp:

```js
stepKm      = (v / 3600) × 0.016 × timeMultiplier
tripKm     += stepKm
tripUnits  += stepKm × L100 / 100
tripCost   += stepKm × eurKm
tankUnits  -= stepKm × L100 / 100
```

Serbatoio: **45 L** di capacità, si parte da **27.5 L** (~61%, come la barra nella
foto). La barra `#fuelbar` è larga `livello × 100` unità SVG.

## Valori iniziali

Sono quelli della foto — 48.09 km, 5.02 L, EUR 9.55 — perché all'apertura la
schermata coincida col riferimento. Non sono reciprocamente coerenti (l'immagine
originale non lo era); dopo un RESET tutto torna coerente.

**E sono lo stesso viaggio per tutte e quattro le motorizzazioni.** `OPEN_KM` è
condiviso; l'ICE porta `openUnits` / `openCost` tipati (la foto), gli altri tre li
derivano dalla propria curva a `OPEN_KMH`. È l'unica copia di questa tabella:

| | distanza | consumo | totale |
|---|---|---|---|
| ICE | 48.09 km | 5.02 L | € 9.55 |
| HEV | 48.09 km | 3.07 L | € 6.30 |
| PHEV | 48.09 km | 0.68 L + 11.40 kWh | € 4.25 |
| BEV | 48.09 km | 11.57 kWh | € 2.89 |

Stessa strada, quattro conti. Il PHEV apre **a pacco scarico** e il perché sta in
[13-phev.md](13-phev.md); l'ibrido in [12-full-hybrid.md](12-full-hybrid.md).

## Se vuoi ritarare

- **Vettura diversa** → cambia i tre coefficienti della curva `L100`
- **Velocità massima diversa** → cambia `VMAX` (e i numeri del quadrante si
  rigenerano da soli) e il max dello slider
- **Rapporto giri/velocità** → `K_SPEED`; ricordati che rompe la coincidenza
  2250 rpm ↔ 128 km/h con la foto
