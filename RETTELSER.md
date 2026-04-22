# Rettelser til DATA1500 Prosjekt

## 4. Brukerscenarioer (NS 4102)

### Scenario 6: Kjøp av utenlandsk verdipapir (flervaluta)

**Hendelse:** Bedriften kjøper 10 aksjer i Apple Inc. (AAPL) for 175 USD per aksje. Valutakursen er 10,50 NOK/USD. Handelen koster totalt 1 750 USD = 18 375 NOK.

| Kontonr | Kontonavn | Kontoklasse | Debet | Kredit |
|---|---|---|---|---|
| 1350 | Aksjer i utenlandske selskaper | 1 — Eiendeler | 18 375 kr | — |
| 1920 | Bankinnskudd | 1 — Eiendeler | — | 18 375 kr |

**Forklaring:** Dette er en sammensatt flervalutatransaksjon. Posteringen på konto 1350 har `antall_teller = 10` (antall AAPL-aksjer) og `belop_teller = 175000` / `belop_nevner = 100` (1 750 USD). Posteringen på konto 1920 har `belop_teller = 1837500` / `belop_nevner = 100` (18 375 NOK). `Valutakurser`-tabellen inneholder kursen 10,50 NOK/USD for denne datoen. Et `Lot` opprettes for å spore kostprisen for fremtidig gevinstberegning (FIFO).

---
**Studentene har lagt merke til inkonsistens i spesifikasjonen. Håper dette oppklarer forvirring.**

Den korrekte forklaringen skal være:

Dette er en sammensatt flervalutatransaksjon. Posteringen på konto 1350 har `antall_teller = 10` (antall AAPL-aksjer) og `belop_teller = 1837500` / `belop_nevner = 100` (18 375 NOK). Posteringen på konto 1920 har `belop_teller = 1837500` / `belop_nevner = 100` (18 375 NOK). `Valutakurser`-tabellen inneholder kursen 10,50 NOK/USD for denne datoen. Et `Lot` opprettes for å spore kostprisen for fremtidig gevinstberegning (FIFO).

Transkasjon skal foregå i den valutaen som spesifisert i feltet `valuta_guid` `Transaksjoner`. Antall aksjer kjøpt spesifiseres i feltet `antall_teller` i `Posteringer.` I hvilken valuta ble aksjene kjøpt, skal lagres i feltet `valuta_guid` i `Kontoer` for kontoen 1350. `Lot` trenger man ikke ta hensyn til siden den ikke var spesifisert i Oppgave 1.

---

### Scenario 7: Innbetaling av MVA til staten (avregning)

**Hendelse:** Bedriften sender inn MVA-oppgave for 1. termin og betaler netto MVA til Skatteetaten. Utgående MVA er 12 500 kr (fra Scenario 3), inngående MVA er 875 kr (fra Scenario 2). Netto å betale: 11 625 kr.

| Kontonr | Kontonavn | Kontoklasse | Debet | Kredit |
|---|---|---|---|---|
| 2700 | Utgående MVA, høy sats | 2 — Egenkapital og gjeld | 12 500 kr | — |
| 2710 | Inngående MVA, høy sats | 2 — Egenkapital og gjeld | — | 875 kr |
| 2740 | Oppgjørskonto MVA | 2 — Egenkapital og gjeld | — | 11 625 kr |
| 1920 | Bankinnskudd | 1 — Eiendeler | — | 11 625 kr |

**Forklaring:** MVA-gjelden nulles ut. Netto MVA-beløp betales fra bankkontoen. Dette scenariet demonstrerer verdien av `Regnskapsperioder`-tabellen: perioden for 1. termin kan nå låses (`LAAST`) for å forhindre etterpostering.

---

**Studentene har lagt merke til inkonsistens i spesifikasjonen. Håper dette oppklarer forvirring.**

Tabellen ovenfor viser den faglig fullstendige varianten med 2740 Oppgjørskonto MVA, som krever to separate transaksjoner for å balansere:
- Transaksjon A (avregning): Debet 2700, Kredit 2710 og 2740 — nullstiller MVA-kontoene og registrerer skyldig netto MVA på oppgjørskontoen.
- Transaksjon B (betaling): Debet 2740, Kredit 1920 — betaler det skyldige beløpet fra bankkontoen.

Hvis du ønsker å bokføre dette som én enkelt transaksjon, skal konto 2740 utelates helt: Debet 2700 (12 500), Kredit 2710 (875) og Kredit 1920 (11 625). Begge varianter er regnskapsmessig korrekte.

| Spørsmål                                            | Svar                                                                         |
| --------------------------------------------------- | ---------------------------------------------------------------------------- |
| Er regnskapet i ubalanse mellom Transaksjon A og B? | **Nei** — 2740 representerer en legitim gjeld, regnskapet balanserer         |
| Kan de slås til én transaksjon?                     | **Ja** — men da uten 2740, og man mister tidsskillet                         |
| Hva er 2740 sin rolle?                              | En mellomkonto som viser at forpliktelsen eksisterer men ikke er betalt ennå |
| Når er tosplittingen viktig?                        | Når avregning og betaling faller i ulike regnskapsperioder                   |

---

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

**OBS! Endring fra den opprinnelige oppgaveteksten**

#### Opprinnelig oppgavetekst
Bruk Python programmeringsspråk for å simulerer "samtidighet", dvs. at Ane og Bjørn kobler seg til databasen med to separate Psycopg2-koblinger (hvor Psycopg2 er en mye brukt adapter for Python programmeringsspråket; https://www.psycopg.org/), som da utføres i to separate *tråder* i Python (`threading` modulen). 

Ta utgangspunkt i Python programmet i filen ´startkode/oppgave9.py´ hvor funksjoner for alle operasjoner mot databasen og samtidighet som simulerer den usikre *les-beregn-skriv*-syklusen er implementert. Dette skal gjennomgås på forelesninger.

#### Oppdatert oppgavetekst (2026-04-22)

Et fullstendig Python-kode for problemet som er beskrevet er gitt i mappen `oppgave9`. 
- Utfør koden og verifiser output, som spesifisert i den opprinnelige oppgaveteksten.

Ta med i rapporten:
- Hvordan simuleres samtidighet i Python-koden?
- Hvordan er transaksjoner implementert i Python-koden? 
- Hvorfor er en implementasjon basert på en modell med en saldo mer sårbar for brudd på ACID enn en implementasjon basert på debet/kredit og `belop_teller` og `belop_nevner`? 


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