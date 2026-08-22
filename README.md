# ⛳ HCPheute

Spielerunabhängiges Vergleichs-Handicap nach WHS Rule 5.2a (und 5.1b bei
9-Loch-Runden).

🔗 **[https://hyper472.github.io/HCPheute/](https://hyper472.github.io/HCPheute/)**

---

## Idee

HCPheute berechnet aus einer einzelnen Golfrunde ein Vergleichs-Handicap —
unabhängig davon, ob der Spieler schon ein offizielles Handicap hat. Dazu
nutzt die App bewusst die WHS-Erst-Einstufungs-Logik (Rule 5.2a) statt der
normalen Expected-Score-Logik für Spieler mit bestehendem Handicap (Rule
5.1b). So bekommen zwei Spieler mit identischer Tagesform auf demselben Platz
denselben Wert — echte Vergleichbarkeit statt eines persönlichen,
handicap-abhängigen Ergebnisses.

Die Berechnung geht dabei immer von 8 angenommenen, identischen Runden aus
(fest, nicht einstellbar) — so, als wäre der Spieler vor 8 Tagen mit der
Platzreife auf die Welt gekommen und hätte seitdem jeden Tag exakt dasselbe
Ergebnis auf demselben Platz gespielt.

**9-Loch-Runden:** Standardmäßig wird die ungespielte Neun wie bei 18 Loch
als identisch angenommen (spielerunabhängig, gleiches Prinzip wie beim
Rest der App) — das HCPI-Feld bleibt dafür einfach leer. Trägst du dort
dein aktuelles HCPI ein, wechselt die Berechnung bewusst auf die
offizielle WHS-Formel (Rule 5.1b): dann wird die ungespielte Neun anhand
deines bestehenden Handicaps geschätzt statt verdoppelt — das Ergebnis
hängt dann nicht mehr nur von der gespielten Runde ab, sondern auch vom
eigenen Handicap (bewusste Abweichung von der sonstigen
Spielerunabhängigkeit, nur auf ausdrücklichen Wunsch).

**Wichtig:** HCPheute ersetzt kein offizielles WHS-Handicap. Es ist ein
Vergleichswert für eine einzelne Runde, keine laufende, amtliche Einstufung.

---

## Platz anlegen

1. Tab **Plätze** → **+ Platz hinzufügen**
2. Name und Löcher (9/18) eintragen
3. Für jeden Abschlag (z. B. Gelb, Rot): Name, Geschlecht, Par sowie Course
   Rating und Slope Rating eintragen. Ein Abschlag ist eine einzelne
   Tee-Markierung für ein Geschlecht — spielen Herren und Damen
   unterschiedliche Markierungen (Regelfall), für jede einen eigenen
   Abschlag anlegen.
4. Mit **+ Abschlag hinzufügen** weitere Abschläge desselben Platzes ergänzen
5. **Speichern**

Course Rating, Slope Rating und Par pro Abschlag stehen auf der Scorecard
oder der Spielvorgabentabelle des Platzes (oft "Course Handicap"-Tabelle
genannt) — unterschiedliche Abschläge haben unterschiedliche Werte, teils
sogar unterschiedliches Par.

---

## Berechnung durchführen

1. Tab **Berechnung** → Platz wählen
2. Abschlag wählen (zeigt Geschlecht sowie die zugehörigen CR/Slope-Werte)
3. Gewertetes Bruttoergebnis eintragen (nach WHS-Deckelung: pro Loch
   höchstens Netto-Doppelbogey — im Scoring Record als "GBE" ausgewiesen)
4. Bei 9-Loch-Plätzen optional: aktuelles HCPI eintragen, wenn die
   regelkonforme, handicap-abhängige Schätzung statt der spielerunabhängigen
   Verdopplung gewünscht ist (leer lassen = spielerunabhängig)
5. **Berechnen** — der komplette Rechenweg wird angezeigt

---

## Platz bearbeiten oder löschen

In der Platzliste unter jedem Eintrag → **✏ Bearbeiten** oder **✕ Löschen**
(mit Sicherheitsabfrage).

---

## Daten

Alle Plätze werden zentral gespeichert – auf jedem Gerät derselbe Stand,
ohne Anmeldung. Ältere Plätze in einem der beiden Vorgänger-Formate werden
beim ersten Laden automatisch ins aktuelle Abschlag-Format übernommen —
danach in der Platzverwaltung ggf. selbst umbenennen oder ergänzen.
