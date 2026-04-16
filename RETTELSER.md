# Rettelser til DATA1500 Prosjekt

## 5. Datamodellen — Entiteter og Attributter

`KONTO` skal være `KONTOER` (følger samme skjema for navngiving av entiteter, - substantiver i flertall) alle steder i mermaid-koden:

```bash
# Eksempel på mermaid kode
BØKER ||--o{ KONTOER                 : "inneholder"
BØKER ||--o{ TRANSAKSJONER           : "inneholder"
BØKER ||--o{ BUDSJETTER              : "har"
BØKER ||--o{ REGNSKAPSPERIODER       : "definerer"
BØKER ||--o{ PLANLAGTE_TRANSAKSJONER : "har"
BØKER ||--o{ KUNDER                  : "har"
BØKER ||--o{ LEVERANDORER            : "har"
BØKER ||--o{ FAKTURAER               : "inneholder"

KONTOKLASSER ||--o{ KONTOER     : "klassifiserer"
KONTOER      ||--o{ KONTOER     : "er overordnet" # for å implementere hierarki av kontoer
KONTOER      }o--|| VALUTAER  : "denominert i"
KONTOER      }o--o| MVA_KODER : "bruker"
POSTERINGER  }o--|| KONTOER   : "berører"
LOT          }o--|| KONTOER   : "tilhører"
```
LOT trenger dere ikke å ta med, siden den var utelatt fra Oppgave 1 DEL A. Se kommentarer i avsnitt `Oppgave 1` under.

## 6. Prosjektoppgaver (NS 4102)

### Oppgave 1: Implementasjon av datamodellen, mermaid-diagrammet og normalform

- Skal kommentarer og ytelsesindekser implementeres, dvs. legges inn i skjema?
  - Ja
- I DEL A er ikke tabellen `Lot` nevnt, men i spesifikasjonen i "5.3 Transaksjoner og posteringer" er tabellen spesifisert, samt en fremmednøkkel er spesifisert mot `Lot` i tabellen `Posteringer`. 
  - Dere kan la være å implementere `Lot`. Dere kan da også droppe attributtet `lot_guid` i `Posteringer`.
 
### Oppgave 3: Grunnleggende SQL spørringer mot dobbelt bokholderi

Rettelser av feil:
- Del B: Aggregeringer med GROUP BY: `B.1:B.4` skal erstattes med `B.1`

### Oppgave 5: Ytelsesanalyse med `EXPLAIN ANALYZE` og `MATERIALIZED VIEW`

Initialiseringsskriptet for de nødvendige entitetene er lagt til i mappen `oppgave5`.

### Oppgave 9: Samtidighetsproblemer og Låsing (K10.7, K10.8, K10.9, K10.10, K10.11)

Rettelser av feil:
- I kommentarene i startkoden var det spesifisert en scenario, som potensielt kan føre til inkonsistent datagrunnlag: 

``` 
RACE CONDITION-MØNSTERET (Les-Beregn-Skriv):
---------------------------------------------
Tid  Tråd A (Ane)                    Tråd B (Bjørn)
 1   LES: saldo = 24 850 000 kr
 2                                   LES: saldo = 24 850 000 kr   ← SAMME verdi!
 3   [barriere.wait() — begge klare]
 4   BEREGN: 10 000 + 3 000 = 13 000
 5                                   BEREGN: 10 000 + 1 500 = 11 500
 6   SKRIV: INSERT postering +3 000
 7                                   SKRIV: INSERT postering +1 500
 8   COMMIT → saldo = 13 000
 9                                   COMMIT → saldo = 14 500 her

``` 

Den korrekte versjonen er: 

```
RACE CONDITION-MØNSTERET (Les-Beregn-Skriv):
---------------------------------------------
Tid  Tråd A (Ane)                    Tråd B (Bjørn)
 1   LES: saldo = 248 500 kr
 2                                   LES: saldo = 248 500 kr   ← SAMME verdi!
 3   [barriere.wait() — begge klare]
 4   BEREGN: 248 500 + 3 000 = 251 500
 5                                   BEREGN: 248 500 + 1 500 = 250 000
 6   SKRIV: INSERT postering +3 000
 7                                   SKRIV: INSERT postering +1 500
 8   COMMIT → saldo = 251 500
 9                                   COMMIT → saldo = 251 500
``` 