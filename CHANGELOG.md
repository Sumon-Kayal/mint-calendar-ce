# Changelog

## [1.0] - 2026-10-02

First release of **Mint Calendar CE** as an independently versioned project.

This release is based on **GNOME Calendar v50.0**, with Linux Mint's compatibility patch carried forward and the application ported to the GTK4/libadwaita versions available in **Linux Mint 22 / Ubuntu 24.04**.

### 1. Upstream Base

Mint Calendar CE rebases Linux Mint's maintained `gnome-calendar` package (`48.1+mint1`) onto the tagged **GNOME Calendar v50.0** stable release.

- Linux Mint's `48.1+mint1` package is based on GNOME Calendar 48 with Mint-specific patches.
- Mint Calendar CE uses upstream GNOME Calendar **v50.0**, two feature releases newer.
- An earlier iteration tracked upstream `main` during development (`51.beta`), but this was replaced with the stable v50.0 tag.

### 2. Linux Mint Compatibility Patch

Linux Mint's desktop-environment-aware Settings launcher patch was carried forward:

- Function: `gcal_utils_launch_gnome_settings`
- File: `src/utils/gcal-utils.c`
- The implementation was checked directly against Mint's `48.1+mint1` source.
- The function body is byte-for-byte identical to Mint's maintained version.

### 3. GTK4 / libadwaita Compatibility

GNOME Calendar v50.0 requires newer libraries than Linux Mint 22 / Ubuntu 24.04 provide:

- GTK4 `>= 4.21.2`
- libadwaita `>= 1.8~alpha`

Mint 22 / Ubuntu 24.04 instead provide the GTK4 4.14.x and libadwaita 1.5.0 stack targeted by this release.

Two alternative approaches were investigated and rejected:

- `ppa:savoury1/gtk4` does not provide a newer GTK4 for Noble; this was confirmed through CI.
- Building directly on Ubuntu 26.04 produced a `.deb` that could not install on an actual Mint 22 system.

Instead, the v50.0 feature set was ported to the libraries available on Mint 22 / Ubuntu 24.04.

#### Compatibility changes

- `Adw.ButtonRow` → `Adw.ActionRow`
  - 5 call sites, including the reminders section, event editor delete row, Add Calendar row, and edit-calendar export/remove rows.
- `Adw.ToggleGroup` / `Adw.Toggle` → linked `GtkToggleButton` pair
  - Used by the AM/PM picker and all-day/time-slot switch.
- `Adw.Spinner` → `Gtk.Spinner`
  - Used by the sync indicator and import dialog.
- `Adw.InlineViewSwitcher` → `Adw.ViewSwitcher` with `policy: narrow`.
- `Adw.ShortcutsDialog` → `Gtk.ShortcutsWindow`.
  - This also exposed that the `app.shortcuts` action was not wired to a handler.
  - Added `gcal_application_show_shortcuts()` so the shortcuts menu item works.
- `GtkListBox:tab-behavior` → removed for GTK4 4.14 compatibility.
- `GtkSettings:gtk-interface-reduced-motion` → removed.
  - Animation handling falls back to the older `gtk-enable-animations` setting.
- `gtk_filter_list_model_set_watch_items()` → backported.
  - The writable-calendars model now watches each calendar's `read-only` property directly through `GtkExpressionWatch`, retaining automatic refresh behavior.

The compatibility work was checked file-by-file against a clean upstream v50.0 checkout. Full technical details are documented in [BUILD.md](BUILD.md).

### 4. Blueprint Compiler

A real CI build exposed a separate build-tool compatibility issue.

Ubuntu 24.04 ships `blueprint-compiler` 0.12.0, while the v50.0 source uses the `not-swapped` signal flag across 30 `.blp` files.

The important distinction is that Blueprint Compiler is a **build-time dependency**. Its version does not become a runtime dependency of the installed application.

#### Fix

- Added [`subprojects/blueprint-compiler.wrap`](subprojects/blueprint-compiler.wrap).
- Pinned Blueprint Compiler to **v0.16.0** from GNOME GitLab.
- Updated every `find_program('blueprint-compiler')` call under `src/gui/` to require `>= 0.14.0`.
- Meson therefore falls back to the pinned v0.16.0 compiler whenever the system compiler is too old.
- Removed the now-unused `blueprint-compiler` Build-Depends from `debian/control`.
- Added `git`, `python3-gi`, and `gir1.2-gtk-4.0` to support fetching and running the wrapped compiler.

