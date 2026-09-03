# 10 · Pubblicazione: dove vive il progetto e come si aggiorna

Il *perché* di questa impostazione è nella voce del 2026-09-03 in
[01-decisioni.md](01-decisioni.md). Qui c'è solo il come.

## I tre indirizzi

| Cosa | Dove |
|---|---|
| **La demo live** (il link da mandare in giro) | https://zirullo.github.io/True-Cost/ |
| Il repo | https://github.com/Zirullo/True-Cost — pubblico, ramo `main` |
| La cartella di lavoro | `d:\Cowork\Simulator\` |

Il link della demo **non cambia mai**: GitHub Pages serve sempre l'`index.html` in
cima a `main`. È legato al ramo, non a una versione, quindi si aggiorna senza che
l'indirizzo si sposti — che è il motivo per cui esiste in questa forma.

## Come si pubblica una modifica

Dalla cartella di lavoro, in quest'ordine:

```
git add -A                     # metti in conto tutto quello che hai toccato
git commit -m "..."            # congela l'istantanea, sul PC
git push                       # mandala su GitHub → la demo si aggiorna in ~1 min
```

I primi due sono locali e reversibili. Il terzo è quello che il pubblico vede.

**Il messaggio di commit conta**: fra sei mesi è l'unica cosa che dice cosa avevi
fatto. "I pedali guidano davvero" si legge, "Add files via upload" no.

## La regola che non va rotta

**Non caricare file dalla pagina web di GitHub.** Crea un commit che il PC non ha; il
`push` successivo viene rifiutato (`! [rejected] main -> main`) e va risolto un
conflitto a mano dentro `index.html`. Se dovesse succedere comunque: `git pull`
prima di riprendere a lavorare, e sistemare i marcatori `<<<<<<<` nel file.

## Tornare a una versione passata

Ogni commit è un'istantanea sigillata dell'intera cartella. I tag sono solo nomi
leggibili appiccicati sopra, per ritrovare i punti che contano.

| Tag | Commit | Cos'è |
|---|---|---|
| `v1.0` | `da8bc75` | Prima versione marcata: cockpit, strada procedurale, pedali che guidano, cartelloni, cavalcavia, stazioni, dirigibile e UFO |

Tre modi, dal più cauto al più invasivo:

```
# 1. solo guardare — zip in Download, il progetto non si tocca
#    https://github.com/Zirullo/True-Cost/archive/refs/tags/v1.0.zip

# 2. recuperare un singolo file, lasciando intatto il resto
git checkout v1.0 -- index.html

# 3. riportare indietro tutto il progetto, senza cancellare la storia in mezzo
git revert --no-commit v1.0..HEAD
git commit -m "Torno alla 1.0"
```

Per marcare una nuova versione: `git tag -a v1.1 -m "..."` poi `git push origin v1.1`,
e aggiungere una riga alla tabella qui sopra.

## Cosa non finisce su GitHub

`.gitignore` tiene fuori i backup manuali — `BCK/`, `OLD index.html`, le copie di
`index`. Restano sul disco, ma la storia che raccontavano è nei commit.

## Le copie che esistono

Due indipendenti: `d:\Cowork\Simulator\.git` sul PC, e GitHub. Perderle entrambe
richiede un guasto **e** la cancellazione del repo. Lo zip di un tag su un disco
esterno è la terza, e non dipende da nessuna delle due.
