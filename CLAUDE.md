# CLAUDE.md

## Projekt
**Angličtina RPG** – interaktivní hra na učení angličtiny. Hráč leze po **nekonečné věži**; každé patro je souboj
s nepřítelem (vizuální styl inspirovaný Shakes & Fidget: velkohlavé kreslené postavy, tlusté obrysy, sytě barevné).
Správné slovíčko = energie ⚡, energie se utrácí za útoky (Seknutí, Ohnivá koule, Ultimátní úder, Léčení).
Za výhry jsou 🪙 mince, truhla a občas kořist (vybavení). Vybavení se kupuje v obchodě, spravuje v profilu/inventáři
a je vidět na postavě. Uživatel si na začátku zvolí pokročilost A1–C1; vyšší úroveň = víc mincí.

## Technické konvence
- Jediný soubor **`index.html`** (vanilla HTML/CSS/JS, bez buildu, bez závislostí) – nutné pro GitHub Pages.
- UI i komentáře česky; cvičení = překlad CZ → EN.
- Spuštění: otevřít `index.html` v prohlížeči.
- Postup hráče se ukládá do `localStorage` (klíč `anglictina-rpg-save-v2`; při změně struktury řešit migraci v `loadSave`).
  Pozor: v `data:`/preview kontextu je storage zakázané – vše je v try/catch.

## Architektura `index.html`
- Obrazovky: `s-start` → `s-map` (věž, `renderTower`) ⇄ `s-profile` (`renderProfile`) ⇄ `s-shop` (`renderShop`) → `s-battle` → `s-result`; přepíná `showScreen(id)`.
- Data:
  - `LEVELS[úroveň]` – `{label, coins, hp, dmg, zones[]}`; zóna = `{theme, icon, boss:{name}, words:["česky=anglicky",…]}`.
    Zóna trvá `ZONE_LEN` (10) pater, 10. patro zóny je boss; po vyčerpání zón se cyklí (boss dostane číslo II, III…).
  - `ATTACKS`, `DROP` (šance kořisti), `TYPING_LEVELS` (úrovně, kde se odpověď vždy píše).
- **Vybavení:** 12 typů (`TYPES`: zbraň, štít, helma, ramena, brnění, rukavice, boty, prsten, náušnice, opasek, amulet, trinket) a 15 slotů
  (`SLOT_KEYS`; prsteny, náušnice a trinkety mají po dvou). `it.slot` = typ, `save.equip[slotKey] = id`. Zbraně a brnění jsou v `ITEMS` ručně,
  ostatní typy generuje `gen()` (2× zelená, 2× modrá, 2× žlutá; bonus ×1 / ×2,6 / ×6). Bonusy: `atk` %dmg, `hp`, `def` % obrana (max 60), `en` %energie, `coin` %mincí.
- Profil (`renderProfile`) je MMO paper-doll: postava uprostřed, sloty kolem ní (levý sloupec, pravý sloupec, trinkety pod postavou), pod tím karta vybraného předmětu a inventář jako mřížka čtvercových polí; klik na slot/předmět vybere `profSel`.
- Nasazování: `equipItem`, `equipAction`; `snap()` + `diffText()` počítají změnu statistik a poškození kouzel (náhled v kartě předmětu i toast po nasazení).
  Poškození kouzel = `attackDmg(a)` (základ × level × vybavení) – používá souboj, profil i náhledy.
- Věž (`renderTower`): patra se kreslí zdola nahoru jako věž (střecha, patra se zdí podle zóny, náhled 3 pater výš a 2 vyčištěných níž, země na dně); obloha ve věži přechází do prostředí další zóny a po výhře hrdina "vyšplhá" (`climbPending`, animace `climbup`).
- Prostředí (`ENVS`, `envOf`, `applyEnv`): 8 prostředí (louka, les, hrad, sopka, led, nad mraky, noc, vesmír) se mění každou zónu (10 pater) a určují pozadí arény v souboji i barvu zdí ve věži. Nové prostředí = položka v `ENVS`.
- Věž (data): `getEnemy(level, patro)` – nepřítel je deterministický podle (úroveň, patro) díky `rngFor`. HP +12 %/patro, dmg +6 %/patro;
  boss zóny ×2,2 HP, ×1,4 dmg. `getDifficulty` – časový limit (od 6. patra), psaní odpovědí (od 9.), frekvence útoků bosse.
- **Nepřátelé:** jméno = přídavné jméno + druh (`NOUN`); `SP[druh](look)` vykreslí druh (16 druhů: Goblin, Skřet, Troll, Imp, Ogr, Slizoun, Netopýr,
  Kostlivec, Yetti, Golem, Démon, Vlkodlak, Zombík, Šotek, Pavoukoun, Faraón). Bossové zón mají druh v `BOSS_SPECIES` + korunu. `look` jen mění odstín, barvu očí a doplněk.
- **Hrdina:** `heroSVG()` skládá `ARMOR_ART`, `WEAPON_ART` (ručně podle id) a `GEAR_ART[typ](n, strana)` (každý z 6 kusů typu má vlastní tvar a barvy podle pořadí `it.n`) přes `gearSVG(slotKey)`.
  Nový typ/kus vybavení = položka v `ITEMS`/`gen()` + případně kresba v `*_ART`.
- Odměny (`endBattle`): základ `LEVELS[l].coins * (1 + 0,1*(patro-1)) * (1 + bonus mincí)`, ×1,2 bezchybně, porážka 25 %; truhla po výhře
  (boss ×3), kořist `rollDrop` (35 %, boss 100 %, čím lepší rarita, tím vzácnější; duplikát se prodá za 50 %).
- Migrace: staré sloty „šperky“ (`j1`–`j6`) se při načtení uložené hry proplatí (`OLD_JEWELRY_REFUND`).

## Jak rozšiřovat
- **Slovíčka:** `"česky=anglicky"` do `words` zóny. **Zóna/boss:** objekt do `LEVELS[...].zones` + druh v `BOSS_SPECIES`.
- **Druh nepřítele:** funkce v `SP` + jméno do `NOUN` (rod mužský kvůli přídavným jménům). **Útok:** `ATTACKS`.
- Po změnách ověřit v prohlížeči celý průběh: start → věž → souboj → výhra/prohra → truhla → profil → obchod → reload.

## Nápady na rozvoj
Zvuky, více typů cvičení (poslech, věty), speciální schopnosti vybavení, sady vybavení s bonusem, denní série,
statistika slabých slovíček a jejich opakování, žebříček.

## Testovací režim
`index.html?test=1` načte hotový postup (hrdina Lv. 50, všechno žluté vybavení, patro 50, 100 000 mincí) a ukládá ho do
samostatného klíče `anglictina-rpg-save-test`, takže skutečný postup zůstane nedotčený. Volitelné parametry: `&level=A1..C1`,
`&floor=N`, `&hero=N`, `&coins=N`, `&fresh=1` (znovu vytvořit testovací postup). Implementace: `makeTestSave`, `TEST`, `QS`.
