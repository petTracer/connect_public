# petTracer Firmware – gemeinsames Repository

Ein Ablageort für alle Gerätearten. Die App liest genau eine Datei:
`firmware.json`. Pro Geräteart wird immer genau **eine** Firmware
angeboten – die aktuelle.

## Struktur

```
firmware.json                                  ← einziges Manifest
collar/catcollar_release_13.5.0.gbl            ← genau eine Datei
homestation/homestation_release_14.0.2.gbl     ← genau eine Datei
```

Beim Veröffentlichen wird die alte `.gbl` gelöscht und die neue abgelegt.
Alte Stände gehören nicht ins Repository – sie sind über die Git-Historie
weiterhin auffindbar, falls man doch einmal zurückmuss.

## Wie die Trennung abgesichert ist

Früher lagen Halsband und Homestation in getrennten Repositories. Jetzt
liegen sie am selben Ort, deshalb übernimmt das Manifest die Trennung –
mit drei Prüfungen in der App:

1. Die App liest **nur den Abschnitt ihrer Geräteart** (`collar` bzw.
   `homestation`). Der andere Abschnitt wird nie angefasst.
2. Im Abschnitt muss das Feld `deviceType` mit dem Abschnittsnamen
   **übereinstimmen**. Wird ein Eintrag versehentlich in den falschen
   Abschnitt kopiert, fällt das auf und es wird kein Update angeboten.
3. Vor der Übertragung wird die **SHA-256-Prüfsumme** der heruntergeladenen
   Datei gegen den Wert im Manifest geprüft. Stimmt sie nicht oder fehlt
   sie, bricht die App ab, bevor ein einziges Byte auf das Gerät geht.

Die zweite Prüfung ist strenger als die alte Lösung: Zwei getrennte
Repositories haben ein falsch abgelegtes `.gbl` nicht bemerkt.

## Neue Firmware veröffentlichen

Beispiel Halsband 13.6.0:

```bash
git rm collar/catcollar_release_13.5.0.gbl
cp ~/Downloads/catcollar_release_13.6.0.gbl collar/
shasum -a 256 collar/catcollar_release_13.6.0.gbl
```

Danach in `firmware.json` im Abschnitt `collar` anpassen: `version`, den
Dateinamen in `url`, `sha256` und `releaseDate`. `deviceType` bleibt
unverändert.

`showNotes` auf `true` setzen, wenn der Änderungstext unter „Das ist neu"
in der App erscheinen soll – bei `false` bleibt er verborgen.

Committen und pushen. Die App holt das Manifest bei jedem Versionscheck
neu; ein App-Update ist nicht nötig.

## Felder in `firmware.json`

| Feld | Pflicht | Bedeutung |
|---|---|---|
| `deviceType` | ja | `collar` oder `homestation`, muss dem Abschnitt entsprechen |
| `version` | ja | Semantische Version, z. B. `13.5.0` – die App vergleicht numerisch und bietet nie ein Downgrade an |
| `url` | ja | Direkter Link auf die `.gbl`-Datei |
| `sha256` | ja | Prüfsumme der Datei, Kleinbuchstaben |
| `releaseDate` | nein | Nur zur Dokumentation |
| `showNotes` | nein | Steuert, ob `notes` in der App sichtbar sind |
| `notes` | nein | Änderungstext je Sprache (`de`, `en`) |

## Umstellung auf den petTracer-Server

Im Produktivbetrieb wird nur eine Zeile in der App getauscht:
`OtaConstants.firmwareManifestUrl` in `lib/ble/ota_constants.dart`.
Die `url`-Felder im Manifest zeigen dann auf den petTracer-Server statt
auf GitHub. Am Format ändert sich nichts.
