Program v plc, uz si spominam par logik, program je rozdelený na bloky a signály majú mená — tzv. tagy: Tlacidlo_Start, Safety_OK, Motor_Bezi. Logika je: ak platí Tlačidlo ŠTART a zároveň Safety OK a zároveň Auto režim → zapni motor. primitivne ale co ked je toho mnoho?? 
Reálny program vo fabrike má stovky blokov a tisíce tagov
A teraz ten problém. 
Keď sa motor nerozbehne, neikde vtom reťazci jedna podmienka neplatí — ale ktorá? Technik ju hľadá ručne: otvorí blok, nájde, že chýba Cmd_Start, preklikne sa do ďalšieho bloku, kde zistí, že chýba Safety_OK, preklikne do safety bloku... Pri poruche na linke to môže trvať hodinu, a linka medzitým stojí — čo v priemysle znamená veľké peniaze každú minútu.
Naša appka robí presne toto hľadanie za teba. Zoberie program (exportovaný z TIA ako XML), zostaví z neho mapu „kto ovplyvňuje koho", a na otázku „prečo nejde motor?" vypľuje celý strom príčin naraz: motor ← povel štart ← safety OK ← núdzové tlačidlo na vstupe. Namiesto hodiny preklikávania jeden pohľad. To je celá pointa — nič viac, nič menej



...................... A FEW MOMENTS LATER :D........


algoritmicky je to jednoduché, ťažké je to znalostne?? 
uvidim. 1hod.

Kód má tri časti s úplne rôznou náročnosťou:

1. Parser (asi60 % kódu) — drina, ziadny skill. tss ze algo,skor len skriptik. len prechádzanie XML stromu a prekladanie Siemens formátu do našich objektov. Ťažké na tom je vedieť, čo tie elementy znamenajú: že Coil s portom operand je zápis, ale Contact s tým istým portom je čítanie; diely co??? wtf ake diely a opat studovanie plc a detailny na nete, ...
2.
3.2 hod.
4.  Wire spája dielytakze cez UId čísla; že lokálna premenná sa musí kvalifikovať blokom. To nie je computer science, to je znalosť domény — presne preto to konkurencia nemá a presne preto máš ako automatizér výhodu oproti čistému programátorovi.
5. Xref index — triviálny. Doslova slovník: writesOf["Cmd_Start"] = [network 5, network 12]. Zostavenie je O(n), jeden prechod cez všetky networky. Vyhľadanie O(1).
6. Trasovanie — jediný skutočný algoritmus, a je učebnicový. Je to rekurzívne prehľadávanie grafu do hĺbky (DFS), ktoré poznáš zo školy. Celé jadro je zhruba toto:
funkcia trasuj(tag):
    pre každý network, ktorý tag ZAPISUJE:
        pre každý tag, ktorý ten network ČÍTA:
            trasuj(tag)   # rekurzia
Dve poistky navyše: množina visited, aby sa to nezacyklilo (tag A závisí od B a B od A — v PLC bežné, napr. samodrž motora), a limit hĺbky ako druhá brzda. Zložitosť: každý network navštíviš najviac raz na vetvu, takže prakticky O(počet networkov × priemerný počet tagov v networku) — aj projekt s 10 000 networkami prebehne v prehliadači za zlomok sekundy, lebo je to len skákanie po slovníku v pamäti.
Takže ak by si to mal niekomu zhrnúť: „skript s jedným klasickým grafovým algoritmom, kde 80 % práce bolo reverzné inžinierstvo Siemens XML formátu." Žiadne ML, žiadna veda — hodnota je v doménovej znalosti zakódovanej do parsera.
Mimochodom, DFS s visited setom je tá istá myšlienka, akou sa prehľadáva bludisko alebo závislosti balíčkov v npm — ak chceš, na demo dátach ti krok po kroku ukážem, ako presne rekurzia „šliape" od Q_Motor_Contactor dozadu, nech ten algoritmus vidíš naživo.

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
