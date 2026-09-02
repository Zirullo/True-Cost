# 00 · Il progetto

## L'idea

Chi guida ragiona in **soldi**, non in litri o kWh. Nessun cruscotto oggi mostra
quanto costa realmente un chilometro, in tempo reale. **True Cost** è una vista
del quadro strumenti che risponde a una domanda sola:

> *"Quanto mi sta costando questo viaggio, adesso?"*

Tre numeri, sempre visibili:

1. **€ / km istantanei** — consumo corrente × prezzo del carburante in serbatoio
2. **Costo totale del viaggio** in euro, che sale mentre guidi
3. **Contesto prezzi** — quanto è costato il pieno che stai bruciando vs quanto
   costa il carburante *qui e ora*

## Perché è credibile

Non serve hardware nuovo: i dati ci sono già.

- Consumo istantaneo e velocità → già sul bus del veicolo
- Prezzo del pieno e prezzo locale → **Stellantis Connected Services** + dati
  prezzi carburante aperti (vedi [05-dati-esterni.md](05-dati-esterni.md))
- Il resto è software di visualizzazione

## Contesto

- **Programma**: Star\*Up 2026 (Stellantis)
- **Form feedback**: https://forms.office.com/e/jJ7Sgw7WkD (linkato nella UI)
- **Interesse dichiarato**: integrazione con Stellantis Connect Fleet (citato nei
  materiali del progetto — da riverificare prima di ripeterlo in un pitch, vedi
  [08-domande-aperte.md](08-domande-aperte.md))

## Cosa è il simulatore

`index.html` è una **demo interattiva a scopo dimostrativo**: mostra l'idea a chi
non può salire su una vettura prototipo. Non è codice destinato al veicolo.
Serve per pitch, raccolta feedback e validazione del concetto.

Il target visivo è la foto `True-Cost project picture.jpg`: cockpit reale con il
cluster digitale che mostra la vista True Cost. Il simulatore riproduce quella
schermata, ma **viva**: i numeri si muovono con la guida.

Vedi [02-architettura.md](02-architettura.md) per come è costruito e
[01-decisioni.md](01-decisioni.md) per le scelte fatte.
