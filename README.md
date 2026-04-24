# 🎨 Montana Cans – Produktdatabase

En SQL-database for å holde styr på Montana Cans sine produkter, farger og lagerbeholdning – bygget for å drive et søkefelt på en nettside.

---

## 💡 Ideen

Jeg lagde denne databasen fordi jeg ville ha et søkefelt på nettsiden min der brukere kan søke opp ulike produkter. I stedet for å hardkode alt inn i HTML, ligger all produktinformasjon i databasen – slik at søkefeltet kan slå opp farger, produktnavn, HEX-verdier og serier dynamisk.

Montana Cans har hundrevis av produkter og farger. Det er umulig å holde styr på alt manuelt. Med en database kan du oppdatere ett sted, og nettsiden din oppdateres automatisk. Ingen dobbeltarbeid, ingen feil.

All informasjon om farger, serienavn, koder, HEX-verdier, CMYK og RGB er hentet fra [montana-cans.no](https://www.montana-cans.no). Informasjonen ble deretter sortert og strukturert i Excel før den ble lagt inn i databasen.

---

## 🔎 Hvordan søket fungerer

Når en bruker skriver noe i søkefeltet på nettsiden, sendes søkeordet til databasen og matcher mot produktnavn, fargenavn og serie.

**Eksempel:**
Brukeren skriver `gold` i søkefeltet → databasen returnerer alle produkter som har ordet "gold" i seg:

```
Montana Gold 400ml – VANILLA
Montana Gold 400ml – CITRUS
Montana Gold 400ml – EASTER YELLOW
Montana Black Artist Edition GOLDEN GREEN – GALAXY
... og alle andre Gold-produkter
```

Dette skjer fordi vi bruker `LIKE '%gold%'` i SQL-spørringen, som betyr:
> "Finn alt som inneholder ordet gold, uansett hvor i teksten det står"

**Spørringen som kjører i bakgrunnen når du søker:**
```sql
SELECT * FROM v_compact
WHERE product_name LIKE '%gold%'
   OR color LIKE '%gold%'
   OR series LIKE '%gold%';
```

Du kan også søke på fargenavn:
- Skriv `blue` → får alle produkter med blue i fargenavnet
- Skriv `black 400` → får alle Montana Black 400ml produkter
- Skriv `#FF0000` → finner produktet med den eksakte HEX-fargen
- Skriv `shock` → får alle produkter i Shock-serien
- Skriv `high` → får alle produkter med høyt trykk

---

## 🧠 Hvorfor alt ble gjort som det ble gjort

### Én tabell i stedet for mange
Det første valget var om vi skulle ha én stor tabell eller dele opp i flere mindre. Vi valgte én tabell fordi alle produktene deler de samme kolonnene – produktnavn, farge, HEX, serie og så videre. Det hadde ikke gitt noe praktisk fordel å splitte det opp i separate tabeller for Montana Black og Montana Gold. Med én tabell er det enkelt å søke på tvers av alt på én gang, noe som er akkurat det søkefeltet trenger.

### `LIKE '%søkeord%'` for søk
Vi bruker `LIKE` med prosenttegn på begge sider fordi søkefeltet skal fungere uansett om brukeren skriver hele eller deler av et ord. Om noen skriver "blue", skal de få opp "BABY BLUE", "ROYAL BLUE", "LIGHT BLUE" og alle andre varianter – ikke bare eksakte treff. Det gjør søket mye mer brukervennlig.

### Visninger (views) for lesbarhet
Montana Cans har mange kolonner per produkt – HEX, CMYK, RGB, serie, kode og mer. Om du alltid henter alle kolonnene blir det kaotisk å lese, spesielt i terminalen. Derfor lagde vi to visninger: `v_compact` som viser det viktigste på en linje, og `v_summary` som gir deg en rask oversikt over produktlinjene. Du velger selv hvor mye du vil se.

### Produktnavn som grupperingsnøkkel
I stedet for å lage en egen tabell for hver produktlinje (Montana Black 400ml, Montana Gold 400ml osv.), bruker vi `product_name` som en kolonne. Det betyr at alle produktene med samme navn automatisk hører sammen. Vil du se kun Gold-produkter, filtrerer du bare på produktnavnet. Dette gjør det enkelt å legge til nye produktlinjer i fremtiden uten å endre strukturen.

### NULL for manglende data
Noen produkter – som Montana Effect-serien og Primer – har ikke fargedata, HEX-verdier eller serie. I stedet for å fylle inn falske verdier eller la feltet stå tomt på en måte som kan skape feil, bruker vi `NULL`. Det betyr tydelig "denne informasjonen finnes ikke", og databasen håndterer det ryddig uten å krasje søk eller visninger.

### Pris og lager satt til 0 og 10 som standard
Produktene ble lagt inn uten faktiske priser og lagertall siden det ikke var en del av dataen. I stedet for å la det stå tomt, satte vi `price = 0.00` og `stock = 10` som standardverdier. Det gjør at databasen fungerer med en gang, og du kan oppdatere prisene og lageret etter hvert som du legger inn ekte tall.

### MariaDB i stedet for SQLite
Nettsider kjører vanligvis på en server med MariaDB eller MySQL, ikke SQLite. Siden målet er å koble databasen til et søkefelt på en nettside, var det naturlig å bygge den i MariaDB fra starten av – så slipper du å konvertere den senere.

### utf8mb4 som tegnsett
Databasen bruker `utf8mb4` som tegnsett. Det betyr at den støtter alle tegn – norske bokstaver som æ, ø og å, spesialtegn i fargenavn, og til og med emojier. Uten dette ville spesielle tegn kunne bli korrupte eller vises feil på nettsiden.

### Kolonnen `type` og `pressure_lvl`
Montana Cans sine produkter har ulike trykknivåer (High, Medium, Low) og typer (Tech, Effect). Dette er nyttig informasjon for kunder som skal velge riktig produkt. Ved å ha det som egne kolonner kan nettsiden enkelt filtrere på dette – for eksempel vise kun Low-pressure produkter for nybegynnere, eller kun Effect-produkter i en egen kategori.

---

## 📦 Innhold

Totalt **516 produkter** fordelt på disse produktlinjene:

| Produktlinje | Antall |
|---|---|
| Montana Black 400ml | 181 |
| Montana Gold 400ml | 215 |
| Montana Black 600ml - Extended | 14 |
| Montana ULTRA WIDE 750ml | 12 |
| Montana Black Artist Editions | 31 |
| Montana Chalk 400ml | 10 |
| Montana Black 150ml | 7 |
| Montana Effect-produkter | 11 |
| Samarbeid (Heineken, Supreme osv.) | 12 |
| Andre (Stencil, Blackout, Whiteout osv.) | 13 |

---

## 🗂️ Tabelloppsett

```
products
│
├── id               – Unik ID for hvert produkt
├── brand            – Merkenavn (Montana)
├── product_name     – Produktlinje (f.eks. Montana Black 400ml)
├── content_ml       – Størrelse i milliliter
├── color_name       – Fargenavn (f.eks. ROYAL BLUE)
├── series           – Serie (f.eks. BLK, G, P, SHOCK)
├── code             – Fargekode (f.eks. 5077)
├── hex              – HEX-fargeverdi (f.eks. #054B90)
├── cmyk             – CMYK-verdier
├── rgb              – RGB-verdier
├── pressure_lvl     – Trykknivå (High, Medium, Low)
├── type             – Type (Tech, Effect)
├── type_finish      – Finish-type
├── price            – Pris
├── amount           – Antall per pakke
└── stock            – Lagerbeholdning
```

---

## 👁️ Visninger

**`v_compact`** – Kompakt visning, terminalvennlig. Viser de viktigste kolonnene på én linje per produkt.
```sql
SELECT * FROM v_compact;
```

**`v_summary`** – Oversikt over produktlinjer og hvor mange farger hver har. Nyttig for å se hva som finnes i databasen på et overordnet nivå.
```sql
SELECT * FROM v_summary;
```

---

## 🔍 Nyttige spørringer

```sql
-- Filtrer på produktnavn
SELECT * FROM v_compact WHERE product_name = 'Montana Black 400ml';

-- Filtrer på serie
SELECT * FROM v_compact WHERE product_name LIKE '%Gold%';

-- Søk etter fargenavn
SELECT * FROM v_compact WHERE color LIKE '%BLUE%';

-- Søk etter HEX-verdi
SELECT * FROM products WHERE hex = '#FF0000';

-- Søk på tvers av alt (slik søkefeltet på nettsiden gjør)
SELECT * FROM v_compact
WHERE product_name LIKE '%blue%'
   OR color LIKE '%blue%'
   OR series LIKE '%blue%';

-- Vis kun Effect-produkter
SELECT * FROM v_compact WHERE type = 'Effect';

-- Vis kun High pressure produkter
SELECT * FROM v_compact WHERE pressure = 'High';

-- Oppdater pris
UPDATE products SET price = 9.99 WHERE product_name = 'Montana Black 400ml';

-- Oppdater lager
UPDATE products SET stock = 50 WHERE id = 9;
```

---

## 🚀 Kom i gang

**Last inn databasen:**
```bash
mysql -u root -p < products.sql
```

**Åpne databasen:**
```sql
USE product_db;
```

**Test det online:** [dbfiddle.uk](https://dbfiddle.uk) – velg MariaDB og lim inn koden

Siden jeg ikke kunne ta med PC-en hjem, brukte jeg [dbfiddle.uk](https://dbfiddle.uk) for å teste og kjøre koden hjemmefra. Det er en gratis nettside der du kan kjøre SQL direkte i nettleseren uten å installere noe.
