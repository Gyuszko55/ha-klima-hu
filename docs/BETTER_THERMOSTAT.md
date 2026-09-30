# Better Thermostat beállítása osztott klímához

Egy valós, teraszi, reverzibilis (fűtő és hűtő) osztott klíma bevált beállítása. A menüpontok nevei a Better Thermostat verziójától függően kicsit eltérhetnek.

## Miért kell?

- **A klíma beépített hőmérője** a beltéri egységben, a mennyezet közelében, a kifújt levegő mellett mér. Télen jellemzően kevesebbet, nyáron többet mutat a valós szobahőmérsékletnél. Egy mért példa ugyanabban a pillanatban: a klíma 18,0 °C-ot, a szobahőmérő 19,1 °C-ot mutatott.
- **Relével kapcsolgatni a klímát nem szabad.** A kompresszornak árt, és a garanciát is érintheti.
- **A Better Thermostat megoldása:** a klímát bekapcsolva hagyja, csak a *célhőmérsékletét* tolja el a külső hőmérő szerint. Így a szobában lesz meg a kívánt hőfok.

## Előtte ellenőrizd

1. A klíma gyári entitása (pl. `climate.nappali_klima`) működik: a HA-ból be- és kikapcsolható, és állítható a célhő.
2. A külső hőmérő **élő** értéket ad. Nézd meg az előzményeit: ha órák óta ugyanaz a szám, a szenzor lefagyott. Ez a leggyakoribb rejtett hiba: egy lefagyott kinti hőmérő nálunk három rendszert is elrontott, mire kiderült.
3. A Better Thermostat (HACS) telepítve van, és a HA újra lett indítva.

## Beállítás lépésenként

Beállítások → Eszközök és szolgáltatások → **Integráció hozzáadása** → **Better Thermostat**.

| Mező | Mit adj meg | Megjegyzés |
|---|---|---|
| **Név** | pl. „Nappali klíma” | Ebből lesz a `climate.bt_…` entitás, a továbbiakban ezt használd. |
| **Termosztát / fűtőeszköz** | a klíma gyári entitása | |
| **Hűtőeszköz (cooler)** | **ugyanaz** a gyári entitás | Reverzibilis gépnél ugyanaz az egység látja el mindkét szerepet. Így lesz fűtés és hűtés is (`heat_cool`). |
| **Hőmérséklet-szenzor** | a külső szobahőmérő | a legfontosabb mező |
| **Páratartalom-szenzor** | a szobahőmérő páratartalma | nem kötelező, a kártyán látszik |
| **Kültéri hőmérő** / **időjárás** | élő kinti hőmérő, illetve időjárás-entitás | Ellenőrizd, hogy élő-e (lásd fent). |
| **Ablakérzékelők** vagy **ajtóérzékelők** | a helyiség nyílászárója | Csak azt add meg, ami tényleg ahhoz a helyiséghez tartozik, ne egy egész házas csoportot. |
| **Ablak / ajtó késleltetés** | pl. ajtónál 60 mp, ablaknál 10 perc | Ennyi nyitva tartás után szünetel a klíma. |
| **Tolerancia** | pl. 0,2 °C | |
| **Presetek** | `eco`, `away`, `comfort`, `home`, `sleep` | Legyen benne mind, amit az ütemezés használ. |

**A klíma haladó beállításai** (a termosztát-eszköznél):

| Mező | Érték nálunk | Megjegyzés |
|---|---|---|
| **Kalibráció** | célhő alapú (`target_temp_based`) | Klímánál ez a helyes, mert a klíma nem ad szelepállást. |
| **Kalibrációs mód** | `heating_power_calibration` | Bevált. Az újabb módok (MPC, PID, TPI) finomabbak lehetnek, de ezek külön hangolást igényelnek. |

A presetek hőfokait a létrehozás után állíthatod be. Fűtés/hűtés esetén mindegyiknek két értéke van: alsó (fűtés) és felső (hűtés) határ. Egy bevált példa teraszra:

| Preset | Fűtés (alsó) | Hűtés (felső) | Mikor |
|---|---|---|---|
| `eco` | 19 °C | 27 °C | nappal |
| `sleep` | 18 °C | 22 °C | éjjel |
| `away` | 16 °C | 28 °C | távollét, szabadság |

## Buktatók, amikbe mi belefutottunk

1. **Lefagyott kinti szenzor:** a Better Thermostat nem jelez, csak rosszul dönt. Ha a viselkedés furcsa, először a kinti hőmérő előzményeit nézd meg.
2. **Ajtóérzékelő vs. ablakérzékelő:** ha a nyílászárót *ajtóérzékelőként* adod meg, a Better Thermostat `door_open` állapotot jelez. A Better Thermostat UI kártya viszont csak a `window_open`-t figyeli, ezért nyitott ajtónál nem mutat semmit. A **`hu-klima-card`** ezt megoldja: a beágyazott kártya nyitott ajtónál is szünetet mutat, és fölötte sáv jelzi, hány másodperc múlva áll le.
3. **Kattintás az éppen aktív presetre:** a Better Thermostat UI kártya ilyenkor „nincs preset” (kézi) módba vált, és a hőfokok elállítódnak. A `hu-klima-card` ezt a kattintást elnyeli, ugyanarra a presetre kattintva nem történik semmi.
4. **Egy egész házas ablakcsoport** bekötése: a klíma akkor is szünetel, ha a ház túlsó végén nyitnak ablakot. Csak a saját helyiség érzékelőjét add meg.
5. **Esti kikapcsolás:** ha a klímát este kikapcsolod, azt csak hűtéskor tedd (a blueprint 3. szekciója ezt tudja). Téli fűtésnél a hajnali lehűlés után a klíma nagy teljesítménnyel indulna újra.
6. **Gyári vs. BT entitás:** az ütemezésben, a kártyán és a hangvezérlésben mindig a `climate.bt_…` entitást használd. A gyárit csak az esti kikapcsolás „gyári entitás” mezőjében add meg.

## Fogyasztás (nem kötelező)

- **Teljesítmény:** ha a klíma áramkörén teljesítménymérő van (pl. Shelly), a kártya a pillanatnyi teljesítményt mutatja (`power_entity`).
- **H-tarifa:** ha a klíma H-tarifás mérőn van, a [ha-energia-elszamolas-hu](https://github.com/Gyuszko55/ha-energia-elszamolas-hu) csomag a téli és nyári fogyasztást külön számolja. A kártya ezt is ki tudja írni (`h_winter_yearly_entity`, `h_summer_yearly_entity`).
