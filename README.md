# Oppdrag-April-2026

# 🎨 Montana Cans – Produktdatabase

En SQL-database for å holde styr på Montana Cans sine produkter, farger og lagerbeholdning.

---

## 💡 Ideen

Jeg lagde denne databasen fordi jeg ville ha et søkefelt på nettsiden min der brukere kan søke opp ulike produkter. I stedet for å hardkode alt inn i HTML, ligger all produktinformasjon i databasen – slik at søkefeltet kan slå opp farger, produktnavn, HEX-verdier og serier dynamisk.

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
├── id
├── brand
├── product_name
├── content_ml
├── color_name
├── series
├── code
├── hex
├── cmyk
├── rgb
├── pressure_lvl
├── type
├── type_finish
├── price
├── amount
└── stock
```

---

## 👁️ Visninger

**`v_compact`** – Kompakt visning, terminalvennlig
```sql
SELECT * FROM v_compact;
```

**`v_summary`** – Oversikt over produktlinjer og hvor mange farger hver har
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

---

## 🛠️ Krav

- MariaDB eller MySQL
- Eller bruk [dbfiddle.uk](https://dbfiddle.uk) uten installasjon
