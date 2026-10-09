# Klíma okos vezérlése Home Assistanttal (magyar)

Osztott klíma (reverzibilis hőszivattyú, fűt és hűt) pontos, kíméletes szabályozása **Better Thermostat**-tal. A klíma saját, gyakran rosszul mérő hőmérője helyett egy **külső szobahőmérő** alapján szabályoz. Nem kapcsolgatja a kompresszort, hanem a klíma saját célhőmérsékletét tolja el.

**Mi van benne:**

| Rész | Mit csinál |
|---|---|
| **[Better Thermostat beállítási útmutató](docs/BETTER_THERMOSTAT.md)** | lépésenként: külső hőmérő, fűtés+hűtés (heat_cool), ajtó/ablak, presetek, kalibráció, buktatók |
| **Klíma ütemezés blueprint** | éjszakai preset (pl. 22:00 alvás, 06:00 takarékos), távollét preset, **esti kikapcsolás csak hűtéskor** |
| **Klíma-kártya** (`hu-klima-card`) | a Better Thermostat UI tárcsáját egészíti ki: ajtó-sáv, a klíma belső hőmérője és a jelenlét a tárcsában, teljesítmény és fogyasztás, automatika-sor. A kártya kódja a [ha-futes-hu](https://github.com/Gyuszko55/ha-futes-hu) tárolóban van, onnan telepíthető. |

> [!WARNING]
> A csomag a klímát csak a Home Assistant szokásos szolgáltatásain át vezérli (preset, üzemmód). A klíma saját védelmeit (kompresszor-késleltetés, fagyvédelem stb.) nem helyettesíti. Saját felelősségre, garancia nélkül (MIT licenc).

---

## Telepítés feltételei – mit kell előre telepíteni

Kipipálható lista, sorrendben:

| # | Mit | Honnan / hogyan | Kötelező? |
|---|---|---|---|
| 1 | **Home Assistant 2025.10 vagy újabb** | Beállítások → Rendszer → Frissítések | igen |
| 2 | **A klíma gyári integrációja**, amivel a HA látja és vezérli a klímát: pl. Gree, Daikin, Midea, Mitsubishi, Panasonic, LG ThinQ, Tuya, vagy egy infravörös vezérlő (Broadlink + SmartIR) | Beállítások → Eszközök és szolgáltatások → Integráció hozzáadása (vagy HACS) | igen. A klímának `climate` entitásként kell megjelennie, fűtés és/vagy hűtés üzemmóddal. |
| 3 | **Külső szobahőmérő** a klíma által fűtött/hűtött helyiségben (Zigbee, BLE, WiFi, bármi) | a hőmérő saját integrációja | igen. Ez adja a pontos hőmérsékletet. |
| 4 | **[HACS](https://hacs.xyz/)** (Home Assistant Community Store) | https://hacs.xyz/docs/use/ – telepítési útmutató | igen (a 5–6. ponthoz) |
| 5 | **Better Thermostat** integráció | HACS → Integrációk → keresés: „Better Thermostat” → Letöltés → HA újraindítás → Beállítások → Integráció hozzáadása → Better Thermostat | igen |
| 6 | **Better Thermostat UI** kártya | HACS → Frontend → keresés: „Better Thermostat UI” → Letöltés | a kártyához igen |
| 7 | **Ajtó/ablak-nyitásérzékelő** abban a helyiségben | az érzékelő integrációja | nem, de ajánlott: nyitott ajtónál a klíma szünetel |
| 8 | **Home Assistant Companion app** a telefonokon, `person` entitásokhoz rendelve, **„Mindig” helyhozzáféréssel és akkumulátor-korlátozás nélkül** (lásd lent: [Telefonok a távolléthez](#telefonok-a-távolléthez)) | App Store / Google Play; Beállítások → Emberek | csak a távollét-szekcióhoz |
| 9 | **Teljesítménymérő** a klíma áramkörén (pl. Shelly EM / Plug) | a mérő integrációja | nem, csak a kártya W-kijelzéséhez |
| 10 | *Nem kötelező:* a [ha-futes-hu](https://github.com/Gyuszko55/ha-futes-hu) **jelenlét csomagja** (Otthon / Távol / Szabadság), és a [ha-energia-elszamolas-hu](https://github.com/Gyuszko55/ha-energia-elszamolas-hu) a H-tarifás fogyasztáshoz | a tárolók README-je | nem |

## Telepítés

### 1. Better Thermostat beállítása

Kövesd a **[Better Thermostat beállítási útmutatót](docs/BETTER_THERMOSTAT.md)**. Az eredmény egy új termosztát, pl. `climate.bt_nappali_klima`. Ezt használd mindenhol, ne a gyárit.

### 2. Klíma ütemezés blueprint

[![Blueprint importálása](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FGyuszko55%2Fha-klima-hu%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fhu_klima%2Fklima_utemezes.yaml)

Vagy kézzel: Beállítások → Automatizálások és jelenetek → Blueprintek → **Blueprint importálása**, és illeszd be ezt a címet: `https://github.com/Gyuszko55/ha-klima-hu/blob/main/blueprints/automation/hu_klima/klima_utemezes.yaml`.

1. *Nem kötelező, de ajánlott:* hozz létre egy **kapcsoló segédet** („Klíma automatika” → `input_boolean.klima_auto`): Beállítások → Eszközök és szolgáltatások → Segédek → Segéd létrehozása → Kapcsoló. Ezzel az automatikát egy mozdulattal ki lehet kapcsolni.
2. Automatizálások → Új → Blueprintből → **Klíma ütemezés**:
   - **Klíma:** a Better Thermostat entitás;
   - **Fő kapcsoló:** a fenti segéd;
   - **Figyelt személyek:** a háztartás tagjai;
   - **1. Éjszakai preset:** pl. 22:00 `sleep`, 06:00 `eco`;
   - **2. Távollét:** ha kell, pl. 30 perc után `away`. Háromállapotú (Szabadság) működéshez inkább a ha-futes-hu „Jelenlét alapú preset” blueprintjét használd, és ezt hagyd kikapcsolva.
   - **3. Esti kikapcsolás hűtéskor:** pl. 20:30. Gyári entitásnak a klíma saját entitását add meg, mert annak az üzemmódja mutatja a hűtést.

### 3. Kártya

1. Töltsd le a kártya fájlját a ha-futes-hu tárolóból, a `config/www/hu-termosztat/` mappába. Terminálból (Terminal & SSH add-on):

   ```sh
   mkdir -p /config/www/hu-termosztat && curl -sL -o /config/www/hu-termosztat/hu-termosztat.js \
     https://raw.githubusercontent.com/Gyuszko55/ha-futes-hu/main/www/hu-termosztat/hu-termosztat.js
   ```

2. Beállítások → Irányítópultok → ⋮ → **Erőforrások** → Hozzáadás:
   - URL: `/local/hu-termosztat/hu-termosztat.js?v=3`
   - Típus: **JavaScript modul**

   Ha nem látod az Erőforrások menüt, a profilodban kapcsold be a Haladó módot.
3. Kártya hozzáadása → Kézi kártya → a [dashboard/klima_kartya.yaml](dashboard/klima_kartya.yaml) példája a saját entitásaiddal.

   A kártya fájlja a ha-futes-hu tárolóval közös: ha ott új változat jön, töltsd le újra, és az erőforrás `?v=` számát
   emeld meg, különben a böngésző a régit használja.

### Telefonok a távolléthez

A 2. szekció (távollét) a telefonok helyzetéből dönt. Ha egy telefon helyzete a háttérben nem frissül, a klíma nem
vált távollét presetre, vagy hazaérkezéskor későn vált vissza. Minden családtag telefonján:

1. a Home Assistant appnak a helymeghatározás **„Mindig engedélyezve”** és **„Pontos hely”**;
2. akkumulátor: **„Nincs korlátozás”** – Xiaomi / Redmi / POCO telefonon az **Automatikus indítás** is legyen bekapcsolva,
   Samsungon vedd ki az „Alvó alkalmazások” közül;
3. az appban a háttérbeli helymeghatározás bekapcsolva, és a telefon `device_tracker`-e a `person` entitáshoz rendelve
   (Beállítások → Emberek);
4. ellenőrzés: elmenve a `person` pár percen belül „Távol”-ra (vagy egy zóna nevére) váltson, hazaérve „Otthon”-ra.

---

## Hogyan működik

- **A kalibráció:** ha a szobahőmérő 21 °C-ot mér, a klíma saját hőmérője 23 °C-ot, és 22 °C-ot szeretnél, a Better Thermostat a klímának 24 °C-ot állít be. Így a *szobában* lesz 22 °C. A kompresszort nem kapcsolgatja, a klíma a saját ütemében szabályoz.
- **Fűtés és hűtés egyben (heat_cool):** a termosztát két célhőt kap. Az alsó alatt fűt, a felső fölött hűt, a kettő között pihen. A presetek (eco, sleep, away …) mindkét értéket beállítják.
- **Nyitott ajtó/ablak:** a beállított késleltetés után a Better Thermostat szünetelteti a klímát, bezáráskor folytatja. A kártya ezt sávval jelzi.
- **Az ütemezés** csak presetet vált, és este, hűtés közben kikapcsol. Minden más a Better Thermostat dolga.

Részletek és buktatók: [docs/BETTER_THERMOSTAT.md](docs/BETTER_THERMOSTAT.md).

## Licenc

MIT, garancia nélkül.
