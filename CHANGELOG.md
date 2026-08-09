# Changelog

## [1.0] - 2026-08-02

First release of Mint Calendar CE as its own independently-versioned project. This entry
describes what changed relative to **Linux Mint's own maintained `gnome-calendar` package,
version `48.1+mint1`** — the baseline this fork started from.

### Feature base: GNOME Calendar 48 → 50

Mint's `48.1+mint1` package is GNOME Calendar 48 with Mint's own patches on top. This fork
rebases onto upstream GNOME Calendar's tagged **v50.0** stable release instead — two feature
releases newer — while keeping Mint's own patch (below). An earlier iteration of this project
tracked upstream's `main` branch mid-development (`51.beta`) rather than a tagged release;
that was dropped in favor of the stable v50.0 tag.

### Mint's compatibility patch: carried forward, verified

Reapplied Mint's desktop-environment-aware Settings launcher patch
(`gcal_utils_launch_gnome_settings` in `src/utils/gcal-utils.c`) on top of the new v50.0 base.
Checked directly against Mint's `48.1+mint1` source: the function body is byte-for-byte
identical to Mint's own version.

### Making v50 actually run on Mint 22 / Ubuntu 24.04

This is the part that took the most real work. GNOME Calendar v50.0 needs GTK4 `>= 4.21.2` and
libadwaita `>= 1.8~alpha` — neither of which Mint 22 / Ubuntu 24.04 (Noble) ship, and Mint's own
`48.1+mint1` predates entirely (it's built against the older GTK4/libadwaita those releases do
have). Two other routes were tried and ruled out first: `ppa:savoury1/gtk4` doesn't provide a
newer GTK4 for Noble (confirmed via a real CI run — it 404s for that suite; its actual purpose
is backporting an *old* GTK4 to ancient LTS releases), and building against Ubuntu 26.04
directly worked but produced a `.deb` that couldn't install on actual Mint 22.

Instead, the v50.0 feature set was ported down to Mint 22 / Ubuntu 24.04's real GTK4 (4.14.x)
and libadwaita (1.5.0), matching what Mint's own `48.1+mint1` package already targets:

- `Adw.ButtonRow` → `Adw.ActionRow` (5 call sites: reminders section, event editor's delete
  row, "Add Calendar" row, edit-calendar page's export/remove rows)
- `Adw.ToggleGroup` / `Adw.Toggle` → a linked `GtkToggleButton` pair (AM/PM picker, all-day /
  time-slot switch)
- `Adw.Spinner` → `Gtk.Spinner` (sync indicator, import dialog)
- `Adw.InlineViewSwitcher` → `Adw.ViewSwitcher` with `policy: narrow`
- `Adw.ShortcutsDialog` → plain `Gtk.ShortcutsWindow` — which also surfaced that the
  `app.shortcuts` action was never wired to a handler at all; added
  `gcal_application_show_shortcuts()` so the shortcuts menu item actually works
- `GtkListBox:tab-behavior` and `GtkSettings:gtk-interface-reduced-motion` — removed; the
  latter falls back to the older, broader `gtk-enable-animations` check already present
- `gtk_filter_list_model_set_watch_items()` — backported rather than dropped: the
  writable-calendars model now watches each calendar's `read-only` property directly via
  `GtkExpressionWatch` (core GTK4 API since 4.0), so it still auto-refreshes when a calendar's
  read-only flag changes in place

Full technical detail in [BUILD.md](BUILD.md). Checked file-by-file against a clean upstream
v50.0 checkout to confirm this list is complete and nothing else was silently dropped in the
process.

### blueprint-compiler: a build-tool version gap, caught by a real CI run

Source-level checking has its limits — this one only showed up once an actual CI build ran.
Ubuntu 24.04 ships `blueprint-compiler` 0.12.0, whose parser doesn't recognize the
`not-swapped` signal flag used throughout the codebase (30 `.blp` files use it — this comes
straight from upstream v50.0, not from anything this fork changed). `ninja` failed on its very
first blueprint-compiling step, before any C code built at all.

Unlike GTK4/libadwaita, `blueprint-compiler` only runs at build time — nothing about which
version compiled a `.blp` file is retained in the installed app — so the fix didn't need to
touch any `.blp` source. Added a meson wrap
([`subprojects/blueprint-compiler.wrap`](subprojects/blueprint-compiler.wrap)) pinning a
known-good v0.16.0 fetched from GNOME's GitLab, and required `>= 0.14.0` on every
`find_program('blueprint-compiler')` call across `src/gui/`'s `meson.build` files, so meson
automatically falls back to the pinned version wherever the system one is too old — this is
the pattern blueprint-compiler's own documentation recommends for exactly this situation.
`debian/control` updated to match: dropped the now-unused `blueprint-compiler` Build-Depend,
added `git` (to fetch the wrap) and `python3-gi` / `gir1.2-gtk-4.0` (blueprint-compiler's own
runtime dependencies). One real consequence: the build now needs network access on first
configure to fetch the wrap, which wasn't required before.

### Rebrand

Application ID, icons, and desktop launcher entry moved to `org.mint.calendar.ce` /
**"Mint Calendar CE"**, with a flat, Mint-Y-Dark-inspired icon palette (dark neutral body,
Mint green accent) in place of upstream's purple gradient. Gives this fork its own GSettings
schema (`org.mint.calendar.ce`) rather than reusing upstream's `org.gnome.calendar`, so
settings stay independent if both this app and stock GNOME Calendar are ever installed on the
same system.

### Packaging

Ships as a native `.deb` only — Flatpak packaging (`build-aux/org.gnome.Calendar.json`) and
the runtime Flatpak-sandbox check in the About dialog were removed; upstream never shipped a
Snap, so there was nothing to remove there. CI builds and publishes the `.deb` on every `v*`
tag push, with CodeQL security scanning on every change, both running on a plain `ubuntu-24.04`
runner — matching real Mint 22 / Ubuntu 24.04, no PPA or alternate base needed.

### Translations

All 77 languages from GNOME Calendar's own community translations are carried over unchanged.
None of the compatibility work above touched any translatable string text — only the
surrounding widget markup — so every existing translation still matches the current source
exactly. The one string that isn't translated is the app's own name ("Mint Calendar CE" in the
title bar / launcher), which was never part of GNOME Calendar's translated string set even
before this fork existed.

## How this compares

| | Linux Mint's `gnome-calendar` (48.1+mint1) | GNOME Calendar (stock) | Mint Calendar CE |
|---|---|---|---|
| Based on | GNOME Calendar 48 | GNOME Calendar 50 | GNOME Calendar 50 |
| Linux Mint's compatibility patch | Yes | No | Yes |
| Branding | Stock GNOME | Stock GNOME | Mint-branded, Mint-Y-Dark icon |
| Runs on current Mint 22 | Yes | No (needs newer GTK4/libadwaita) | Yes |
| Own GSettings schema | — | — | Yes (`org.mint.calendar.ce`) |
| Official Mint project | Yes | — | No — independent/community |
| Where to get it | Mint's own repos | GNOME / Flathub | This project's [Releases page](https://github.com/Sumon-Kayal/mint-calendar-ce/releases) |

In short: GNOME Calendar 50's feature set, with the same Mint compatibility patch Mint's own
48.1 package uses, ported to actually run on Mint 22 today, under independent Mint branding.