One consequence is that the first Meson configure now requires network access to fetch the Blueprint subproject when it is not already available locally.

#### Two more issues a follow-up CI run caught

The wrap fix above got the compile itself all the way to success — 350/350 targets, all 10
tests passing — but two packaging-step issues only showed up once that CI run actually reached
them:

- `dpkg-buildpackage` runs meson with `--wrap-mode=nodownload`, which blocks the wrap from
  being fetched mid-build (`nodownload` only blocks _downloading_, not _using_ an
  already-fetched wrap). Added an explicit `meson subprojects download` step to
  `release-deb.yml`, run before `dpkg-buildpackage`, while the job still has network access.
- `debhelper`'s meson support installs via plain `ninja install` rather than `meson install`
  (a known, still-open debhelper limitation — Debian bug #1006805), but tries to pass
  `--skip-subprojects blueprint-compiler` along with that call anyway, to keep the wrap
  subproject's own files out of this package. Plain `ninja` has never supported that flag, so
  the combination failed outright (`unrecognized option '--skip-subprojects'`). Fixed by
  having `debian/rules`' `override_dh_auto_install` call `meson install --skip-subprojects
blueprint-compiler` directly instead of going through `dh_auto_install`'s default path.

### 5. Rebranding

Mint Calendar CE now has its own application identity.

- Application ID: `org.mint.calendar.ce`
- Name: **Mint Calendar CE**
- Own GSettings schema: `org.mint.calendar.ce`
- Own icons and desktop launcher.
- Replaced upstream's purple gradient icon with a flat, Mint-Y-Dark-inspired design using a dark neutral body and Mint green accent.

The independent GSettings schema prevents configuration from being shared with stock GNOME Calendar if both applications are installed.

### 6. Packaging

Mint Calendar CE 1.0 ships as a native **`.deb` package**.

Changes include:

- Removed upstream Flatpak packaging: `build-aux/org.gnome.Calendar.json`.
- Removed the runtime Flatpak-sandbox check from the About dialog.
- No Snap packaging existed upstream, so nothing was required there.
- CI builds and publishes the `.deb` on every `v*` tag push.
- CodeQL security scanning runs on every change.
- Both workflows use a plain `ubuntu-24.04` runner to match the real Mint 22 / Ubuntu 24.04 target environment.
- No PPA or alternate Ubuntu base is required.

### 7. Translations

All **77 languages** from GNOME Calendar's community translations are carried over unchanged.

- Compatibility changes did not alter translatable string text.
- Existing translations therefore remain aligned with the current source.
- The application name, **Mint Calendar CE**, is intentionally not translated because the application name was not part of GNOME Calendar's translated string set.

### 8. Release Comparison

|                                | Linux Mint `gnome-calendar` 48.1+mint1 | GNOME Calendar 50                   | Mint Calendar CE 1.0         |
| ------------------------------ | -------------------------------------- | ----------------------------------- | ---------------------------- |
| Based on                       | GNOME Calendar 48                      | GNOME Calendar 50                   | GNOME Calendar 50            |
| Linux Mint compatibility patch | Yes                                    | No                                  | Yes                          |
| Branding                       | Stock GNOME                            | Stock GNOME                         | Mint-branded                 |
| Mint-Y-Dark-inspired icon      | No                                     | No                                  | Yes                          |
| Runs on current Mint 22        | Yes                                    | No — requires newer GTK4/libadwaita | Yes                          |
| Own GSettings schema           | —                                      | —                                   | Yes (`org.mint.calendar.ce`) |
| Official Mint project          | Yes                                    | —                                   | No — independent/community   |
| Distribution                   | Mint repositories                      | GNOME / Flathub                     | Project releases             |

### Summary

Mint Calendar CE 1.0 combines the **GNOME Calendar 50 feature set** with Linux Mint's compatibility patch and a complete compatibility port to the GTK4/libadwaita versions available on **Linux Mint 22 / Ubuntu 24.04**.

It is independently branded, independently versioned, packaged as a native `.deb`, and retains the existing GNOME Calendar translations.
