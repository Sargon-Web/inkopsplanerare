# Inköpsplaneraren — regler och beslut

Detta dokument beskriver HUR verktyget tänker och VARFÖR. Koden ligger i `index.html`
(en enda fil). Läs det här innan du ändrar beräkningslogik.

Senast uppdaterad: 2026-09-28

---

## Grundprincip

För varje produkt: *givet vad som står på hyllan, vad som är på väg och när det
landar, samt hur fort produkten säljer — hur mycket behöver beställas?*

Tre saker styr allt:

1. **Horisonten** = den dag ordern vi lägger IDAG blir säljbar hos oss.
2. **Täckningsmålet** = hur länge ordern ska räcka efter att den landat.
3. **Efterfrågemåttet** = vilken försäljningstakt vi räknar med.

PO-registret (steg 2) är enda källan för när inkommande leveranser landar.
Säljbart datum = ETA + dagar till hylla. Se avsnittet PO-registret nedan.

---

## Leverantörer

| Leverantör | Kanal | Ledtid till hylla | Beräkningsväg |
|---|---|---|---|
| Cheng | Båt + flygbrygga | båt 155 d, flyg 52 d | `runCalc` |
| Shalin Container | Endast båt | 120 d | `runCalc` |
| Shalin Vecka | Veckoköp (flyg) | 28 d | `runVeckaCalc` |
| EU | Dagligt köp | 8–10 d | `runEuCalc` |

Cheng och Shalin Container delar kod. Skillnaderna ligger i konfigurationen.

---

## Shalin Container

- **Täckning:** 6 mån (trend upp) / 5 (stabil) / 4 (ned).
  Motsvarar tiden till nästa container (~4–5 mån). Marginalen mot underköp
  ligger i veckoköpet, som är på hyllan på ~28 dagar.
- **Ingen flygbrygga.** `useAirBridge:false`. Vecka är snabbare (28 mot 52 dagar)
  till samma fraktkostnad och kan stoppas om containern kommer tidigt.
  Glappet fram till containern täcks alltså av Shalin Vecka.
- **Endast produkter som sålt 25+/år.** Resten hanteras av Vecka.
  Veckoköpet fyller glappet fram till containern (bryggan i Shalin Vecka).
- **Tom Lev.art.nr = Shalin är inte huvudleverantör** (gäller Container och
  Vecka, `skipNoLevArt`). Produkten köps inte här, och dess inköpsorder hör
  till en annan leverantör — den visas inte i registret och behöver inget
  datum. Produkterna listas i Granska.

## Cheng

- Behåller flygbryggan — det finns ingen veckokanal för Cheng.
- Beställs sällan, oftast bara i samband med container. Flygbryggan ska täcka
  hela glappet fram till att containern landar.
- Familjer: Genuine / 3Card / Övriga, med egna ny-order-ledtider.

## Shalin Vecka

- Beställs varje vecka. Ledtid 28 dagar till säljbar (uppmätt spann 22–38:
  pack 10–14 + frakt 5–10 + inleverans 7–14).
- **Beställningsgrind 35 dagar** = ledtid + en veckocykel (7). Räcker lagret
  (inkl. det som hinner fram före en ny veckoorder) minst så länge, väntar köpet
  — nästa veckas order hinner fram i tid. Grinden gäller ALLA köp, även bryggan.
  Ändras ledtiden ska grinden ändras lika mycket.
- Köpmängd: svag månad 3 st, stark månad 5 st. Bevisad (10+ på 3 mån) → 50 dagars täckning.
- **Bryggan är ett tak, inte ett mål.** Finns en leverans som landar efter att
  veckoordern hunnit fram, kapas köpet vid vad glappet kräver — aldrig mer.
  Stegprodukter lyfts aldrig över sitt steg (3/5).
  Ingen behöver hålla reda på containerdatum.
- **25+/år-produkter köps här som alla andra** (grind + brygga). Räcker det
  som är på väg köps inget. Räcker det inte köps det som behövs, och produkten
  flaggas "lägg på nästa container" — utom i bryggan, där en senare leverans
  redan är på väg. (Tidigare hoppades en 25+-produkt över helt så fort något
  var på väg inom ledtiden, även bara förra veckans lilla veckoorder — den
  spärrades då tills ordern var inlevererad och kunde hinna ta slut.)

## EU

- Beställs varje vardag. Måste nå ett minsta ordervärde för fri frakt.
- Arbetsregel: beställ det som tar slut inom 10–12 dagar.
- Klasser: Dropship (väntar tills minus) / Sålt-minus / Bevisad / Utfasning.

