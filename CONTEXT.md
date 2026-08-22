# CONTEXT – HCPheute

**Stand: v1.0.1 (zementiert 2026-08-09)** — Herren/Damen-Abschläge, feste
Turnierzahl 8, Rule 5.1b als Opt-in bei 9-Loch-Runden, gegen zwölf reale
DGV-Produktivrunden validiert (s. u.), Access-Key-Setup, live auf GitHub
Pages.

## Zweck
Private Web-App zur Berechnung eines spielerunabhängigen Vergleichs-Handicaps
nach WHS Rule 5.2a (Erst-Einstufungs-Logik). Gehostet auf GitHub Pages
(`hyper472.github.io/HCPheute/`), Daten in JSONBin.io.

Vorbilder: technischer Aufbau und JSONBin-Mechanismus nach [Tanklog](../Tanklog/),
Design (Farben, PWA-Meta-Tags) nach [Braurechner](../Braurechner.zip).

---

## Technischer Aufbau
**Eine einzige Datei:** `index.html` – kein Build-Schritt, kein Framework, kein Node.

**Ein Abschlag = eine Tee-Markierung für ein Geschlecht (Entscheidung vom
2026-08-22, ersetzt den Stand vom 2026-07-12):** Zunächst konnte ein Abschlag
Herren- und Damen-Wertesatz gleichzeitig tragen (gedacht für den Fall, dass
beide dieselbe Markierung nutzen). In der echten Nutzung stellte sich das als
reine Verwirrungsquelle heraus — auf dem einzigen bisher genutzten Platz
(GolfRange Nürnberg) spielen Herren und Damen von zwei verschieden benannten
Markierungen (Gelb/Rot), und das Formular fragte trotzdem bei jedem Abschlag
beide Wertesätze ab. Ergebnis war ein produktiver Datensatz mit vier
Fragment-Plätzen statt zwei sauberen. Auf ausdrücklichen Wunsch ("lieber eine
Lücke in der App als die ständige Verwirrung") entfernt: ein Abschlag hat
jetzt genau ein Geschlecht (`gender: 'herren' | 'damen'`) und einen CR/Slope.
Courses mit unterschiedlichen Markierungen pro Geschlecht bekommen zwei
Abschlag-Einträge.

### Abhängigkeiten
Keine externen Bibliotheken (kein CDN) – nur Systemschriften (Georgia,
-apple-system, Menlo) und die JSONBin.io REST API v3.

### Farbschema (Muster: Braurechner)
| Variable | Wert | Bedeutung |
|----------|------|-----------|
| `--bg` | `#f4f6fa` | Seitenhintergrund (helles Grau) |
| `--surface` | `#ffffff` | Karten-Hintergrund |
| `--surface2` | `#eaf0f8` | Eingabefelder, Ergebnisbox |
| `--blue` | `#1a4a9e` | Primärfarbe (Header, Titel, Ergebniswert) |
| `--blue2` | `#2a5abf` | Blau hell (Verläufe, Fokus-Rahmen) |
| `--orange` | `#e07010` | Akzentfarbe (Buttons, aktiver Tab) |
| `--orange2` | `#f08020` | Akzentfarbe hell (aktiver Button-Zustand) |
| `--text` | `#111820` | Haupttext |
| `--muted` | `#506080` | Labels, Hinweistexte |
| `--red` | `#c0392b` | Löschen, Fehlermeldungen |
| `--green` | `#2a6a3a` | Erfolgsmeldungen |

### Schriften
- **Georgia** – Logo, Kartentitel, Ergebniswert, Modal-Titel
- **-apple-system / Helvetica Neue** – Fließtext, Labels, Hinweistexte
- **Menlo** – Zahleneingaben, Rechenweg, Metadaten (Löcher/CR/Slope)

---

## Datenspeicher (JSONBin.io)

### Konfiguration (oben in `<script>`)
```js
const CONFIG = {
  ACCESS_KEY: '$2a$10$KovQ...',   // JSONBin Access Key – read+update, bin-spezifisch
  BIN_ID:     '6a52ba8f...',      // fest eingetragen, keine Auto-Erstellung mehr möglich
};
```

**Historie:** Ursprünglich (bis 2026-07) ein Account-weiter Master Key, mit
Tanklog geteilt – dieser wurde öffentlich im Repo sichtbar, irgendwann
rotiert, und HCPheute war dadurch offline. Seit 2026-08 stattdessen ein auf
Read+Update beschränkter, bin-spezifischer Access Key: kann weder neue Bins
anlegen noch löschen, hat keinen Zugriff auf den Tanklog-Bin. Bleibt zwar
weiterhin im öffentlichen HTML-Quelltext sichtbar (clientseitiges JS lässt
sich nicht anders bauen), der Schaden bei erneutem Leak ist aber auf diesen
einen Bin begrenzt statt auf den ganzen Account.

### Storage-Abstraktion
Identisch zu Tanklog:
1. **JSONBin** – wenn `CONFIG.ACCESS_KEY` gesetzt (GitHub Pages, Normalfall)
2. **`window.storage`** – wenn in Claude-Artifact-Umgebung ausgeführt
3. **`localStorage`** – Fallback (kein Sharing zwischen Geräten)

### Datenstruktur im Bin
Ein Platz hat einen oder mehrere Abschläge (`tees`); jeder Abschlag ist eine
einzelne Tee-Markierung für ein Geschlecht, mit eigenem Par, CR und Slope:
```json
{
  "courses": [
    {
      "id": "course_...",
      "name": "Golfclub Musterstadt",
      "holes": 18,
      "tees": [
        {
          "id": "tee_...",
          "name": "Gelb",
          "gender": "herren",
          "par": 72,
          "courseRating": 72.6,
          "slopeRating": 130
        },
        {
          "id": "tee_...",
          "name": "Rot",
          "gender": "damen",
          "par": 72,
          "courseRating": 76.9,
          "slopeRating": 133
        }
      ],
      "createdAt": "ISO8601"
    }
  ]
}
```

Spielen beide Geschlechter von derselben Markierung, einfach zwei Abschläge
mit identischem `name`, aber unterschiedlichem `gender` anlegen — das
Datenmodell erzwingt keine Eindeutigkeit des Namens.

**Migration (`normalizeCourse()`), zwei Alt-Formate:**
1. Ganz altes Format ohne `tees`-Array (ein CR/Slope/Par direkt am Platz,
   Stand vor 2026-07-12) → ein Abschlag "Standard", Geschlecht Herren.
2. Zwischenformat mit `tees`-Array, aber `herren`/`damen` als zwei optionale
   Wertesätze *in einem* Abschlag (Stand 2026-07-12 bis 2026-08-22) → pro
   erfasstem Geschlecht ein eigener, flacher Abschlag. War an einem Abschlag
   ausnahmsweise beides erfasst, werden daraus zwei Abschläge mit demselben
   Namen plus Geschlecht in Klammern angehängt.

Beide Migrationen laufen automatisch beim Laden und schreiben das Ergebnis
zurück. Betroffen beim jeweiligen Umstieg: zunächst ein echter Bestandsplatz
("Golfrange Nürnberg", 2026-07-12), später vier Fragment-Plätze aus
Zwischenformat-Verwirrung (2026-08-22, s. o.).

Kein Speichern einzelner Berechnungsergebnisse (bewusste MVP-Entscheidung,
s. u.) – nur die Platzliste ist persistent.

---

## Kernfunktion: `calculate()`

Implementiert die WHS Rule 5.2a Erst-Einstufungs-Tabelle (Tabelle `table`)
plus `roundHalfUp()` (WHS rundet auf 1 Nachkommastelle, 0,5 wird
aufgerundet – nicht das alte USGA-Trunkierungsverfahren vor 2020).

Die Begründung, warum grundsätzlich Rule 5.2a (Erst-Einstufung) statt Rule
5.1b (Expected-Score-Logik für bestehendes Handicap) verwendet wird, steht
als Kommentar direkt über der Funktion in `index.html`. Die Turnierzahl ist
fest auf 8 gesetzt (`TOURNAMENT_COUNT`, nicht einstellbar) — Idee: so tun,
als wäre der Spieler vor 8 Tagen mit der Platzreife auf die Welt gekommen
und hätte seitdem jeden Tag exakt dasselbe Ergebnis gespielt.

**9-Loch-Runden — zwei Anläufe, ein Fazit (Entscheidungen vom 2026-07-12):**
Erst wurde die ungespielte Neun grundsätzlich per Rule 5.1b geschätzt
(Default HCPI 54 ohne Eingabe, später sogar Pflichtfeld). Das erwies sich
in der Praxis als genau der Bias, den die App mit Rule 5.2a eigentlich
vermeiden soll: zwei Spieler mit identischer Tagesform bekommen
unterschiedliche Ergebnisse, weil nur die Schätzung für die ungespielte
Neun vom bestehenden Handicap abhängt (die tatsächlich gespielte Neun wird
für beide gleich bewertet). Endstand:
- **Kein HCPI eingetragen (Default):** ungespielte Neun wird wie bei
  18-Loch-Runden als identisch angenommen — `diff18 = roundHalfUp(diffRaw * 2)`,
  spielerunabhängig, gleiches Prinzip wie der Rest der App.
- **HCPI eingetragen:** wechselt bewusst auf die offizielle, aber
  handicap-abhängige WHS-Formel (Rule 5.1b): "Expected Score Differential"
  für die ungespielte Neun = `expectedScoreDifferential(hcpi) = 0.52 × hcpi + 1.2`
  (unrundete Werte, erst die Summe mit dem gespielten Differenzial wird
  gerundet).

**Empirischer Beleg für diese Entscheidung (Daten vom 2026-08-16):** Der Bias ist
an echten Daten messbar, ohne dass es dafür zwei Spieler braucht. Derselbe
Spieler hat auf demselben Abschlag (GolfRange Nürnberg, gelb, 9 Loch, Par 34,
CR 33,3, Slope 124) dreimal exakt 55 Schläge gespielt — bei deutlich
unterschiedlichem Handicap-Index. Die gespielte Leistung ist damit als Konstante
kontrolliert, der Index bleibt die einzige Variable:

| Datum | HCPI | Expected Score | SD nach Rule 5.1b | HCPheute (Default) |
|---|---|---|---|---|
| 04.05.2025 | 44,3 | 24,236 | 19,7750 + 24,236 = **44,0** | 39,6 |
| 14.06.2025 | 43,7 | 23,924 | 19,7750 + 23,924 = **43,7** | 39,6 |
| 14.08.2026 | 36,4 | 20,128 | 19,7750 + 20,128 = **39,9** | 39,6 |

Identische Leistung, Spanne von **4,1 Schlägen** im offiziellen Wert — allein
durch den gesunkenen Index. HCPheute liefert dreimal denselben Wert, weil die
ungespielte Neun mit der gespielten gleichgesetzt wird
(`roundHalfUp(19,775) = 19,8`, verdoppelt 39,6).

Praktische Folge über den App-Zweck hinaus: 9-Loch-Score-Differenziale sind im
Scoring Record über längere Zeiträume **nicht** vergleichbar. Wer daraus einen
Formverlauf ablesen will, muss die 9-Loch-Runden ausklammern oder auf
spielerunabhängige Werte umstellen.

**Datenherkunft:** Detaillierter Scoring-Record-Export des DGV vom 16.08.2026
(Spieler HB). Der Wert für den 14.08.2026 ist direkt abgelesen (`ExSc: 0`,
keine Anpassung). Die beiden Werte von 2025 sind aus den angezeigten 42,0 bzw.
41,7 zurückgerechnet: Beide Runden liegen vor den Ausnahme-Ereignissen vom
21.06.2025 und 08.08.2026 und tragen daher je zwei Anpassungen von −1.

Bei 18-Loch-Runden hat das HCPI-Feld keine Wirkung (UI blendet es aus) —
Rule 5.1a ist unter WHS handicap-unabhängig, das bildet die App 1:1 nach.

Quelle der 0.52/1.2-Formel: WHS Rules of Handicapping Rule 5.1b, über zwei
unabhängige Sekundärquellen bestätigt (Reverse-Engineering gegen
USGA-Beispiele durch ein nationales Handicap-Committee); der
USGA-Originaltext war zum Zeitpunkt der Umsetzung nicht automatisiert
abrufbar (403) — bei Zweifel gegen eine Primärquelle nachprüfen.

### Validierung gegen DGV-Produktivdaten (2026-08-09)

Die Berechnung wurde gegen zwölf reale Runden aus dem detaillierten
Scoring-Record-Export des DGV geprüft (Spieler HB, GolfRange Nürnberg, Abschlag
gelb; 18 Loch: Par 68, CR 66,6, Slope 125 — 9 Loch: Par 34, CR 33,3, Slope 124).
Alle zwölf stimmen exakt überein. Damit ist insbesondere die 0.52/1.2-Formel aus
Rule 5.1b nicht mehr nur über Sekundärquellen belegt, sondern gegen die
Produktivrechnung eines nationalen Verbandes verifiziert.

**Methodischer Hinweis:** golf.de weist den SD *inklusive* aller
Exceptional-Score-Anpassungen aus, HCPheute berechnet den Rohwert. Die
Spalte "golf.de (roh)" ist deshalb zurückgerechnet: angezeigter Wert + 1 für
Runden nach dem 21.06.2025, + 2 für ältere (zwei ExSc-Ereignisse: 21.06.2025
und 08.08.2026, beide je −1, jeweils rückwirkend auf alle Einträge der
Scoring Record).

**Zum PCC:** Der Export weist für den 08.08.2026 und den 14.06.2026 je einen
PCC von −1 aus. Nach WHS-Formel müsste er den SD erhöhen
(`SD = (113/Slope) × (GBE − CR − PCC)`, bei PCC −1 also +1 im Klammerausdruck).
Nachgerechnet trifft jedoch bei beiden Runden die Variante *ohne* PCC exakt, die
*mit* PCC verfehlt um 0,9:

| Runde | GBE | ohne PCC | mit PCC | golf.de zeigt |
|---|---|---|---|---|
| 08.08.2026 | 99 | 29,3 → 28,3 | 30,2 → 29,2 | **28,3** |
| 14.06.2026 | 109 | 38,3 → 37,3 | 39,2 → 38,2 | **37,3** |

(Zweite Spalte jeweils nach Abzug der ExSc-Anpassung von −1.)

Der ausgewiesene PCC ist im angezeigten SD also nicht enthalten — trotz
gegenteiliger Angabe in der Legende des Exports ("Score Differential inklusive
aller Anpassungen (PCC, ExSc.)"). Ursache ungeklärt; denkbar wäre, dass der PCC
dokumentiert, aber auf diese Wettspiele nicht angewendet wurde, oder ein
Anzeigefehler. Für die Validierung von HCPheute ohne Belang, da die App keinen
PCC verarbeitet — hier nur festgehalten, damit die Rechenwege oben
nachvollziehbar bleiben.

#### 18 Loch — Rule 5.1a, handicap-unabhängig

Rechenweg: `(GBE − 66,6) × 113 / 125`, gerundet nach `roundHalfUp`.

| Datum | GBE | Berechnet | golf.de (roh) | golf.de (angezeigt) |
|---|---|---|---|---|
| 08.08.2026 | 99 | 29,2896 → **29,3** | 29,3 | 28,3 |
| 14.06.2026 | 109 | 38,3296 → **38,3** | 38,3 | 37,3 |
| 07.06.2026 | 114 | 42,8496 → **42,8** | 42,8 | 41,8 |
| 13.09.2025 | 113 | 41,9456 → **41,9** | 41,9 | 40,9 |
| 21.06.2025 | 104 | 33,8096 → **33,8** | 33,8 | 31,8 |

#### 9 Loch — Rule 5.1b, Expected Score Differential

Rechenweg: `(GBE − 33,3) × 113 / 124 + (0,52 × HCPI + 1,2)`, unrundet addiert,
erst die Summe nach `roundHalfUp` gerundet — exakt wie in `calculate()`
implementiert.

| Datum | GBE | HCPI | Gespielt | Erwartet | Summe | golf.de (roh) |
|---|---|---|---|---|---|---|
| 02.08.2026 | 56 | 39,0 | 20,6863 | 21,480 | 42,1663 → **42,2** | 42,2 |
| 11.07.2026 | 52 | 39,0 | 17,0411 | 21,480 | 38,5211 → **38,5** | 38,5 |
| 18.04.2026 | 51 | 39,4 | 16,1298 | 21,688 | 37,8178 → **37,8** | 37,8 |
| 12.10.2025 | 53 | 39,5 | 17,9524 | 21,740 | 39,6924 → **39,7** | 39,7 |
| 14.06.2025 | 55 | 43,7 | 19,7750 | 23,924 | 43,6990 → **43,7** | 43,7 |
| 04.05.2025 | 55 | 44,3 | 19,7750 | 24,236 | 44,0110 → **44,0** | 44,0 |
| 19.04.2025 | 53 | 45,0 | 17,9524 | 24,600 | 42,5524 → **42,6** | 42,6 |

Bemerkenswert: Der DGV rechnet 9-Loch-Runden nach Rule 5.1b. Der in HCPheute
optionale, handicap-abhängige Modus (HCPI-Feld ausgefüllt) ist damit derjenige,
der den offiziellen Wert reproduziert. Der Default (Verdopplung) bleibt richtig
für den Zweck der App — spielerunabhängige Vergleichbarkeit —, liefert aber
bewusst nicht den amtlichen Wert.

**Nicht in die Validierung aufgenommen:** Runden vor dem 13.10.2024. Der Export
weist dort für 9-Loch-Runden bereits auf 18 Löcher hochgerechnete GBE-Werte
(109–135) neben 9-Loch-CR/Slope aus; mit welcher Bezugsgröße der DGV diese Werte
gerechnet hat, ließ sich aus dem Export nicht rekonstruieren.

---

## Features
- Platzverwaltung: Plätze mit mehreren Abschlägen (je Herren oder Damen)
  anlegen/bearbeiten/löschen (Löschen mit Sicherheitsabfrage)
- Berechnung: Platz → Abschlag (Geschlecht steckt bereits im Abschlag) →
  Schlägeanzahl (→ bei 9-Loch optional aktuelles HCPI, s. o.) eingeben, Ergebnis mit sichtbarem
  Rechenweg (Differenzial → ggf. Expected-Score/18-Loch-Äquivalent →
  Tabellenzeile → Endwert)
- PWA: `apple-mobile-web-app-capable` + Apple-Touch-Icon (Muster:
  Braurechner) sowie zusätzliches `manifest.json` für breitere Kompatibilität
- Crawler-Sperre: Meta-Tags (noindex, noai) + `robots.txt` (Muster: Tanklog)
- Responsive, mobile-first (primär für iPhone-Nutzung)
- Bottom-Sheet-Modals, Toast-Benachrichtigungen (Muster: Tanklog)

---

## Bewusst nicht umgesetzt (MVP-Entscheidung vom 2026-07-11/12)
- **Keine Historie gespielter Runden.** Jede Berechnung ist eine
  Momentaufnahme; es wird nichts als "gespielte Runde" gespeichert. Nur die
  Plätze selbst sind persistent. Falls später gewünscht: eigenes
  Datenmodell (z. B. `rounds`-Array im Bin) und CRUD dafür ergänzen.
- **Kein Service Worker / Offline-Cache.** App benötigt bei jedem
  Laden/Speichern eine Internetverbindung (wie Tanklog).
- **Keine einstellbare Turnierzahl mehr.** War im ersten MVP ein Eingabefeld
  (3–20), ist seit 2026-07-12 fest auf 8 gesetzt (s. o.).

---

## Dateien im Repository
| Datei | Zweck |
|-------|-------|
| `index.html` | Komplette App (HTML + CSS + JS) |
| `manifest.json` | PWA-Manifest (Name, Icons, standalone) |
| `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` | Icon-Assets (Golf-Fahne, Blau/Weiß/Orange) |
| `robots.txt` | Sperrt Suchmaschinen und KI-Crawler |
| `README.md` | Einrichtungsanleitung |
| `CONTEXT.md` | Diese Datei – technische Dokumentation |

---

## Bekannte Einschränkungen
- JSONBin Free Tier: 10.000 Requests/Monat, geteilt mit Tanklog (gleicher
  Account)
- Access Key liegt im öffentlichen HTML-Quelltext (Repo ist public) –
  auf diesen einen Bin beschränkt (kein Account-weiter Zugriff mehr, s. o.),
  Schreibzugriff auf den Bin bleibt trotzdem möglich
- `robots.txt` schützt nur wohlerzogene Crawler; kein technischer
  Zugriffsschutz
- Live auf GitHub Pages (`hyper472.github.io/HCPheute/`), Repo
  `hyper472/HCPheute` (seit 2026-08-03)
