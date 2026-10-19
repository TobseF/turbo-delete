# Turbo Delete auf winget veröffentlichen

Die fertigen und mit `winget validate` geprüften Manifeste liegen in
[`manifests/t/Tobse/TurboDelete/1.0.0/`](manifests/t/Tobse/TurboDelete/1.0.0/):

| Datei | Inhalt |
|---|---|
| `Tobse.TurboDelete.yaml` | Versionsmanifest |
| `Tobse.TurboDelete.installer.yaml` | Installationsmanifest (Inno Setup, Benutzer-Scope) |
| `Tobse.TurboDelete.locale.en-US.yaml` | Standardgebietsschema (Englisch) |
| `Tobse.TurboDelete.locale.de-DE.yaml` | Deutsche Beschreibung (optional) |

## 1. Release vorbereiten

Das Release `1.0.0` enthält bisher ein Setup, das sich in Windows als Version **0.2.0** registriert.
winget würde das Paket dann für immer als veraltet anzeigen. Das Setup in diesem Branch ist als
1.0.0 gebaut und hat zusätzlich das Icon.

1. Im Release [`1.0.0`](https://github.com/TobseF/turbo-delete/releases/tag/1.0.0) die beiden Assets
   durch `dist/Turbo-Delete-Install.exe` und `bin/turbo-delete.exe` aus diesem Branch ersetzen.
2. Prüfen, ob der Hash des hochgeladenen Setups mit dem Manifest übereinstimmt:

   ```powershell
   (Invoke-WebRequest https://github.com/TobseF/turbo-delete/releases/download/1.0.0/Turbo-Delete-Install.exe -OutFile $env:TEMP\tdi.exe); (Get-FileHash $env:TEMP\tdi.exe).Hash
   ```

   Erwartet: `FE618964D6A20AB7570A1637E4CCECA85AEDBF2407407BA445AD072FFCD28B32`

   Inno Setup schreibt beim Bauen einen Zeitstempel ins Setup. Jeder neue Build hat also einen
   anderen Hash. `python automate-release.py` gibt den Hash nach dem Bauen aus.

## 2. Einreichen

### Variante A: fertige Manifeste einreichen (empfohlen)

```powershell
winget install wingetcreate
wingetcreate token --store          # einmalig: GitHub-Token hinterlegen
wingetcreate submit .\winget\manifests\t\Tobse\TurboDelete\1.0.0
```

`wingetcreate` legt dann den Pull Request in `microsoft/winget-pkgs` an. Dabei bleiben alle
Felder erhalten, auch die deutsche Beschreibung, die Installationshinweise und
`AppsAndFeaturesEntries`.

### Variante B: Wizard (`wingetcreate new`)

Installer-URL:

```
https://github.com/TobseF/turbo-delete/releases/download/1.0.0/Turbo-Delete-Install.exe
```

Der Wizard erkennt `InstallerType: inno` und berechnet den SHA256 selbst. Diese Werte trägst du ein:

| Feld | Wert |
|---|---|
| PackageIdentifier | `Tobse.TurboDelete` |
| PackageVersion | `1.0.0` |
| Scope | `user` |
| ProductCode | `{71679A1F-AE2C-4DC8-BB8E-03C0652B6527}_is1` |
| UpgradeBehavior | `install` |
| Commands | `turbo-delete` |
| Publisher | `Tobse` |
| PublisherUrl | `https://github.com/TobseF` |
| PublisherSupportUrl | `https://github.com/TobseF/turbo-delete/issues` |
| Author | `Tejas Ravishankar, Tobse Fritz` |
| PackageName | `Turbo Delete` |
| PackageUrl | `https://github.com/TobseF/turbo-delete` |
| License | `Apache-2.0` |
| LicenseUrl | `https://github.com/TobseF/turbo-delete/blob/master/LICENSE.txt` |
| Copyright | `Copyright 2022 Tejas Ravishankar` |
| ShortDescription | `Delete folders with hundreds of thousands of files in seconds instead of minutes` |
| Moniker | `turbo-delete` |
| Tags | `delete, rmdir, rm, remove, cleanup, node-modules, temp, context-menu, cli, fast, multithreaded, rust` |
| ReleaseNotesUrl | `https://github.com/TobseF/turbo-delete/releases/tag/1.0.0` |
| ReleaseDate | `2026-10-19` |

Description und InstallationNotes kopierst du aus `Tobse.TurboDelete.locale.en-US.yaml`.

`Publisher` und `PackageName` müssen zum Eintrag in Einstellungen > Apps passen. Das Setup trägt sich
dort als „Turbo Delete“ von „Tobse“ ein. So erkennt winget die Installation bei `upgrade` und `list`.

## 3. Optional: vorher lokal testen

```powershell
# einmalig, als Admin
winget settings --enable LocalManifestFiles

winget install --manifest .\winget\manifests\t\Tobse\TurboDelete\1.0.0
winget list Tobse.TurboDelete      # muss Version 1.0.0 zeigen, ohne verfügbares Update
winget uninstall Tobse.TurboDelete
```

## Icon

Das Icon lässt sich nicht über das winget-Manifest setzen:

- Die aktuelle Schema-Dokumentation für das Standardgebietsschema
  ([Manifest 1.12](https://github.com/microsoft/winget-pkgs/blob/master/doc/manifest/schema/1.12.0/defaultLocale.md))
  enthält gar kein Icon-Feld mehr.
- Als „restricted“ und nur für verifizierte Entwickler gekennzeichnet sind dort nur `Agreements`.
  Deine Beobachtung aus dem Wizard passt dazu, dass Icons für normale Einreichungen nicht vorgesehen sind.
- Eine offizielle Aussage, dass Icons nur für von Microsoft autorisierte Entwickler erlaubt sind,
  habe ich nicht gefunden.

Deshalb steckt das Mülleimer-Icon jetzt im Setup selbst (`assets/turbo-delete.ico`). Es erscheint
im Setup-Fenster, in Einstellungen > Apps und im Kontextmenü. Tools wie UniGetUI lesen das Icon
von dort.

## Nach dem Merge in winget-pkgs

- In der README unter „Installation“ ergänzen:

  ```powershell
  winget install Tobse.TurboDelete
  ```

- Für neue Versionen:

  ```powershell
  wingetcreate update Tobse.TurboDelete --version <neu> --urls <Installer-URL> --submit
  ```