---

## Efterfrågemått

**Korta bindningar** (Vecka, EU, flygbrygga) använder `getDemand`:
vid trend upp tas MAX av viktat snitt, 1-månaderstakt och 3-månaderstakt.
Det är rätt när ordern rättar sig inom veckor.

**Långa bindningar** (båtordern) använder `getLongDemand`:
`0,2 × 1mån + 0,4 × 3mån + 0,4 × år`, aldrig under 3-månaderstakten.
En enskild het månad får inte driva en halvårsorder.

**Undantag — genuint ny produkt.** Sålde under 1,5 st/mån under månad 4–12,
alltså för lite historik för att årssiffran ska betyda något. Där styr senaste
takten. Testet mäter ÅLDER, inte tillväxttakt.

---

## Restnoteringsspärr

Negativt saldo betyder att varan redan är såld till en kund. Om inkommande
order inte täcker bristen ska den alltid beställas, oavsett klass — även för
utfasningsprodukter, där bara exakt bristen köps utan påslag.

Gäller alla leverantörer.

---

## PO-registret

- **Ett register, men varje leverantör sköter sina egna ordrar.** Varje order
  märks med leverantör (Shalin Vecka och Shalin Container räknas som en).
- **Synkas när alla filer är inlästa** — när man går till steg 2, och vid
  Beräkna om steg 2 hoppats över. Då hämtas nya ordrar ur filerna automatiskt
  och den inlästa leverantörens ordrar som inte längre finns i filerna
  (inlevererade) tas bort. Andra leverantörers ordrar rörs inte.
- **Borttagna ordrar sparas 60 dagar.** Dyker en order upp i filerna igen
  återställs den med sina datum i stället för att komma in tom.
- **Nollställ inte registret** mellan körningar — datumen följer med.
- **Vilka ordrar behöver datum:**
  - Cheng och Shalin Container: alla. Beräkningen stoppar annars.
  - Shalin Vecka: containerordrar. En order utan datum räknas som att den
    landar inom ledtiden — rätt för veckoordrar, fel för en container (då
    räknas hela containern som lager redan nu). Listan efter beräkningen
    markerar ordrar med 10+ st per produkt som "ser ut som en containerorder".
  - EU: inga. Allt som är på väg dras av: en ny order och en redan lagd har
    samma ledtid, så den lagda hinner alltid först. En EU-order som varit
    öppen i 21+ dagar (räknat från första dagen appen såg den) flaggas i
    steg 2 och i granskningen — annars räknas en död order som på väg för
    alltid och produkten föreslås aldrig igen. Appen minns per webbläsare.
- **Får vi en leverantör med längre ledtid** ska den läggas upp med egen
  ledtid och grind (som Vecka), inte köras på EU-logiken — EU-regeln
  förutsätter att leveransen kommer inom ~10 dagar.

---

## Kända begränsningar

- **Exporten från Kodmyran är trunkerad.** Ladda alltid in ALLA filer, och
  använd samma filuppsättning för Vecka och Container. En artikel som saknas
  i filerna beställs aldrig. Kontroll: 3-månadersfilens sista artikelnummer ska
  vara lika högt som 1-månadsfilens. 27 sep slutade 3-månadersfilen vid 244687
  medan 1-månadsfilen gick till 248552 — den var alltså avkapad.
- **Enhetspris exporteras inte**, så ordervärdet går inte att räkna i verktyget.
- **Partiorder syns inte.** Verktyget ser summerade antal och kan inte skilja
  "33 kunder köpte en var" från "två kunder köpte 29". Ordersnittet finns i
  Kodmyrans ProductStatistics men är inte inbyggt.

---

## Innan du ändrar beräkningslogik

1. Kör med gårdagens filer först och jämför rad för rad mot förra utkastet.
   Bara de rader som SKA ändras får ändras.
2. Ändra en sak i taget och deploya separat. Två ändringar samtidigt gör att
   du inte kan se vilken som gjorde vad.
3. Uppdatera Regelverket (i appen) och hjälptexterna i samma ändring, annars
   säger de snart något annat än koden gör.
4. Skriv en post överst i ÄNDRINGAR.md: vad som ändrades, vilka leverantörer
   det påverkar, facit (rader/styck med vilka filer och datum) och vad man ska
   hålla koll på. Spara indatafilerna och utkastet från körningen — med dem kan
   gammal och ny kod köras på samma data när ett fel dyker upp senare.
