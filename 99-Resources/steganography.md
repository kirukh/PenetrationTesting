# 🖼️ Steganografie & versteckte Credentials

> Passwörter verstecken sich oft nicht im Dateisystem-Text, sondern **in** Dateien:
> in Bildern (sichtbar oder eingebettet), in Metadaten oder in Binaries.
> Immer prüfen, wenn du eine Bild-/Mediendatei oder eine ungewöhnliche Binary findest.

---

## 1. `strings` — der erste Griff bei JEDER unbekannten Datei

Liest alle lesbaren Zeichenketten aus einer Binär- oder Mediendatei. Findet
hardcodierte Passwörter, Pfade, URLs, Kommentare.

```bash
strings datei
strings -n 8 datei                 # nur Strings >= 8 Zeichen (weniger Rauschen)
strings -e l datei                 # 16-bit little-endian (Windows-Strings)
strings datei | grep -i -E 'pass|pwd|user|key|token|cred'
```

> **Lab-Beispiel VM1 (192.168.1.155):** In der für alle lesbaren Binary
> `customPermissionApp` (im `.hiddenadmindirectory/` des Users `bulldogadmin`)
> lag das Klartext-Passwort des `django`-Users — über mehrere Zeilen verteilt.
> `strings customPermissionApp | grep -i -A1 pass` förderte es zutage.
> Lektion: **jede** auffindbare Binary mit `strings` prüfen, bevor man weitersucht.

---

## 2. Sichtbarer Text im Bild

Manchmal ist gar nichts "versteckt" — die Credentials stehen einfach lesbar
**im Bildinhalt** (z.B. auf einem Schild, einer Boje, einem Screenshot).

```bash
# Bild einfach ansehen
xdg-open bild.jpg
feh bild.jpg
eog bild.jpg

# Bei kleinen/pixeligen Bildern: hochskalieren zum Lesen
convert bild.png -resize 400% gross.png
```

> **Lab-Beispiel VM11 (192.168.2.221):** In `backup.zip` lag eine Bilddatei
> (`PWD.jpeg`), auf der die Zugangsdaten direkt sichtbar abgedruckt waren
> (`gamma:gammacanbedangerous!`) → direkter RDP-Login.

---

## 3. EXIF / Metadaten

Creds, Kommentare oder Hinweise stecken oft in den Metadaten.

```bash
exiftool bild.jpg
exiftool -a -u -g1 bild.jpg        # alle Tags, auch unbekannte, gruppiert
exiftool *.jpg | grep -i -E 'comment|author|description|owner'

# Schnell der Comment-Tag:
exiftool -Comment bild.jpg
identify -verbose bild.jpg         # ImageMagick-Variante
```

---

## 4. Eingebettete Daten (echte Steganografie)

### steghide (JPEG/BMP/WAV/AU — sehr verbreitet in Challenges)

```bash
# Info anzeigen (ist was eingebettet?)
steghide info bild.jpg

# Extrahieren (fragt nach Passphrase — leer lassen mit Enter probieren)
steghide extract -sf bild.jpg
steghide extract -sf bild.jpg -p 'passphrase'

# Passphrase brute-forcen (wenn nötig)
stegseek bild.jpg /usr/share/wordlists/rockyou.txt
stegseek --crack bild.jpg rockyou.txt out.txt
```

### Weitere Tools / Dateitypen

```bash
# PNG-spezifisch (LSB, Kanäle, Bit-Ebenen)
zsteg bild.png
zsteg -a bild.png                  # alle Methoden

# Generisch: angehängte/eingebettete Dateien finden
binwalk datei                      # zeigt eingebettete Dateisignaturen
binwalk -e datei                   # extrahiert sie
foremost -i datei -o out/          # File-Carving

# Manuell: angehängtes Archiv hinter einem Bild?
unzip bild.jpg                     # JPEG+ZIP-Trick funktioniert oft
cat bild.jpg | tail -c +<offset> > extrahiert.zip
```

---

## 🔁 Workflow bei einer verdächtigen Mediendatei

```
1. strings datei | grep -iE 'pass|user|key'     → Klartext drin?
2. Bild ansehen (ggf. hochskalieren)             → sichtbarer Text?
3. exiftool datei                                → Metadaten/Comment?
4. steghide info / steghide extract              → eingebettet (JPEG/WAV)?
5. zsteg (PNG) / binwalk / foremost              → versteckte Dateien?
6. stegseek bild rockyou.txt                     → Passphrase brute-forcen
```

## ➡️ Credentials gefunden?

- Hash → [Passwords / Hashes](../02-Exploitation/Passwords/README.md)
- Login-Daten für Dienst → je nach Port: [SMB](../01-Exploration/Services/smb.md) · [RDP](../01-Exploration/Services/rdp.md) · [SSH](../01-Exploration/Services/ssh.md)
