# TIA XRef

Trasovanie signálov a cross-reference nad exportmi z TIA Portalu. Jeden HTML súbor, všetko beží v prehliadači — kód z projektu nikam neodchádza.

## Ako dostať XML z TIA Portalu

- V projekte: pravý klik na blok (OB/FB/FC) → **Export block** / cez Version Control Interface (V17+), alebo Openness export → vznikne SimaticML `.xml`.
- Know-how protected bloky sa exportovať nedajú.
- Do appky môžeš hodiť viac súborov naraz — spoja sa do jedného modelu.

## Čo appka vie

- **Trasovanie**: upstream („čo všetko ovplyvňuje tento tag") a downstream („kam signál pokračuje"), rekurzívne naprieč blokmi, s detekciou cyklov.
- **Graf**: závislosti ako uzly a šípky (Cytoscape), klik na uzol = nové trasovanie.
- **Cross-reference**: každý tag → kde sa zapisuje / číta.
- **Networky**: prehľad naparsovaných networkov vrátane CALL, GOTO a labels v SCL.
- **Export JSON**: celý model na ďalšie spracovanie (SQLite, skripty…).

## Známe obmedzenia (MVP)

- Schéma SimaticML sa líši medzi verziami TIA (V16/V17/V18/V19) — testované na štruktúre V17. Ak niečo nenaparsuje, pošli mi vzorku XML.
- STL (AWL) bloky zatiaľ nie sú podporované, GRAPH tiež nie.
- Zápis cez pointer/ANY/nepriamu adresáciu parser nevidí (to nevidí poriadne ani TIA).
- Volania blokov sa zobrazujú, ale parametre volania sa zatiaľ neprepájajú s interface volaného bloku (formal ↔ actual mapping — ďalší krok).
