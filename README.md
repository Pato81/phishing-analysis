# Phishing-Analyse: IONOS-Homoglyph-Kampagne

> **Blue-Team-Fallstudie** · Analyse zweier echter Phishing-Mails, die im Herbst 2026 an ein eigenes Geschäftspostfach gingen. Beide wurden erkannt, kein Link wurde geklickt.

Die Mails geben sich als **IONOS** aus und zielen auf die Zugangsdaten zum Hosting- bzw. Mailkonto. Sie zeigen eine bemerkenswerte Menge an Täuschungstechniken – von IDN-Homograph-Domains über 439 unsichtbare Füllwörter bis zu einem kompromittierten argentinischen Webserver, der in seinen Headern die IP des Angreifers verraten hat.

---

## Kernergebnisse

| | |
|---|---|
| 🎭 **Lookalike-Domains** | `iοnos.čom` und `iοnos.ćom` – griechisches Omikron im Namen, Sonderzeichen in der TLD (Punycode `xn--inos-0nd.xn--om-dma` / `-ula`) |
| 🔤 **Homoglyphen** | Kyrillische und griechische Buchstaben mitten in deutschen Wörtern |
| 👻 **Hidden Text Salting** | 4 bzw. **439** unsichtbare `<span>`s zerlegen jedes Wort in 2-Buchstaben-Stücke |
| 🔁 **Hash-Busting** | Zero-Width-Zeichen im Betreff, Zufallskommentare, 75 einzigartige Rausch-Bilder |
| ☁️ **Trusted Hosting** | Phishing-Link auf AWS S3 (Seoul) bzw. hinter einer Google-Weiterleitung |
| 🏚️ **Kompromittierte Infrastruktur** | PHP-Mailer im Webroot einer gehackten Website (PHP 7.2, EOL) |
| 🎯 **Angreifer-IP** | `45.89.60[.]94` – aus dem cPanel-Header `X-PHP-Script` |
| ✅ **`spf=pass` ≠ sicher** | Mail 2 bestand SPF, weil sie von einem legitimen (gehackten) Server kam |
| 🧬 **Gleiches Kit** | Identischer Style-String, Message-ID-Format und Punycode-Schema trotz komplett wechselnder Infrastruktur |
| 🛡️ **Detection** | YARA-Regel auf Kit-Ebene: erkennt beide Mails, keine Fehlalarme im Test |

---

## Die beiden Mails

| Mail 1 – „Sicherheitsinformation" | Mail 2 – „Rechnung bereit" |
|---|---|
| ![Mail 1](artifacts/screenshots/mail1-ionos-spam.png) | ![Mail 2](artifacts/screenshots/mail2-ionos-spam.png) |
| Angst-Köder, minimal gehaltenes HTML | Geklontes IONOS-Rechnungstemplate |

*Beide Screenshots zeigen die Mails wie empfangen; die Empfängeradresse ist in jedem Vorkommen geschwärzt.*

---

## Inhalt

| Datei | Inhalt |
|---|---|
| [docs/01-mail1-sicherheitsinformation.md](docs/01-mail1-sicherheitsinformation.md) | Mail 1: „Sicherheitsticket“ – Header, Body, Link, IOCs, ATT&CK |
| [docs/02-mail2-rechnung.md](docs/02-mail2-rechnung.md) | Mail 2: „Rechnung bereit“ – kompromittierter Webserver, Template-Klon, Google-Redirect |
| [docs/03-kampagnen-vergleich.md](docs/03-kampagnen-vergleich.md) | Gleicher Akteur? Fingerprint-Vergleich, Pyramid of Pain, Zeitachse |
| [docs/04-detection-abwehr.md](docs/04-detection-abwehr.md) | YARA-Regel mit Testergebnis, Heuristiken, SPF/DKIM/DMARC-Härtung, Awareness, Incident Response |
| [docs/05-glossar-und-interviewfragen.md](docs/05-glossar-und-interviewfragen.md) | Alle Fachbegriffe erklärt + 10 Interviewfragen mit Antworten aus dem Fall |
| [docs/06-infrastruktur-analyse.md](docs/06-infrastruktur-analyse.md) | Passive Threat Intelligence: whois/DNS-Syntax erklärt, alle IPs und das Angreifer-Netz analysiert |
| [iocs/iocs.csv](iocs/iocs.csv) | Alle IOCs maschinenlesbar (entschärft) |
| [detection/ionos_homoglyph_kit.yar](detection/ionos_homoglyph_kit.yar) | YARA-Regel |
| [artifacts/](artifacts/) | Bereinigte Artefakte (eingebettete Bilder) |
| [reports/](reports/) | Aufbereitete Word-Berichte: Blue-Team-Analyse und Red-Team-Perspektive |

