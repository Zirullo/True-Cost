# 04 · Modello: dalla velocità all'euro

La catena è tutta qui. Ogni anello è una riga di `update()` in `index.html`.

```
slider Throttle → giri → velocità → consumo L/100km → € / km → € totali
                                                    ↘ litri → livello serbatoio
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

Curva a U, la firma di ogni motore termico: pessimo a bassa velocità, ottimo intorno
ai 60–70 km/h, poi risale con l'aerodinamica.

```js
L100 = 0.000282·v² − 0.0163·v + 4.61        // litri / 100 km
```

Punti di controllo:

| Velocità | L/100 km | Nota |
|---|---|---|
| 30 km/h | 4.4 | traffico |
| 60 km/h | 4.6 | vicino all'ottimo |
| 90 km/h | 5.4 | extraurbano |
| 128 km/h | 7.1 | il caso della foto |
| 160 km/h | 9.2 | |
| 240 km/h | 16.9 | l'aerodinamica domina |

Il termine quadratico è la resistenza aerodinamica, quello lineare l'efficienza
crescente del motore che sale di carico. È la curva ereditata dal simulatore
precedente, plausibile per una compatta a benzina.

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
tripLiters += stepKm × L100 / 100
tripCost   += stepKm × eurKm
tankLiters -= stepKm × L100 / 100
```

Serbatoio: **45 L** di capacità, si parte da **27.5 L** (~61%, come la barra nella
foto). La barra `#fuelbar` è larga `livello × 100` unità SVG.

## Valori iniziali

Sono quelli della foto — 48.09 km, 5.02 L, EUR 9.55 — perché all'apertura la
schermata coincida col riferimento. Non sono reciprocamente coerenti (l'immagine
originale non lo era); dopo un RESET tutto torna coerente.

## Se vuoi ritarare

- **Vettura diversa** → cambia i tre coefficienti della curva `L100`
- **Velocità massima diversa** → cambia `VMAX` (e i numeri del quadrante si
  rigenerano da soli) e il max dello slider
- **Rapporto giri/velocità** → `K_SPEED`; ricordati che rompe la coincidenza
  2250 rpm ↔ 128 km/h con la foto
