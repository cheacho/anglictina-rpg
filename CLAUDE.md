# CLAUDE.md

## Projekt
**Angličtina RPG** – interaktivní hra na učení angličtiny. Hráč leze po **nekonečné věži**; každé patro je souboj
s bossem (vizuální styl inspirovaný Shakes & Fidget: velkohlavé kreslené postavy, tlusté obrysy, sytě barevné).
Správné slovíčko = energie ⚡, energie se utrácí za útoky (Seknutí, Ohnivá koule, Ultimátní úder, Léčení).
Za souboje se dostávají 🪙 mince, za které se v obchodě kupuje vybavení (zbraně, oblečení, šperky).
Uživatel si na začátku zvolí pokročilost A1–C1; vyšší úroveň dává víc mincí.

## Technické konvence
- Jediný soubor **`index.html`** (vanilla HTML/CSS/JS, bez buildu, bez závislostí) – nutné pro GitHub Pages.
- UI i komentáře česky; cvičení = překlad CZ → EN.
- Spuštění: otevřít `index.html` v prohlížeči.
- Postup hráče se ukládá do `localStorage` (klíč `anglictina-rpg-save-v2`; při změně struktury zvýšit verzi
  a řešit migraci v `loadSave`). Pozor: v `data:`/preview kontextu je storage zakázané – vše je v try/catch.

## Architektura `index.html`
- Obrazovky: `s-start` (jméno + úroveň) → `s-map` (věž, `renderTower`) ⇄ `s-shop` (`renderShop`) → `s-battle` → `s-result`; přepíná `showScreen(id)`.
- Data:
  - `LEVELS[úroveň]` – `{label, coins, bosses[]}`; boss = `{name, theme, skin, horns, hp, dmg, words:["česky=anglicky",…]}`. `coins` = základ odměny.
  - `ATTACKS` – cena energie, poškození / léčení.
  - `ITEMS` – vybavení: `slot` (weapon/armor/jewelry), `rar` (green/blue/yellow), `price` a bonusy `atk` (% dmg), `hp`, `en` (% energie), `coin` (% mincí).
  - `TYPING_LEVELS` – úrovně, kde se odpověď vždy píše.
- Věž: `getBoss(level, patro)` cykluje přes bosse úrovně a škáluje HP (+12 %/patro) a dmg (+6 %/patro);
  každé 10. patro je mega boss (×1,6 HP, ×1,3 dmg, koruna). `getDifficulty(patro)` určuje časový limit (od 6. patra),
  šanci na psaní odpovědi (od 9. patra) a frekvenci útoků bosse bez chyby.
- Mince: výhra = `LEVELS[l].coins * (1 + 0,1*(patro-1)) * (1 + bonus mincí)`, ×1,2 za bezchybný souboj; porážka = 25 %.
- Stav: `save` (perzistentní: jméno, úroveň, xp, coins, `tower[úroveň]`, `best`, `owned`, `equip`), `battle` (aktuální souboj).
- Klíčové funkce: `startBattle`, `nextQuestion`, `answer`, `bossAttack`, `useAttack`, `endBattle`, `updateBars`, `totals()` (součet bonusů vybavení).
- Grafika: `heroSVG()` (barva čepele/tuniky/drahokamu podle rarity vybavení), `bossSVG(boss)` – inline SVG; efekty `floatText`, `pow`, `hitFx`, `lunge`.

## Jak rozšiřovat
- **Slovíčka:** přidat řetězec `"česky=anglicky"` do `words` bosse (min. 4 slova kvůli možnostem).
- **Boss:** přidat objekt do `LEVELS[...].bosses` (patra se cyklují přes celý seznam).
- **Útok:** přidat do `ATTACKS`. **Vybavení:** přidat do `ITEMS` (obchod se generuje automaticky).
- Po změnách vždy ověřit v prohlížeči celý průběh: start → věž → souboj → výhra/prohra → obchod → reload.

## Nápady na rozvoj
Zvuky, více typů cvičení (poslech, věty, nepravidelná slovesa), speciální schopnosti vybavení, další sloty,
denní série, statistika slabých slovíček a jejich opakování, žebříček nejvyššího patra.