---

## Berichte (Word) zum Download

| Bericht | Umfang | Inhalt |
|---|---|---|
| [Blue-Team-Analyse](reports/Blue-Team-Analyse-IONOS-Phishing.docx) | 12 Seiten | Komplette forensische Analyse inkl. whois/DNS-Syntax, YARA, IOCs, Härtung |
| [Red-Team-Perspektive](reports/Red-Team-Perspektive-IONOS-Phishing.docx) | 7 Seiten | Angreifer-Entscheidungslog: pro Entscheidung Vorteil, Spur und Gegenmaßnahme |

---

## MITRE ATT&CK

| Technik | Mail 1 | Mail 2 |
|---|:-:|:-:|
| T1566.002 Phishing: Spearphishing Link | ✓ | ✓ |
| T1598.003 Phishing for Information: Spearphishing Link | ✓ | ✓ |
| T1036 Masquerading | ✓ | ✓ |
| T1027 Obfuscated Files or Information | ✓ | ✓ |
| T1583.006 Acquire Infrastructure: Web Services | ✓ | |
| T1584 Compromise Infrastructure | | ✓ |
| T1608.005 Stage Capabilities: Link Target | | ✓ |

---

## Methodik

1. **Beweissicherung:** Original-Quelldaten exportiert, SHA-256 gehasht, schreibgeschützt; Analyse nur an Kopien
2. **Statische Analyse:** Header-Kette, Authentifizierungsergebnisse, Dekodierung von Punycode, RFC 2047, Quoted-Printable, Base64, Unix-Timestamps
3. **Body-Analyse:** Parsing mit Python (`email`, `BeautifulSoup`), Unicode-Analyse jedes Nicht-ASCII-Zeichens, Entfernen versteckter Elemente, Extraktion eingebetteter Bilder
4. **Korrelation:** Vergleich beider Mails auf Kit-Fingerprints
5. **Detection Engineering:** YARA-Regel, Test gegen Positiv- und Negativ-Beispiele
6. **Keine aktive Interaktion** mit Angreifer-Infrastruktur in dieser Phase

## Datenschutz & Ethik

- Empfängeradresse und -domain sind geschwärzt; Provider-interne Header (`UI-InboundReport`) wurden entfernt
- Die Original-`.eml`-Dateien sind **nicht** Teil des Repositories (`.gitignore`)
- Alle IOCs sind entschärft (`hxxps://`, `[.]`)
- Die betroffene Website `organicaf[.]com[.]ar` ist selbst Opfer eines Angriffs

## Ausblick

- [x] Infrastruktur-Analyse (whois, DNS, Hoster, ASN, Angreifer-IP) — siehe [docs/06](docs/06-infrastruktur-analyse.md)
- [ ] Lookalike-Domains & DKIM-Selector `s1` per DNS auflösen
- [ ] Landingpages isoliert untersuchen (urlscan.io, Sandbox-VM)
- [ ] Abuse-Meldungen an AWS, Google, Hoster der Versand-IPs, IONOS
- [ ] **Automatisierung:** Python-Tool, das `.eml`-Dateien nach dieser Methodik analysiert und einen Report erzeugt

---

*Analyse: Pato81 · Oktober 2026*
