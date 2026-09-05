# RKK Project 410 In-Game Companion API Specification

:::info
**OTHER LANGUAGES**

This document is also available in [Russian](/docs/RKK%20Project%20410) and [German](/docs/RKK%20Project%20410.de).
:::


The **RKK Hook** module is an in-game companion for the **RKK Project 410 Mod Manager**. The module is bundled directly with the manager and does not require separate installation by players. In the game's modification menu, it appears as **"RKK Companion"** (or *"RKK Компаньон"*).

This module is **not** a story mod and is not distributed as a standalone file on the Steam Workshop. Do not include your own copy of the `.rpy` hook file in your mod's distribution archive.

This guide describes the public API for package version **1.3.4** (`HOOK_VERSION` 13) on Windows and Linux. All API functions are designed to fail silently: if the manager is not installed or inactive, calls execute as safe no-ops and never raise exceptions.

> **Important: Handling Missing Manager Integration**  
> If a user runs your mod without the RKK Project 410 Mod Manager installed, the `rkk_*` functions will not exist in the `store` namespace. Calling them directly will throw a `NameError`. To prevent crashes, use safe access via `getattr` or declare a compatibility shim in your mod's `init` block.

---

## Architecture and Ren'Py API Interception

To track session state, capture crash dumps, and handle seamless transitions between the game and the launcher, the hook intercepts several native Ren'Py mechanisms:

* **`config.exception_handler`**: The module installs its own error handler and **always invokes the previously registered handler** in the chain. If you overwrite `exception_handler` in your own mod, always preserve and call the previous handler.
* **`os.startfile` & `webbrowser.open`**: The hook intercepts attempts to open system crash logs (`traceback.txt`, `errors.txt`, `error.txt`). This prevents Windows Explorer windows from popping up unexpectedly during a crash. All other file paths and URLs are processed normally.
* **`renpy.quit`**: Wrapped once. Session state saving and cleanup are registered via `config.quit_callbacks` and `config.python_exit_callbacks`.
* **`config.label_callbacks` & `config.interact_callbacks`**: Hook callbacks are appended to these lists without clearing existing handlers.

> **Warning: Preserving Callback Chains**  
> When registering custom `exception_handler` functions or wrapping `renpy.quit`, always store a reference to the previous function and call it at the end of your handler. Do not assign empty lists to `label_callbacks`, `interact_callbacks`, or `quit_callbacks`—doing so will break crash reporting or drop handlers installed by other mods.

---

## Ensuring Compatibility (Shims)

### Method 1: One-Off Calls
For occasional API calls, use a safe lookup via `getattr`:

```renpy
$ getattr(store, "rkk_note", lambda *a, **k: None)("Player reached a storyline branch")
```

### Method 2: Compatibility Shim (Recommended)
If your mod calls the API frequently, add the following shim block to one of your project's `.rpy` files. Dummy functions are created only when the actual companion functions are absent. Load order does not matter: if the manager is installed, its live functions will always take precedence over the dummy definitions.

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

## API Reference

### `rkk_note(text, tag=None)`
Logs a lightweight event marker (breadcrumb) to the current session timeline and crash dump. Captures the message string, timestamp, and current script label. Notes **are not written** to a separate log file on disk.

```renpy
$ rkk_note("Character gallery opened")
$ rkk_note("Player died at the cafeteria", tag="death")
```

* **Parameters:**
  * `text` (`str`) — Note content (up to 160 characters).
  * `tag` (`str`, optional) — Event category tag (up to 40 characters).
* **Limits:** Only the last 20 notes are preserved in crash reports.
* **Best Practices:** Call this function at key narrative branch points or major milestones. Avoid invoking it inside high-frequency `interact` callbacks.

---

### `rkk_set_context(key, value)` / `rkk_get_context()`
Sets or retrieves persistent key-value tags for the active session. Unlike one-off `rkk_note` entries, context key-value pairs persist until explicitly changed or cleared. To delete a key, pass `None` or an empty string as the `value`.

```renpy
$ rkk_set_context("route", "Ulyana")
$ rkk_set_context("day", "7")
$ rkk_set_context("route", None) # Clears the key

$ current_context = rkk_get_context()
```

