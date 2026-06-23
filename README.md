# Bananfläta – komma igång (GitHub Pages + Firebase)

Spelet är en enda fil: **`index.html`**. Den hostas statiskt (GitHub Pages) och
använder **Firebase Realtime Database** som realtidsrygg så att två (eller upp till
åtta) telefoner kan spela mot varandra var som helst över internet.

Du gör två saker en gång: skapar ett gratis Firebase-projekt och lägger upp filen på
GitHub Pages. Sen är det bara att dela länken.

---

## 1. Skapa Firebase-projekt (gratis, ~3 min)

1. Gå till **console.firebase.google.com** → *Add project*. Ge det ett namn
   (t.ex. `bananflata`). Du kan hoppa över Google Analytics.
2. I vänstermenyn: **Build → Realtime Database → Create Database**.
   - Välj region (t.ex. *europe-west1*).
   - Välj **Start in test mode** → Enable.
3. Hämta din webbkonfig: kugghjulet uppe till vänster → **Project settings** →
   längst ned under *Your apps* → klicka webb-ikonen **`</>`** → registrera appen →
   kopiera objektet **`firebaseConfig`** (apiKey, authDomain, **databaseURL**,
   projectId, appId …).

> Viktigt: objektet måste innehålla **`databaseURL`** (den med
> `...firebasedatabase.app` eller `...firebaseio.com`). Saknas den har du inte
> aktiverat Realtime Database i steg 2.

### Databasregler
"Test mode" ger öppen läs/skrivning men **slutar gälla efter 30 dagar**. För ett
familjespel kan du sätta en enkel regel som inte går ut: i **Realtime Database →
Rules**, klistra in och publicera:

```json
{ "rules": { ".read": true, ".write": true } }
```

Detta är medvetet öppet (vem som helst med din databas-URL kan läsa/skriva). Det är
okej för ett litet privat spel. Vill du strama åt senare kan jag hjälpa dig med det.

---

## 2. Lägg in din config

**Alternativ A – i appen (enklast, inget filpil):** öppna sidan, tryck
**"Anslut Firebase"**, klistra in `firebaseConfig`, spara. Configen sparas på den
enheten. Varje telefon gör detta en gång (samma projekt på båda).

**Alternativ B – i filen (en gång för alla):** öppna `index.html`, leta upp blocket
högst upp:

```js
window.__FIREBASE_CONFIG__ = {
  apiKey: "KLISTRA_IN",
  ...
  databaseURL: "KLISTRA_IN_databaseURL",
  ...
};
```

Ersätt med din riktiga config och spara. Då slipper alla mata in den – det bara
funkar när de öppnar länken.

---

## 3. Lägg upp på GitHub Pages

1. Skapa ett repo (t.ex. `bananflata`) och lägg `index.html` i roten.
2. Repo → **Settings → Pages** → *Build and deployment* → Source: **Deploy from a
   branch** → Branch: `main` / `/ (root)` → Save.
3. Efter en stund får du en URL: `https://<ditt-användarnamn>.github.io/bananflata/`.

Öppna den länken på båda telefonerna.

---

## 4. Spela

1. Skriv namn. Tryck **Skapa rum** → du får en 4-teckens kod.
2. Den andra öppnar samma länk, skriver namn, fyller i koden, **Gå med**.
3. Båda trycker **Jag är redo** → gemensam nedräkning → brickorna delas ut samtidigt.
4. Under spel: **SKALA** (klar + klasen räcker → alla drar en bricka), **DUMPA**
   (markera bricka → lägg tillbaka 1, dra 3), **BANAN!** (klasen tom nog och du är
   klar → de andra granskar din fläta). Vinst = **Toppbanan**, sparas i topplistan.

Tips: tryck **Testa anslutning** på startsidan – står det "Firebase svarar ✓" är
allt rätt inkopplat. Diagnostikraden i väntrummet visar hur många spelare som syns.

---

## Felsökning

- **"Firebase svarar ✗" / "Kunde inte skriva/läsa"** – fel/saknad config, eller
  Realtime Database inte aktiverad, eller reglerna inte öppna (se steg 1).
- **Medspelaren syns inte** – kontrollera att båda använder **samma** Firebase-projekt
  (samma config) och rätt rumskod.
- **Allt funkar lokalt men inte mellan nät** – det ska funka över internet via
  Firebase; om inte, dubbelkolla `databaseURL` och regler.
- **Bytt config i appen?** Ladda om sidan efter att du sparat – Firebase
  initieras en gång per sidladdning.

## Robusthet (inbyggt)

Byggt efter Firebase officiella mönster för att undvika vanliga fällor:

- **Servertid** (`.info/serverTimeOffset`) används för alla tidsstämplar, så
  närvaro och nedräkning funkar även om telefonernas klockor går olika.
- **Närvaro via `.info/connected`** – `onDisconnect` återregistreras vid varje
  återanslutning, så en kort WiFi-glapp slänger inte ut en spelare permanent.
- **Värd-auktoritativ** sync: en enhet räknar klasen (inga dubblerade brickor),
  med automatisk övertagning om värden tappar uppkopplingen, och skydd mot
  dubbel-värd när den ursprungliga värden kommer tillbaka.

## Kostnad

Ett familjespel ligger med god marginal inom Firebase gratisnivå (Spark):
upp till 100 samtidiga anslutningar och 1 GB lagring. Du betalar inget.

Bokstavsfördelningen (officiell svensk, 144 brickor, ingen Q/W/Z) ligger under
**Regler & brickor** i appen.
