# CLAUDE.md

## Projekt
**Angličtina RPG** – interaktivní hra na učení angličtiny. Hráč leze po **nekonečné věži**; každé patro je souboj
s nepřítelem (vizuální styl inspirovaný Shakes & Fidget: velkohlavé kreslené postavy, tlusté obrysy, sytě barevné).
Správné slovíčko = energie ⚡, energie se utrácí za útoky (Seknutí, Ohnivá koule, Ultimátní úder, Léčení).
Za výhry jsou 🪙 mince, truhla a občas kořist (vybavení). Vybavení (zbraně, oblečení, šperky) se kupuje v obchodě,
spravuje v profilu/inventáři a je vidět na postavě. Uživatel si na začátku zvolí pokročilost A1–C1; vyšší úroveň = víc mincí.

## Technické konvence
- Jediný soubor **`index.html`** (vanilla HTML/CSS/JS, bez buildu, bez závislostí) – nutné pro GitHub Pages.
- UI i komentáře česky; cvičení = překlad CZ → EN.
- Spuštění: otevřít `index.html` v prohlížeči.
- Postup hráče se ukládá do `localStorage` (klíč `anglictina-rpg-save-v2`; při změně struktury zvýšit verzi
  a řešit migraci v `loadSave`). Pozor: v `data:`/preview kontextu je storage zakázané – vše je v try/catch.

## Architektura `index.html`
- Obrazovky: `s-start` → `s-map` (věž, `renderTower`) ⇄ `s-profile` (`renderProfile`) ⇄ `s-shop` (`renderShop`) → `s-battle` → `s-result`; přepíná `showScreen(id)`.
- Data:
  - `LEVELS[úroveň]` – `{label, coins, hp, dmg, zones[]}`; zóna = `{theme, icon, boss:{name,skin,horns}, words:["česky=anglicky",…]}`.
    Zóna trvá `ZONE_LEN` (10) pater, 10. patro zóny je boss; po vyčerpání zón se cyklí (boss dostane číslo II, III…).
  - `ATTACKS`, `ITEMS` (slot weapon/armor/jewelry, rarita green/blue/yellow, bonusy `atk` %dmg, `hp`, `en` %energie, `coin` %mincí), `DROP` (šance kořisti).
  - `TYPING_LEVELS` – úrovně, kde se odpověď vždy píše.
- Věž: `getEnemy(level, patro)` – nepřítel je deterministický podle (úroveň, patro) díky `rngFor`, takže náhled a souboj sedí.
  Škálování: HP +12 %/patro, dmg +6 %/patro; boss zóny ×2,2 HP, ×1,4 dmg. `getDifficulty` – časový limit (od 6. patra),
  šance na psaní odpovědi (od 9. patra), frekvence útoků bosse bez chyby.
- Vzhled nepřátel: `makeLook(rng)` náhodně skládá tvar hlavy, počet očí, tlamu, rohy, uši, doplněk a barvy; `enemySVG(e)` je vykreslí.
- Vzhled hrdiny: `heroSVG()` skládá `ARMOR_ART[id]` (back/body/front/top), `WEAPON_ART[id]`, `JEWEL_ART[id]` – každý kus
  vybavení má vlastní SVG. **Nové vybavení = položka v `ITEMS` + její SVG v `*_ART` se stejným `id`.**
- Odměny (`endBattle`): základ `LEVELS[l].coins * (1 + 0,1*(patro-1)) * (1 + bonus mincí)`, ×1,2 za bezchybný souboj, porážka = 25 %.
  Truhla (po výhře): `coins * (1,5 + 0,2*patro)` náhodně ±, boss ×3 (připsáno hned, klik ji jen "otevře").
  Kořist (`rollDrop`): 35 % (boss 100 %), rarita váhami green/blue/yellow – čím lepší, tím vzácnější; duplikát se prodá za 50 % ceny.
- Stav: `save` (jméno, úroveň, xp, coins, `tower[úroveň]`, `best`, `owned`, `equip`), `battle` (aktuální souboj).
- Klíčové funkce: `startBattle`, `nextQuestion`, `answer`, `bossAttack`, `useAttack`, `endBattle`, `updateBars`, `totals()`, `itemCard`, `equipAction`.

## Jak rozšiřovat
- **Slovíčka:** přidat `"česky=anglicky"` do `words` zóny. **Zóna/boss:** přidat objekt do `LEVELS[...].zones`.
- **Útok:** `ATTACKS`. **Vybavení:** viz výše (ITEMS + SVG).
- Po změnách ověřit v prohlížeči celý průběh: start → věž → souboj → výhra/prohra → truhla → profil → obchod → reload.

## Nápady na rozvoj
Zvuky, více typů cvičení (poslech, věty), speciální schopnosti vybavení, další sloty, denní série,
statistika slabých slovíček a jejich opakování, žebříček, sady vybavení s bonusem.