* **Limits:**
  * Up to 16 active context keys simultaneously.
  * Key length (`key`) — up to 40 characters.
  * Value length (`value`) — up to 80 characters.

---

### `rkk_report_mod_version(mod_label, version)`
Registers your mod's version string with the companion to correlate error reports. Should be called once during initialization. The `mod_label` identifier must match the key used to register your mod in the global `mods[...]` dictionary.

```renpy
init:
    $ mods["my_cool_mod"] = "My Mod"
    $ rkk_report_mod_version("my_cool_mod", "1.4.2")
```

* **Limits:** `mod_label` length — up to 80 characters; `version` length — up to 40 characters. If omitted, your mod's version will not appear in crash reports.

---

### `rkk_report_mod_title(mod_label, title)`
Registers a human-readable display title for the manager's library view. Use this when the value in `mods[...]` is generated dynamically (e.g., via variables or localization calls like `_()`), which prevents the manager's static parser from reading the title directly from the file.

```renpy
init python:
    my_mod_name = _("My Mod")

init:
    $ mods["my_cool_mod"] = my_mod_name
    $ rkk_report_mod_title("my_cool_mod", my_mod_name)
```

* **Limits:** Title length — up to 120 characters. Data is written to `rkk/mod-titles.json` and exported in session summaries.

---

### `rkk_get_active_mods()`
Returns a `{mod_label: version}` dictionary containing all mods that invoked `rkk_report_mod_version` in the current session. Useful for performing soft-dependency and cross-mod compatibility checks at runtime without requiring launcher UI support.

```renpy
$ active_versions = rkk_get_active_mods()
if "another_mod" in active_versions:
    $ rkk_note("Compatibility check: detected another_mod v" + active_versions["another_mod"], tag="compat")
```

---

### `rkk_is_companion_available()` / `rkk_companion_info()`
Used to verify companion availability and retrieve metadata before rendering custom UI elements. `rkk_is_companion_available()` returns `True` if `hook.ini` exists and the configured manager binary path is valid.

```renpy
if rkk_is_companion_available():
    $ info = rkk_companion_info()
```

Dictionary structure returned by `rkk_companion_info()`:

| Key | Type | Description |
| :--- | :--- | :--- |
| `available` | `bool` | Companion integration status flag |
| `hook_version` | `int` | Numeric hook API version (currently `13`) |
| `hook_version_label` | `str` | Package version string (`"1.3.4"`) |
| `detect_crashes` | `bool` | Crash interception handler state |
| `session_id` | `str` | Unique session identifier for the current run |

When the hook is not loaded, these functions return `False` and `{"available": False}`, respectively.

---

### `rkk_open_manager()`
Saves and exports session state, launches the RKK Project 410 Mod Manager, and terminates the game process. Call this function only after verifying availability with `rkk_is_companion_available()`.

```renpy
if rkk_is_companion_available():
    textbutton _("RKK Project 410 Manager"):
        action Function(rkk_open_manager)
```

If the manager binary cannot be launched, a `renpy.notify` toast is displayed and the game continues running.

---

### `rkk_visual_poll_reload()`
A developer helper for the Visual Author tool (not intended for story code). Checks for a `.rkk_visual_reload` stamp file in the `game/` folder or root project directory. If present, it deletes the stamp file and invokes `renpy.reload_script()`. Returns `True` if a reload was triggered, otherwise `False`.

---

## Full Integration Example

```renpy
# 1. Safe shims for standalone operation
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

# 2. Mod registration
init:
    $ mods["my_mod_label"] = "My Story Mod"
    $ rkk_report_mod_version("my_mod_label", "1.4.2")

# 3. Game script
label my_mod_label:
    $ rkk_set_context("route", "Ulyana")
    $ rkk_set_context("day", "7")

    $ mods_active = rkk_get_active_mods()
    if "busy_patch" in mods_active:
        $ rkk_note("Compatibility patch enabled: busy_patch " + mods_active["busy_patch"], tag="compat")

    "The protagonist approaches the cafeteria."
    $ rkk_note("Player died near the cafeteria", tag="death")
    return
```

When running without the RKK Project 410 Mod Manager installed, this script acts as a standard standalone mod without throwing any errors. When the manager is active, version numbers, context tags, and timeline notes are automatically recorded in session reports.
