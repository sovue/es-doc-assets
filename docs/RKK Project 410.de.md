# API-Spezifikation des RKK Project 410 Begleitmoduls (In-Game Companion Hook)

:::info
**ANDERE SPRACHEN**

Dieses Dokument ist auch auf [Russisch](/docs/RKK%20Project%20410) und [Englisch](/docs/RKK%20Project%20410.en) verfügbar.
:::


Das Modul **RKK Hook** fungiert als systemintegrierter Begleiter (Companion) für den **RKK Project 410 Mod-Manager**. Das Modul wird direkt mit dem Manager ausgeliefert und erfordert keine separate Installation durch den Anwender. Im mod-eigenen Auswahlmenü wird die Komponente unter der Bezeichnung **„RKK Begleiter“** (bzw. *„RKK Companion“*) geführt.

Bei dieser Komponente handelt es sich **nicht** um eine eigenständige Story-Modifikation und nicht um einen Inhalt für den Steam Workshop. Eine Auslieferung der `.rpy`-Datei innerhalb Ihres eigenen Mod-Archivs ist nicht zulässig und nicht erforderlich.

Diese Spezifikation beschreibt die öffentliche Programmierschnittstelle (API) des Pakets **1.3.4** (`HOOK_VERSION` 13) für die Betriebssysteme Windows und Linux. Alle API-Funktionen sind ausfallsicher konstruiert: Bei fehlender Anbindung an den Manager werden sämtliche Aufrufe als leere, wirkungslose Operationen (no-op) ausgeführt und lösen keinerlei Systemausnahmen aus.

> **Wichtiger Hinweis zur Ausfallsicherheit (Schnittstellen-Prüfung)**  
> Wird Ihre Modifikation ohne installierten RKK Project 410 Mod-Manager ausgeführt, sind die Funktionsbezeichner `rkk_*` im Namensraum `store` nicht vorhanden. Ein direkter Aufruf führt unverzüglich zu einem `NameError`-Absturz. Zur Gewährleistung der Betriebssicherheit nutzen Sie den geschützten Zugriff via `getattr` oder hinterlegen Sie einen Schnittstellen-Blindbaustein (Shim) im `init`-Block Ihres Skripts.

---

## Systemarchitektur und Aufruf-Abfangung (Ren'Py-API-Interzeption)

Zur lückenlosen Sitzungserfassung, Absturzprotokollierung und nahtlosen Prozesssteuerung beim Wechsel zwischen Spiel und Manager klinkt sich das Begleitmodul in zentrale Kernfunktionen der Ren'Py-Laufzeitumgebung ein:

* **`config.exception_handler`**: Das Modul installiert eine eigene Fehlerfang-Routine und **ruft den zuvor gesetzten Handler ausnahmslos auf**. Sollten Sie `exception_handler` in Ihrer Modifikation überschreiben, müssen Sie die bestehende Aufrufkette zwingend aufrechterhalten und die vorige Funktion weiterleiten.
* **`os.startfile` und `webbrowser.open`**: Öffnungsaufrufe für Systemprotokolle (`traceback.txt`, `errors.txt`, `error.txt`) werden unterbunden. Dies verhindert das unerwünschte Aufpoppen des System-Explorers im Fehlerfall. Alle übrigen Datei- und URL-Pfade werden unangetastet durchgereicht.
* **`renpy.quit`**: Wird einmalig gekapselt. Die ordnungsgemäße Sitzungsbeendigung und Datensicherung werden zusätzlich in `config.quit_callbacks` und `config.python_exit_callbacks` eingereiht.
* **`config.label_callbacks` und `config.interact_callbacks`**: Die Rückrufroutinen des Begleiters werden an die bestehenden Listen angehängt. Eine Bereinigung der Listen findet nicht statt.

> **Sicherheitshinweis zur Durchkopplung von Rückrufketten**  
> Bei der Einbindung eigener Handler in `exception_handler` oder beim Kapseln von `renpy.quit` ist die vorherige Funktionsreferenz stets zu sichern und am Ende der eigenen Routine auszuführen. Das Zuweisen leerer Listen an `label_callbacks`, `interact_callbacks` oder `quit_callbacks` ist unzulässig, da hierdurch die Protokollierung des Begleiters oder Drittanbieter-Handler unterbrochen werden.

---

## Kompatibilitätssicherung (Schnittstellen-Blindbaustein / Shim)

### Variante 1: Einzellauf ohne Vorbereitung
Für seltene Einzelaufrufe kann die Abfrage direkt und sicher über `getattr` erfolgen:

```renpy
$ getattr(store, "rkk_note", lambda *a, **k: None)("Spieler hat Verzweigung erreicht")
```

