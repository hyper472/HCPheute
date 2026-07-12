# CONTEXT – HCPheute

## Zweck
Private Web-App zur Berechnung eines spielerunabhängigen Vergleichs-Handicaps
nach WHS Rule 5.2a (Erst-Einstufungs-Logik). Gehostet auf GitHub Pages
(`hyper472.github.io/HCPheute/`), Daten in JSONBin.io.

Vorbilder: technischer Aufbau und JSONBin-Mechanismus nach [Tanklog](../Tanklog/),
Design (Farben, PWA-Meta-Tags) nach [Braurechner](../Braurechner.zip).

---

## Technischer Aufbau
**Eine einzige Datei:** `index.html` – kein Build-Schritt, kein Framework, kein Node.

Herren und Damen sind im Datenmodell und in der UI bewusst gleichrangig
behandelt (Entscheidung vom 2026-07-12): pro Abschlag ist mindestens ein
vollständiger Wertesatz (Herren oder Damen oder beide) erforderlich, keines
von beiden ist per Default gesetzt oder als "optional" markiert. Welche(s)
Geschlecht(er) tatsächlich zur Auswahl steht, ergibt sich in der
Berechnungs-UI rein daraus, was am Abschlag hinterlegt wurde
(`getTeeGenders()`/`resolveGender()` in `index.html`).

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
  MASTER_KEY: '$2a$10$Xodn...',   // JSONBin Master Key – gleicher Account wie Tanklog
  BIN_ID:     '6a52ba8f...',      // eigener Bin für HCPheute
};
```

**Achtung:** Der Master Key ist Account-weit, nicht Bin-spezifisch. Er wird
bewusst mit Tanklog geteilt (gleicher JSONBin-Account) – wer den öffentlichen
Quelltext eines der beiden Repos liest, hat damit auch Zugriff auf den Bin
des jeweils anderen Projekts. Für zwei private Freizeit-Apps akzeptiertes
Risiko (Entscheidung vom 2026-07-11).

### Storage-Abstraktion
Identisch zu Tanklog:
1. **JSONBin** – wenn `CONFIG.MASTER_KEY` gesetzt (GitHub Pages, Normalfall)
2. **`window.storage`** – wenn in Claude-Artifact-Umgebung ausgeführt
3. **`localStorage`** – Fallback (kein Sharing zwischen Geräten)

### Datenstruktur im Bin
Ein Platz hat einen oder mehrere Abschläge (`tees`); jeder Abschlag hat
eigenes Par sowie CR/Slope für Herren und/oder Damen (mindestens einer von
beiden erforderlich, keines von beiden per Default):
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
          "par": 72,
          "herren": { "courseRating": 72.6, "slopeRating": 130 },
          "damen": { "courseRating": 76.9, "slopeRating": 133 }
        }
      ],
      "createdAt": "ISO8601"
    }
  ]
}
```

`herren` bzw. `damen` ist `null`, wenn für diesen Abschlag keine Werte für
das jeweilige Geschlecht erfasst wurden. Ist nur eines von beiden gesetzt,
verwendet die Berechnung automatisch dieses eine und die Geschlecht-Auswahl
wird in der UI ausgeblendet; sind beide gesetzt, erscheint die Auswahl.

**Migration:** Platz-Datensätze im alten, flachen Format (ein
CR/Slope/Par direkt am Platz, kein `tees`-Array — Stand vor 2026-07-12)
werden beim Laden automatisch von `migrateLegacyCourse()` in einen
einzelnen Abschlag namens "Standard" (nur Herren-Werte) überführt und
zurückgespeichert. Betraf beim Umstieg genau einen echten Bestandsplatz
("Golfrange Nürnberg").

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

Bei 18-Loch-Runden hat das HCPI-Feld keine Wirkung (UI blendet es aus) —
Rule 5.1a ist unter WHS handicap-unabhängig, das bildet die App 1:1 nach.

Quelle der 0.52/1.2-Formel: WHS Rules of Handicapping Rule 5.1b, über zwei
unabhängige Sekundärquellen bestätigt (Reverse-Engineering gegen
USGA-Beispiele durch ein nationales Handicap-Committee); der
USGA-Originaltext war zum Zeitpunkt der Umsetzung nicht automatisiert
abrufbar (403) — bei Zweifel gegen eine Primärquelle nachprüfen.

---

## Features
- Platzverwaltung: Plätze mit mehreren Abschlägen (Herren/Damen)
  anlegen/bearbeiten/löschen (Löschen mit Sicherheitsabfrage)
- Berechnung: Platz → Abschlag → ggf. Geschlecht → Schlägeanzahl (→ bei
  9-Loch optional aktuelles HCPI, s. o.) eingeben, Ergebnis mit sichtbarem
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
- Master Key liegt im öffentlichen HTML-Quelltext (Repo ist public) –
  bewusst akzeptiertes Risiko, s. o.
- `robots.txt` schützt nur wohlerzogene Crawler; kein technischer
  Zugriffsschutz
- Noch kein GitHub-Repository/Deployment eingerichtet (lokaler Stand,
  Stand: 2026-07-11)
