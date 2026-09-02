# 03 · Mappa del design

Tutte le coordinate sono in **pixel dell'immagine di riferimento**
(`True-Cost project picture.jpg`, 1147 × 641) e coincidono con le unità del
`viewBox` dell'SVG. Origine in alto a sinistra.

## Schermo del cluster

Sagoma (path `#screenClip`), l'"occhio" del display:

```
M214,300 C216,262 250,232 300,214 C360,192 430,172 520,166
L800,163 C880,166 940,180 990,206 C1035,230 1050,265 1048,296
C1046,330 1030,360 990,382 C940,410 870,428 780,436
L560,441 C460,437 380,424 310,400 C250,378 216,340 214,300 Z
```

È leggermente **interna** all'apertura reale della cornice: il nero del bezel nella
foto copre l'eventuale disallineamento. Non allargarla senza verificare.

## Elementi

| Elemento | Posizione | Note |
|---|---|---|
| Barra STELLANTIS | rect 430,164 · 326×42 | testo centrato 596,194 · size 23 · spacing 4.2 |
| Cartello limite 130 | cerchio 772,203 r 15.5 | sfiora il bordo destro della barra, come nella foto |
| "TRUE COST" | 603,241 · size 19 | |
| "?" | cerchio 740,234 r 11 | cliccabile → popup info |
| **Quadrante** | centro **345,305** | anello esterno r 106, disco r 99, mozzo r 56 |
| Tacche | r 83→97 (major) · 89→97 (minor) | major ogni 20, minor ogni 10 |
| Numeri 0–240 | r 71 · size 15 | 0 in basso a sinistra, 240 in basso a destra |
| Velocità | 345,316 · size 52 | "km/h" a 345,341 |
| Giri | 347,404 · size 32 | "RPM" a 347,422 |
| Indicatore laterale | ~238–255, 288–396 | decorativo (pallino rosso + arco + icona) |
| LIVE CONSUMPTION | etichetta 533,268 | valore 533,316 size 56 · "EURO / KM" 533,338 |
| DISTANCE | etichetta 675,269 | valore 675,295 size 27 |
| CONSUMED | etichetta 675,321 | valore 675,347 size 27 |
| Box TOTAL TRIP COST | rect 487,357 · 248×62 r 9 | etichetta 611,379 · valore 611,411 size 34 |
| RESET | 773,409 | cliccabile |
| Alone verde | ellisse 864,303 · rx 104 ry 112 | + anello sfocato rx 100 ry 108 |
| VEHICLE STATUS | 864,240 | |
| Last Refuel Price | etichetta 864,266 · valore 864,292 | separatore y 302 (x 797→931) |
| Local Price | etichetta 864,319 · valore 864,344 | separatore y 354. Il valore è un `<text>` con due tspan: `#local` (prezzo, size 25) e `#localDist` (distanza dalla stazione, size 13, dx 7) |
| Fuel Level | etichetta 864,364 | barra 806,370 · 116×11 |
| Barra carburante | riserva rossa 808,372 13×7 · riempimento da x 821 | larghezza max 100 |
| E / F | 812,392 e 916,392 | sotto le estremità della barra |
| Icona pompa | translate(852,398) | con filtro glow |

## Geometria del quadrante

Il valore `v` (0–240 km/h) sta all'angolo:

```
angolo = -150° + (v / 240) × 300°      (0 = ore 12, orario)
x = 345 + r·sin(angolo)
y = 305 - r·cos(angolo)
```

Quindi: 0 km/h in basso a sinistra (-150°), 120 km/h in alto (0°), 240 km/h in basso
a destra (+150°). Le tacche fino alla velocità corrente sono accese (`.on`, bianche),
le altre spente (`.off`, blu-grigio).

## Bordo del parabrezza

Il foro della maschera segue l'array `edge` in `index.html`. Andamento: il vetro
scende da (0,0) in diagonale lungo il montante sinistro fino a ~(130,128), corre
quasi orizzontale sul cofano cruscotto (y ≈ 121–137 al centro) e risale a destra
fino a y ≈ 106 oltre x 920.

**Attenzione**: tra x 240 e x 320 la plancia ha un riflesso chiaro che la scansione
automatica scambiava per vetro; lì i punti sono stati alzati a mano (y ≈ 138–149).

## Palette

| Uso | Colore |
|---|---|
| Vetro schermo | `#0f2b41` → `#08192a` → `#03080e` (radiale) |
| Barra header | `#1d2c68` → `#16234f` → `#0c1638` |
| Bordo teal del pannello | `#6fd8b4` @ 45% |
| Testi valore | `#ffffff` |
| Etichette | `#cfe0ee` / `#bcd2e2` |
| Tacche accese / spente | `#eaf6ff` / `#40607d` |
| Verde vehicle status | `rgba(70,255,170,…)` · icona `#39e08a` |
| Accento UI comandi | blu `#00AEEF` · verde `#3ddc84` · ambra `#F0A500` |