### Variante 2: Schnittstellen-Blindbaustein (Süddeutscher Entwicklungsstandard)
Bei mehrfacher Schnittstellenverwendung hinterlegen Sie nachfolgenden Blindbaustein (Shim) in einer Ihrer `.rpy`-Quelldateien. Ersatzfunktionen werden ausschließlich dann deklariert, wenn die echten Begleitfunktionen fehlen. Die Ladefolge ist unerheblich: Ist der Manager installiert, überschreiben dessen Echtsystem-Funktionen die Platzhalter stets ordnungsgemäß.

```renpy
init -1500 python:
    if "rkk_note" not in dir(store):
        def rkk_note(text, tag=None):
            pass
    if "rkk_report_mod_version" not in dir(store):
        def rkk_report_mod_version(mod_label, version):
            pass
    if "rkk_report_mod_title" not in dir(store):
        def rkk_report_mod_title(mod_label, title):
            pass
    if "rkk_is_companion_available" not in dir(store):
        def rkk_is_companion_available():
            return False
    if "rkk_companion_info" not in dir(store):
        def rkk_companion_info():
            return {"available": False}
    if "rkk_get_active_mods" not in dir(store):
        def rkk_get_active_mods():
            return {}
    if "rkk_set_context" not in dir(store):
        def rkk_set_context(key, value):
            pass
    if "rkk_get_context" not in dir(store):
        def rkk_get_context():
            return {}
    if "rkk_open_manager" not in dir(store):
        def rkk_open_manager():
            pass
    if "rkk_visual_poll_reload" not in dir(store):
        def rkk_visual_poll_reload():
            return False
```

---

## API-Funktionsreferenz

### `rkk_note(text, tag=None)`
Trägt eine Ablaufmarke (Breadcrumb / Spurpunkt) in den Sitzungsverlauf und in das Absturzprotokoll ein. Erfasst den Hinweistext, den Zeitstempel sowie das aktuelle Skript-Label. Die Einträge werden **nicht** als dauerhafte Logdatei auf dem Datenträger abgelegt.

```renpy
$ rkk_note("Galerieansicht geöffnet")
$ rkk_note("Spieler bei Kantine verstorben", tag="death")
```

* **Parameter:**
  * `text` (`str`) — Hinweistext (maximal 160 Zeichen).
  * `tag` (`str`, optional) — Kategorie- oder Ereignistag (maximal 40 Zeichen).
* **Begrenzung:** Im Absturzprotokoll werden die letzten 20 Ablaufmarken aufbewahrt.
* **Empfehlung:** Setzen Sie den Aufruf gezielt an handlungsrelevanten Knotenpunkten ein. Vermeiden Sie Aufrufe bei jedem regulären `interact`-Schritt.

---

### `rkk_set_context(key, value)` / `rkk_get_context()`
Verwaltet dauerhafte Kontext-Stammdaten der aktuellen Sitzung. Im Gegensatz zu den einmaligen Ablaufmarken von `rkk_note` bleibt der Kontextwert bestehen, bis er explizit geändert oder zurückgesetzt wird. Das Übergeben von `None` oder einer leeren Zeichenkette löscht den Schlüssel.

```renpy
$ rkk_set_context("route", "Ulyana")
$ rkk_set_context("day", "7")
$ rkk_set_context("route", None) # Schlüssel löschen

$ aktueller_kontext = rkk_get_context()
```

* **Begrenzungen:**
  * Maximal 16 aktive Kontextschlüssel gleichzeitig.
  * Schlüssellänge (`key`) — maximal 40 Zeichen.
  * Wertlänge (`value`) — maximal 80 Zeichen.

---

### `rkk_report_mod_version(mod_label, version)`
Übermittelt den Versionsstand Ihrer Modifikation an das Begleitmodul zwecks Zuordnung in Fehlerprotokollen. Der Aufruf erfolgt einmalig während der Initialisierung. Die Kennung `mod_label` muss exakt mit dem Schlüssel im globalen Verzeichnis `mods[...]` übereinstimmen.

```renpy
init:
    $ mods["meine_mod"] = "Meine Modifikation"
    $ rkk_report_mod_version("meine_mod", "1.4.2")
```

* **Begrenzungen:** `mod_label` maximal 80 Zeichen, `version` maximal 40 Zeichen. Wichtig: Ohne diesen Aufruf wird der Versionsstand im Absturzprotokoll nicht aufgeführt.

---

### `rkk_report_mod_title(mod_label, title)`
Meldet den Klarnamen der Modifikation für die Bibliotheksanzeige des Managers ein. Dies ist erforderlich, wenn der Wert in `mods[...]` dynamisch über Variablen oder Lokalisierungsaufrufe `_()` gebildet wird und daher vom statischen Parser des Managers nicht ausgelesen werden kann.

```renpy
init python:
    meine_mod_name = _("Meine Modifikation")

init:
    $ mods["meine_mod"] = meine_mod_name
    $ rkk_report_mod_title("meine_mod", meine_mod_name)
```

