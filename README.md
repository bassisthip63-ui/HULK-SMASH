# HULK-SMASH
GTA IV  IMG Limit Adjuster
# HULKSMASH 1.5.1

**A streaming limit adjuster for GTA IV and Episodes from Liberty City.**

HULKSMASH is an `.asi` plugin that lifts the game's limit of 255 `.img` archives, so large mod setups with
hundreds of `.img` files can load. It can also raise the vehicle and ped streaming budgets, and it writes a
crash log that helps track down what went wrong when the game crashes.

- **Supported game builds:** `GTAIV.exe` 1.0.7.0, 1.0.8.0 and 1.2.0.59 (Complete Edition)
- **Works in:** GTA IV, The Lost and Damned and The Ballad of Gay Tony

---

## Install

1. You need an ASI loader, for example Ultimate ASI Loader (FusionFix already includes it).
2. Copy `HULKSMASH.asi` and `HULKSMASH.ini` into the `plugins` folder next to `GTAIV.exe`.
3. Start the game, then open `plugins\HULKSMASH.log`. It should say:

   ```
   HULKSMASH 1.5.1
   ...
   OK: 1023 archives enabled (58 patches applied).
   ```

## Files

| File | What it is |
|---|---|
| `HULKSMASH.asi` | The plugin |
| `HULKSMASH.ini` | Settings. Every section has its own `Enabled` switch, and the comments explain each key. |
| `HULKSMASH.log` | Written at every start: what each section did. Created by the plugin. |
| `HULKSMASH.crash.log` | Crash records, kept between game sessions (started fresh once it passes 512 KB). Created by the plugin. Delete it before a test run to get a clean report. |

## Settings

### `[ArchiveLimit]` — raises the `.img` archive limit (on by default)

| Key | Default | Meaning |
|---|---|---|
| `Enabled` | `1` | `0` turns the section off. |
| `MaxArchives` | `1023` | `255` or less = stock game. Other values round up to `511`, `1023` or `2047` (the maximum). At `1023`, each archive can still be up to 4096 MB. |
| `ReportUsage` | `1` | Adds a line like `[Usage] archives in use: 368 of 1023 (when the game closed)` to `HULKSMASH.log`, so you can see how close you are to the limit. `0` = off. |

### `[VehicleBudget]` — streaming memory for vehicle models, the "taxi bug" fix (off by default)

| Key | Default | Meaning |
|---|---|---|
| `Enabled` | `0` | `1` turns it on. |
| `Value` | `120000000` | New budget in bytes (game default 40,000,000). |
| `Overwrite` | `0` | `0` leaves the value alone if another mod or exe patch already changed it; `1` always sets it. |

### `[PedBudget]` — streaming memory for pedestrian models (off by default)

| Key | Default | Meaning |
|---|---|---|
| `Enabled` | `0` | `1` turns it on. |
| `Value` | `75000000` | New budget in bytes (game default 50,000,000). |
| `Overwrite` | `0` | Same as for `[VehicleBudget]`. |

### `[CrashLog]` — records serious exceptions to `HULKSMASH.crash.log` (on by default)

| Key | Default | Meaning |
|---|---|---|
| `Enabled` | `1` | `0` turns the crash log off. |

## Reading the crash log

The crash log records **first-chance** exceptions: the game or another mod may still handle them, so not every
entry is a crash. **The entry that matters is the last one before the game closed.** Each full record shows the
exception, the module and offset it happened in, how many archives were in use, the registers, the stack and the
loaded modules.

Things most setups show that are harmless:

- a few access violations in the first seconds, at `archives in use: 0` (game start-up)
- access violations that repeat during play while the game keeps running (another mod; handled)
- one last access violation in `GTAIV.exe` at the moment you quit (the game's shutdown, not this plugin)

If an entry names **`HULKSMASH stub`** or **`HULKSMASH archive table`**, the plugin's own code or memory is
involved, so please report that one.

The crash log never changes or hides an exception: the game crashes, or carries on, exactly as it would without it.

## Compatibility

- The plugin must be loaded at start-up, as an `.asi`. If it loads after the game has already set up its archive
  table, it changes nothing and says so in `HULKSMASH.log`.
- Every byte is checked before it is changed. If another mod already patched the same spot, or the exe is not a
  supported build, that section changes nothing and the log says why. The other sections are not affected.
- Don't use another archive-limit patch alongside it.

## Changes

### 1.5.1

Same features and settings as 1.5. Only how the plugin is built changed, after antivirus programs wrongly flagged 1.5:
- The small pieces of code HULKSMASH adds to the game are now written into normal memory first and only then made
  executable (read-only). No memory is ever writable and executable at the same time.
- The file now carries version information (product name, description, version) and a valid PE checksum.

### 1.5

- **Fixed:** with `ReportUsage=1`, The Ballad of Gay Tony crashed (heap corruption) shortly after loading. 1.4
  counted archives on a background thread, and one extra thread in the game was enough to set off the crash.
  1.5 has no thread: it writes the `[Usage]` line when the crash log records an exception and when the game closes.

### 1.4 (never released)

- Added `ReportUsage` (the `[Usage]` line) and the archive count in crash-log records.
- Known problem: crashes in TBoGT with `ReportUsage=1`. Use 1.5, or set `ReportUsage=0`.

## Building from source

See `HULKSMASH_source_1.5.1.zip` (`SOURCE_NOTES.txt` inside).

- **Windows:** install LLVM (`winget install LLVM.LLVM`), unzip to a short path such as `C:\HULKSMASH`, and run
  `build_win.cmd`.
- **Linux:** `./build.sh` (needs clang, lld and llvm-dlltool).

The test harness in `test/` runs only on Linux that can run 32-bit programs.
