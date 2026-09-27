# Ändringslogg — Inköpsplaneraren

Nyaste överst. En post per uppdatering: vad som ändrades, vilka leverantörer
det påverkar, facit och vad man ska hålla koll på de första körningarna.
Hittar du ett fel senare: leta upp posten här, se om felet ligger i något
som ändrades, och jämför med facit.

---

## 2026-09-28 — Shalin: produkter utan Lev.art.nr köps inte

**Ändrar beräkningen:** Shalin Vecka och Shalin Container.
**Ändrar inte:** Cheng och EU.

### Ändring

- **Tom Lev.art.nr betyder att Shalin inte är produktens huvudleverantör.**
  Produkten är bara kopplad till Shalin i Kodmyran. Sådana produkter köps
  inte längre hos Shalin, varken i Vecka eller i Container.
- **Deras inköpsorder hör till en annan leverantör.** I dag gäller det
  3077 och 3206. De visas inte i Shalins PO-register och behöver inget datum.
  Steg 2 skriver "N order tillhör en annan huvudleverantör".
- **Granska listar produkterna som hoppades över** i en blå ruta.

### Facit

Filerna från 27 sep (alla 3), körd 27 sep, PO-datum som i körningen
(3077 och 3206 utan datum):

- **Shalin Vecka:** 327 rader, 1 267 st (förut 330 rader, 1 310 st).
  Borta: 235295 (25 st), 235233 (15 st), 235260 (3 st).
- **Shalin Container:** 4 rader och 255 st färre.
  Borta: 235295 (180 st), 235233 (40 st), 235275 (25 st), 235260 (10 st).
- Alla andra rader är oförändrade.

### Håll koll på

- **Om en produkt som Shalin faktiskt säljer saknar Lev.art.nr** köps den inte
  längre. Den syns då i den blå rutan i Granska. Rätta Lev.art.nr i Kodmyran.

---

## 2026-09-27 — Grind i bryggan, PO-register per leverantör, EU-flagga

**Ändrar beräkningen:** bara Shalin Vecka.
**Ändrar inte beräkningen:** Cheng, Shalin Container, EU. Där har bara
PO-registret, varningarna och texterna ändrats.

### Ändringar

1. **Shalin Vecka: grinden på 35 dagar gäller även bryggan.**
   Förut köptes produkter med en container längre fram varje vecka, oavsett
   lager. Exempel: 242712 räckte 80 dagar och fick ändå 5 st.
   Nu köps de först när lagret räcker kortare än 35 dagar.
2. **Shalin Vecka: bryggan lyfter inte längre steget 3 till 5.**
3. **Shalin Vecka: 25+-spärren är borttagen.**
   Förut hoppades en produkt som säljer 25 eller fler per år över helt så
   fort något var på väg inom 28 dagar, även bara förra veckans veckoorder.
   Exempel: 235295 hade 6 i lager och 25 på väg, fick inget och kunde bli tom.
   Nu får den 20 st.
4. **Shalin Vecka: bryggrader flaggas inte längre "lägg på nästa container"**,
   eftersom en container redan är på väg för dem.
5. **Alla leverantörer: PO-registret sköts per leverantör.**
   - Nya ordrar hämtas automatiskt när man går till steg 2, och knappen
     Auto-hämta är borttagen.
   - Inlevererade ordrar tas bort, men bara för den leverantör vars filer är
     inlästa, och först när alla filer är inne.
   - Borttagna ordrar sparas i 60 dagar och återställs med sina datum om de
     dyker upp igen.
   - Registret ska inte nollställas mellan körningarna.
6. **EU: ingen datumvarning längre**, eftersom EU inte använder datum.
   I stället flaggas en order som varit öppen i 21 dagar eller mer, räknat
   från första dagen appen såg den.
7. **Steg 1: varning när en exportfil ser avkapad ut**, det vill säga när den
   slutar långt under de andra filernas högsta artikelnummer.
8. **Texter:**
   - ledtid 28 dagar i stället för 33
   - grind 35 dagar i stället för 30
   - EU-kortet: 30 dagar i stället för 14
   - EU-bufferten beskrivs efter årstakten i stället för "+3"
   - den röda PO-rutan
   - guiden, där knappen "? Guide" inte fungerade alls förut

### Facit

Så här ska Shalin Vecka se ut med filerna från 27 sep (3-mån, 1-mån och
slut), körd 27 sep:

- **Förutsättningar i PO-registret:**
  - 3202 och 3203 säljbara 29 dec (ETA 15 dec + 14 dagar), som i körningen
    27 sep. Säljbar 17 dec för 3202 ger samma resultat.
  - övriga Shalin-ordrar säljbara inom 28 dagar, eller utan datum
- **Alla 3 filerna måste vara inlästa.** Med bara 3-mån och 1-mån blir det
  328 rader och 1 304 st, eftersom 245293 och 246086 bara finns i slutfilen.
- **Ny kod:** 330 rader, 1 310 st (filen `FACIT-shalin-vecka-veckokop-2026-09-27.csv`)
- **Gammal kod:** 347 rader, 1 400 st. De 17 rader som skiljer är exakt
  bryggraderna som hade 35 dagar eller mer.

### Håll koll på de första körningarna

- **Shalin Vecka:** en produkt med container på väg ska dyka upp först när
  lagret räcker kortare än 35 dagar. Står en sådan produkt tom, misstänk
  ändring 1 eller 3.
- **Shalin Container och Cheng (nästa gång):** beräkningen är oförändrad.
  Kontrollera att alla ordrar i steg 2 har rätt datum, eftersom registret nu
  sköts automatiskt. Saknas ett datum du fyllt i förut, misstänk ändring 5.
- **EU:** flaggan kan tidigast slå till cirka 21 dagar efter första körningen,
  alltså runt 19 okt.
- **Första körningen per leverantör efter uppdateringen:** var registret
  nollställt kommer ordrarna in utan datum. Fyll i dem en gång.

### Gå tillbaka till förra versionen

- **Vercel:** Deployments → välj förra deployen → gör den till produktion
  igen (rollback).
- **GitHub:** index.html → History → öppna förra versionen och kopiera den.