* **Begrenzungen:** Titellänge maximal 120 Zeichen. Die Angaben werden in `rkk/mod-titles.json` sowie im Sitzungsexport hinterlegt.

---

### `rkk_get_active_mods()`
Liefert ein Verzeichnis des Typs `{mod_label: version}` aller Modifikationen zurück, die in der laufenden Sitzung `rkk_report_mod_version` aufgerufen haben. Dient zur Durchführung von Kompatibilitäts- und Abhängigkeitsprüfungen zur Laufzeit ohne Einbindung der Manager-Oberfläche.

```renpy
$ aktive_versionen = rkk_get_active_mods()
if "andere_mod" in aktive_versionen:
    $ rkk_note("Kompatibilitätsprüfung: andere_mod v" + aktive_versionen["andere_mod"] + " erkannt", tag="compat")
```

---

### `rkk_is_companion_available()` / `rkk_companion_info()`
Dienen der Statusprüfung und Parameterabfrage des Begleitmoduls vor dem Aufbau eigener Benutzeroberflächen. `rkk_is_companion_available()` gibt `True` zurück, wenn die Konfigurationsdatei `hook.ini` vorhanden und der hinterlegte Manager-Pfad gültig ist.

```renpy
if rkk_is_companion_available():
    $ info = rkk_companion_info()
```

Aufbau des Rückgabe-Verzeichnisses von `rkk_companion_info()`:

| Schlüssel | Typ | Beschreibung |
| :--- | :--- | :--- |
| `available` | `bool` | Betriebsbereitschaft der Begleiter-Anbindung |
| `hook_version` | `int` | Numerische API-Version des Begleiters (aktuell: `13`) |
| `hook_version_label` | `str` | Paket-Versionsbezeichnung (`"1.3.4"`) |
| `detect_crashes` | `bool` | Status der Absturzprotokollierung |
| `session_id` | `str` | Eindeutige Kennung der aktuellen Spielsitzung |

Bei fehlender Anbindung liefern die Funktionen `False` bzw. `{"available": False}` zurück.

---

### `rkk_open_manager()`
Sichert und exportiert den Sitzungszustand, startet den RKK Project 410 Mod-Manager und beendet den Spielprozess geordnet. Der Aufruf darf erst nach erfolgreicher Prüfung mittels `rkk_is_companion_available()` ausgeführt werden.

```renpy
if rkk_is_companion_available():
    textbutton _("RKK Project 410 Manager"):
        action Function(rkk_open_manager)
```

Sollte die Ausführungsdatei des Managers nicht gestartet werden können, wird ein Systemhinweis (`renpy.notify`) ausgegeben und der Spielbetrieb aufrechtgehalten.

---

### `rkk_visual_poll_reload()`
Entwickler-Hilfsfunktion für das Tooling von Visual Author (nicht für den regulären Handlungsablauf vorgesehen). Prüft auf das Vorhandensein der Steuerdatei `.rkk_visual_reload` im Verzeichnis `game/` oder im Hauptordner. Bei Fund wird die Datei entfernt und `renpy.reload_script()` ausgeführt. Gibt `True` zurück, wenn ein Skript-Reload veranlasst wurde, sonst `False`.

---

## Vollständiges Einbindungsbeispiel

```renpy
# 1. Sicherheits-Blindbausteine für den autarken Betrieb
init -1500 python:
    if "rkk_note" not in dir(store):
        def rkk_note(text, tag=None):
            pass
    if "rkk_report_mod_version" not in dir(store):
        def rkk_report_mod_version(mod_label, version):
            pass
    if "rkk_set_context" not in dir(store):
        def rkk_set_context(key, value):
            pass
    if "rkk_get_active_mods" not in dir(store):
        def rkk_get_active_mods():
            return {}

# 2. Registrierung der Modifikation
init:
    $ mods["meine_mod_label"] = "Meine Story-Modifikation"
    $ rkk_report_mod_version("meine_mod_label", "1.4.2")

# 3. Spielablauf
label meine_mod_label:
    $ rkk_set_context("route", "Ulyana")
    $ rkk_set_context("day", "7")

    $ aktive_mods = rkk_get_active_mods()
    if "busy_patch" in aktive_mods:
        $ rkk_note("Kompatibilitätspatch aktiv: busy_patch " + aktive_mods["busy_patch"], tag="compat")

    "Der Hauptdarsteller nähert sich dem Kantinengebäude."
    $ rkk_note("Spieler nahe der Kantine verstorben", tag="death")
    return
```

Beim Betrieb der Modifikation ohne den RKK Project 410 Mod-Manager läuft das Skript vollkommen autark und fehlerfrei ab. Ist der Manager installiert, werden Versionsstände, Kontext-Stammdaten und Ablaufmarken automatisch im Sitzungsprotokoll erfasst.
