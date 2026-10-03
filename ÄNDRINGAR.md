# Ändringslogg — Inköpsplaneraren

Nyaste överst. En post per uppdatering: vad som ändrades, vilka leverantörer
det påverkar, facit och vad man ska hålla koll på de första körningarna.
Hittar du ett fel senare: leta upp posten här, se om felet ligger i något
som ändrades, och jämför med facit.

---

## 2026-10-04 — Cheng: flygbryggan, Övrigas flygtid, datumstopp, delleverans

**Ändrar beräkningen:** bara Cheng, och bara flyget. Båtordern är oförändrad
på varje rad.
**Ändrar inte:** Shalin Container, Shalin Vecka och EU.

### Ändringar

1. **Flygbryggan räknas på senaste takten.** Förut räknade flyget med samma
   försiktiga takt som båten. Flyget täcker bara veckorna fram till containern
   och ska följa hur produkten säljer nu, vilket REGLER.md redan sa.
   Exempel: 240932 sålde 20 st förra månaden. Den försiktiga takten var
   ca 11 st/mån. Flyget går från 25 till 50 st, och båten ligger kvar på 80 st.
2. **Övriga Cheng flyger på sin egen ledtid.** Fältet "Flyg-ledtid övriga Cheng"
   (21 d) lästes aldrig förut, så Övriga räknades som om flyget tog 52 dagar.
   Hål före dag 52 räknades då som förlorade och köptes inte.
   Exempel: 228518 Ryggsträckare går från 15 till 30 st i flyg.
3. **Stopp vid Bekräfta och Export.** Om en order saknar datum i en ibockad
   familj, eller har fått datum efter beräkningen, kommer man inte vidare.
   Förut gick det att klicka förbi den röda rutan. Familjerna bockas ju i
   först efter Beräkna, så kontrollen där hoppades över för Cheng. Ett utkast
   blev ca 4 500 st för stort på det sättet (3158 räknades inte av).
   En odaterad Övriga-order (2396) stoppar bara när Övriga är ibockad.
4. **Delleverans räknas inte dubbelt.** Verktyget läser nu även raden
   "Totalt N enheter i befintliga inköpsorders". Är de beställda antalen fler
   än så dras skillnaden från den order som redan har kommit.
5. **Texter:** Regelverket (Cheng) och den röda PO-rutan.

Alla fyra ändringarna är kontrollerade var för sig. Bara 1 och 2 flyttar
siffror. Vilken av dem som ändrade en viss rad står i kolumnen "Varför" i
`cheng-nya-flygregler.xlsx`.

### Facit

Indata: filen `Cheng-alltfranlev-3e-okt-efterinlev.xlsx` (bara den), körd
3 okt kväll eller 4 okt. Inställningar: utfasningsgräns 240000, ny-order-ledtid
120/120, flyg 52, Övriga flyg 21, tröskel 15. PO-register: 3125 ETA 5 okt,
3148 12 okt, 3166 31 okt, 3158/3165/3214 10 dec, alla med 14 dagar till hylla.
2396 saknar datum.

| Familj | Gammal kod (rader / båt / flyg) | Ny kod (rader / båt / flyg) |
|---|---|---|
| Genuine | 155 / 4 520 / 660 | 157 / 4 520 / 825 |
| 3 Card | 84 / 2 250 / 270 | 84 / 2 250 / 350 |
| Övriga | 43 / 1 760 / 250 | 44 / 1 760 / 440 |

- **45 rader ändras, och bara flyget ökar.** Orsaken är senaste takten på 32
  rader, Övrigas flygtid på 11 och båda på 2. Nya rader: 7359, 19866 (Genuine)
  och 225320 (Övriga).
- Körs samma filer tidigare på dagen 3 okt kan enstaka rader flytta 5 st på
  grund av dagavrundningen.
- **Ändring 3:** med Övriga ibockad stoppar Bekräfta på 2396. Med Genuine
  eller 3 Card går det igenom.
- **Ändring 4:** morgonfilen 3 okt (mindre1ars) hade 246012 med 3158 (90) +
  3159 (30) och Totalt 95. 3159 räknas nu som 5 st i stället för 30.
  I efterinlev-filen slår regeln inte till någonstans.
- **Shalin Vecka och EU:** identiskt utfall rad för rad (Shalin-filen 24 aug:
  625 respektive 674 rader). Shalin Containers kodväg: identisk.

### Håll koll på

- **"Flyg-ledtid övriga Cheng" används nu.** Fältet ska vara den verkliga
  tiden till hylla för ett Övriga-flyg. Ett för lågt värde ger för mycket flyg
  på Övriga, ett för högt ger för lite.
- **En tillfälligt het månad ger nu mer flyg**, men inte mer båt. Titta extra
  på rader med varningen "Topptakt".
- **Stopprutan vid Bekräfta:** fyll i datumet i steg 2, gå till steg 3 och
  tryck Beräkna igen.
- **Övriga kan inte bekräftas förrän 2396 har fått ett datum.**

### Gå tillbaka till förra versionen

- **Vercel:** Deployments → välj förra deployen → gör den till produktion
  igen (rollback).
- **GitHub:** index.html → History → öppna förra versionen och kopiera den.

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
