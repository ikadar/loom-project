# Flux Print Shop Scheduling System - Project Brief

## Mi ez a projekt?

Egy nyomdai ütemező rendszert építünk. A nyomdának van ~10 gépe (stációja), és naponta 50-100 nyomtatási munkát kell ütemezniük. Jelenleg Excel-ben csinálják, de ez már nem skálázódik.

## A fő felhasználók

**Ütemezők** - ők azok, akik eldöntik, melyik gép mikor mit csináljon. Napi 8-10 órát töltenek a nyomda padlóján, folyamatosan átszervezik a munkákat, mert:
- Megrendelők változtatnak
- Papír nem érkezik meg
- Gépek elromlanak
- Sürgős munkák jönnek be

## Mit kell tudnia a rendszernek?

### 1. Stációk kezelése

Stáció = egy fizikai gép vagy munkahely. Például:
- Komori (nagy offset nyomógép)
- Massicot (vágógép)
- Heidelberg (kis offset)
- Kondícionálás (csomagolás)

Minden stációnak van:
- **Működési idő** - pl. hétfő-péntek 6:00-22:00, szombat 6:00-14:00
- **Kapacitás** - általában 1 (egy feladatot tud egyszerre), de vannak kivételek
- **Kategória** - hasonló munkákat végző gépek csoportja (pl. "Offset nyomók")

Speciális eset: **kiszervezett munkák**. Egyes feladatokat külső cégek végeznek (pl. lakkozás, stancolás). Ezeknek nincs kapacitás-korlátjuk, de napokban mérik az átfutást, nem percekben.

### 2. Munka (Job) és feladatok (Task)

Egy **munka** = egy megrendelés, pl. "5000 db szórólap, 4+0 szín, 135g matt műnyomó"

Minden munkának van:
- Megrendelő neve
- Határidő (mikor kell kiszállítani)
- Papír státusz (raktáron van? meg kell rendelni?)
- BAT státusz (jóváhagyta a megrendelő a próbanyomatot?)
- Lemezek státusza (elkészültek a nyomólemezek?)

Egy munka több **feladatból** áll, ezeket sorrendben kell végrehajtani:
```
1. [Komori] 20+40 "vízjeles műnyomó"    ← 20 perc beállítás + 40 perc nyomás
2. [Massicot] 15                         ← 15 perc vágás
3. ST [Clément] Lakkozás 2JO             ← kiszervezve, 2 munkanap
4. [Kondícionálás] 30                    ← 30 perc csomagolás
```

### 3. Az ütemező nézet

Ez a rendszer szíve. Egy vizuális rács, ahol:
- **Vízszintes tengely**: stációk (gépek) oszlopokban
- **Függőleges tengely**: idő (lefelé halad, mint egy naptár)

Az ütemező drag-and-drop-pal helyezi el a feladatokat a rácsra. Ha egy feladatot ráhúz egy stáció oszlopára, az "befoglal" egy idősávot.

#### Fontos szabályok:

1. **Nincs átfedés** - egy gép egyszerre csak egy feladatot csinálhat
2. **Sorrend számít** - a munka 2. feladata nem kezdődhet, amíg az 1. nem végzett
3. **Működési idő** - ha egy feladat átlóg a gép szünetébe, "nyúlik" (pl. 17:00-tól 23:00-ig tart a feladat, de a gép 18:00-22:00 között nem megy → 17:00-18:00 + 22:00-23:00)
4. **Határidő** - figyelmeztetés, ha egy munka késni fog

#### Hasonlósági jelzők

Ha két egymást követő feladat ugyanazon a gépen fut, és hasonló tulajdonságaik vannak (pl. ugyanaz a papír típus, méret, festékezés), akkor **kevesebb beállítási idő** kell. Ezt vizuálisan jelezzük:
- ● (teli kör) = ez a kritérium egyezik
- ○ (üres kör) = nem egyezik

Ez segít az ütemezőnek optimalizálni.

### 4. Konfliktus kezelés

Ha az ütemező olyan helyre húzza a feladatot, ami szabályt sért:
- **Precedencia sérül** → automatikusan a legközelebbi érvényes időpontra ugrik
- **Alt+húzás** → kényszeríti a helyet (piros halo jelzi a problémát)
- **Kapacitás túllépés** → sárga highlight

A rendszer megengedi a "rossz" ütemezést, de vizuálisan figyelmezteti az ütemezőt.

### 5. Bal panel: Munkák listája

- Összes munka listája szűrhető keresővel
- Kiválasztott munka feladatai jelennek meg
- Innen húzhatók a feladatok a rácsra
- "Visszahívás" gomb: ha egy már ütemezett feladatot vissza akarunk venni

### 6. Jobb panel: Késő munkák

- Azok a munkák, amik a jelenlegi ütemezés szerint nem férnek be a határidőbe
- Részletek a kiválasztott munkáról

## Technikai elvárások

- **Gyors feedback**: a drag-and-drop alatt <10ms válaszidő
- **100 tile renderelése**: <100ms
- **Optimista frissítés**: azonnal mutassa a változást, háttérben validálja

## Első verzió (MVP) korlátozások

- Egyetlen ütemezés (nincs branch-elés)
- Manuális task completion (nincs automatikus befejezés)
- Magyar és francia ünnepnapok nincsenek kezelve (csak hétfő-péntek)

---

## User Story-k (amit megvalósítunk)

### Stáció kezelés
- Ütemezőként szeretnék stációkat hozzáadni, hogy leképezzem a nyomda gépeit
- Ütemezőként szeretnék működési időt beállítani, hogy lássam mikor dolgozik a gép
- Ütemezőként szeretnék stáció kategóriákat kezelni a hasonlósági kritériumokhoz

### Munka kezelés
- Ütemezőként szeretnék munkát létrehozni az alapadatokkal
- Ütemezőként szeretném a papír és BAT státuszt kezelni
- Ütemezőként szeretném a munka feladatait DSL-lel megadni (gyors bevitel)

### Ütemezés
- Ütemezőként szeretném a feladatokat drag-and-drop-pal ütemezni
- Ütemezőként szeretném látni a hasonlósági jelzőket az optimalizáláshoz
- Ütemezőként szeretném látni a precedencia-sértéseket
- Ütemezőként szeretném a késő munkákat kiemelve látni

### Konfliktus kezelés
- Ütemezőként szeretném, ha a rendszer automatikusan a legjobb helyre ugrana
- Ütemezőként szeretném kényszeríteni a helyet Alt+drag-gel (override)
