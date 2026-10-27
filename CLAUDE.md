# CLAUDE.md

## Projekt
**Angličtina RPG** – interaktivní hra na učení angličtiny. Každé cvičení je souboj s bossem
(vizuální styl inspirovaný Shakes & Fidget: velkohlavé kreslené postavy, tlusté obrysy, sytě barevné).
Správné slovíčko = energie ⚡, energie se utrácí za útoky (Seknutí, Ohnivá koule, Ultimátní úder, Léčení).
Uživatel si na začátku zvolí pokročilost A1–C1.

## Technické konvence
- Jediný soubor **`index.html`** (vanilla HTML/CSS/JS, bez buildu, bez závislostí) – nutné pro GitHub Pages.
- UI i komentáře česky; cvičení = překlad CZ → EN.
- Spuštění: otevřít `index.html` v prohlížeči.
- Postup hráče se ukládá do `localStorage` (klíč `anglictina-rpg-save-v1`; při změně struktury zvýšit verzi).

## Architektura `index.html`
- `<style>`: CSS proměnné v `:root`, obrazovky `.screen`, arena a animace (`shake`, `floatup`, `pow`).
- Obrazovky: `s-start` (jméno + úroveň) → `s-map` (výběr bosse) → `s-battle` → `s-result`; přepíná `showScreen(id)`.
- Data:
  - `LEVELS[úroveň].bosses[]` – `{name, theme, skin, horns, hp, dmg, words:["česky=anglicky",…]}`.
  - `ATTACKS` – cena, poškození / léčení.
  - `TYPING_LEVELS` – úrovně, kde se odpověď píše (jinak výběr ze 4 možností).
- Stav: `save` (perzistentní: jméno, úroveň, xp, poražení bossové `beaten["A1:0"]`), `battle` (aktuální souboj).
- Klíčové funkce: `startBattle`, `nextQuestion`, `answer`, `bossAttack`, `useAttack`, `endBattle`, `updateBars`.
- Grafika: `heroSVG()`, `bossSVG(boss)` – inline SVG; efekty `floatText`, `pow`, `hitFx`, `lunge`.

## Jak rozšiřovat
- **Slovíčka:** přidat řetězec `"česky=anglicky"` do `words` bosse (min. 4 slova kvůli možnostem).
- **Boss:** přidat objekt do `LEVELS[...].bosses` (barva `skin`, `horns`, `hp`, `dmg`).
- **Útok:** přidat do `ATTACKS` (tlačítka se generují automaticky).
- Po změnách vždy ověřit v prohlížeči celý průběh: start → mapa → souboj → výhra/prohra → reload.

## Nápady na rozvoj
Zvuky, více typů cvičení (poslech, věty, nepravidelná slovesa), obchod s vybavením, vylepšování
postavy, denní série, více bossů/úrovní, statistika slabých slovíček, ukládání chybných slov k opakování.
