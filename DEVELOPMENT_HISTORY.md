# Development History

This is the story of how Mint Calendar CE got built — decisions made, dead ends hit, and
what's still open. For "what's different from Linux Mint's own Calendar," see
[CHANGELOG.md](CHANGELOG.md). For build instructions and the technical reference for every
compatibility fix, see [BUILD.md](BUILD.md).

## Rebasing onto GNOME Calendar 50

Linux Mint's own `gnome-calendar` package (`48.1+mint1`) is GNOME Calendar 48 plus one Mint
patch. CE rebases that onto upstream GNOME Calendar's tagged **v50.0** — two feature releases
newer — to bring in meeting attendee/organizer support, the refreshed sidebar and event-editor
UI, and GNOME's move to Blueprint (`.blp`) for UI definitions instead of hand-written `.ui`
XML. An earlier iteration tracked upstream `main` mid-development (`51.beta`); that was
replaced with the stable v50.0 tag before v1.0.

None of the v50.0 feature work itself is CE's doing — it's upstream GNOME development. What CE
added is getting that feature set to actually run on Mint 22.

## Porting v50.0 down to Mint 22 / Ubuntu 24.04

v50.0 expects GTK4 `>= 4.21.2` and libadwaita `>= 1.8~alpha` — newer than Mint 22 / Ubuntu
24.04 ship. Two alternatives were tried and ruled out before porting the code itself:

- `ppa:savoury1/gtk4` doesn't provide a newer GTK4 for Noble — confirmed via a real CI run, it
  404s for the Noble suite. That PPA's actual purpose is backporting an *old* GTK4 to ancient
  LTS releases, not providing a current one here.
- Building directly on **Ubuntu 26.04** worked — it proved the v50.0 code could build — but the
  resulting `.deb` couldn't install on an actual Mint 22 system, because it linked against
  newer libraries Mint 22 doesn't have. That stage is retained as development history, not the
  supported build target.

The approach that stuck: port the v50.0 source to build against Mint 22 / Ubuntu 24.04's actual
GTK4 4.14 / libadwaita 1.5 stack. That meant real per-widget substitutions everywhere v50.0
used a libadwaita widget that doesn't exist at this floor — `Adw.ButtonRow`, `Adw.ToggleGroup`,
`Adw.Spinner`, `Adw.InlineViewSwitcher`, the whole `Adw.ShortcutsDialog` shortcuts window, plus
a couple of removed GTK properties and one manually-backported API
(`gtk_filter_list_model_set_watch_items()`). The full list, file by file, is in BUILD.md's
[Compatibility fixes](BUILD.md#compatibility-fixes-from-the-v500-base) section — it was checked
against a clean upstream v50.0 checkout to confirm every file that differs maps to one of those
fixes, so nothing else was silently dropped porting it down. CI has since built the result
clean: 350/350 targets, all 10 tests passing.

## The blueprint-compiler gap

A separate, build-tool-only issue: Ubuntu 24.04 ships `blueprint-compiler` 0.12.0, and v50.0's
`.blp` files use the `not-swapped` signal flag across 30 files — that version's parser doesn't
recognize it. This isn't fork-specific; it's genuine upstream v50.0 usage. Since
blueprint-compiler only runs at build time (it compiles `.blp` → `.ui`, and nothing about which
version did that is retained in the installed app), the fix was a meson wrap pinning
blueprint-compiler 0.16.0, with every `find_program('blueprint-compiler', ...)` call requiring
`>= 0.14.0` so meson falls back to building the pinned version automatically when needed.

That surfaced two more packaging-only issues once CI actually reached the packaging step:
`dpkg-buildpackage` runs meson with `--wrap-mode=nodownload`, which blocks the wrap from being
fetched *mid-build* (fixed with an explicit `meson subprojects download` step in
`release-deb.yml`, run earlier while the job still has network access); and `debhelper`'s meson
support installs via plain `ninja install` rather than `meson install` (a still-open debhelper
limitation, Debian bug #1006805), which doesn't understand the `--skip-subprojects` flag needed
to keep the wrap's own files out of the package — fixed by having `debian/rules`'
`override_dh_auto_install` call `meson install --skip-subprojects blueprint-compiler` directly.

## Linux Mint's compatibility patch

Mint's one patch on top of stock GNOME Calendar — a desktop-environment-aware Settings
launcher, `gcal_utils_launch_gnome_settings()` in `src/utils/gcal-utils.c` — was carried
forward into the v50.0 base. It was checked directly against Mint's `48.1+mint1` source: the
function body is byte-for-byte identical.

## Package isolation & coexistence

CE is meant to install next to Mint's original `gnome-calendar`, not replace it, so every
system-visible identity was made independent rather than shared: binary name, application ID,
desktop entry, D-Bus service, GSettings schema and path, search-provider identity, AppStream
ID, gettext domain, and private data directory. The full before/after table is in
[BUILD.md](BUILD.md#independent-package-identity-and-coexistence). Internal `gcal_*` C symbols
and embedded `/org/gnome/calendar/...` resource-path strings were deliberately *not* mechanically
renamed — they're internal implementation detail, not installed filesystem paths, so they don't
create package-ownership conflicts either way.

One specific bug this surfaced: the release/development search-provider paths generated by the
`.ini` template and the ones registered/unregistered by the runtime D-Bus code had drifted out
of sync (a `DevelDevel`/`ceDevel` mismatch). Both now consistently use
`/org/mint/calendar/ce/SearchProvider` (release) and `/org/mint/calendar/ce/Devel/SearchProvider`
(development).

A `dpkg -L` file-list comparison against an installed `gnome-calendar` package, confirming zero
overlapping paths, is planned as part of release validation — see BUILD.md's
[coexistence test](BUILD.md#coexistence-test).

## Known issues / outstanding cleanup

- ~~**In-app icons broken**~~ — **fixed.** `src/gui/icons/icons.gresource.xml` declared the old
  `/org/gnome/calendar/icons/scalable/emblems/` prefix while `gcal_application_new()` sets
  `resource-base-path` to `/org/mint/calendar/ce`; GTK derives its icon search path from that
  base, so the ~15 in-app emblem icons (view switcher, reminders, share, etc.) couldn't
  resolve. Prefix corrected to match.
- ~~**Taskbar/window-manager icon mismatch**~~ — **fixed.** The desktop file's
  `StartupWMClass` was hardcoded to `mint-calendar-ce` rather than tracking the actual
  profile-aware application ID, and `g_set_prgname()` was never called at all. Both matter most
  for `-Dprofile=development` builds, where the real app ID carries a `.Devel` suffix the
  hardcoded value never accounted for. `StartupWMClass` is now generated from the same
  `application_id` meson variable used everywhere else, and `main.c` calls
  `g_set_prgname(APPLICATION_ID)` on startup.
- **Partial namespace isolation** — six `.gresource.xml` files (`event-editor`, `views`,
  `common`, `calendar-management`, `importer`, `theme`) plus their matching `.c`
  template-loader strings still reference `/org/gnome/calendar/...` internally. These are
  self-consistent — each file's declared prefix matches exactly what the corresponding C code
  requests — so nothing is broken, but it's incomplete against the goal of no leftover
  `gnome-calendar` namespace anywhere in the tree.
- **`debian/rules` execute bit** — has shown up as `0664` rather than `0755` in at least one
  source snapshot reviewed. `dpkg-buildpackage` needs it executable; likely just that snapshot
  losing the bit in transit, but worth confirming directly in the repository before a release
  build.
- **`debian/watch`** — intentionally not carried over from upstream, since CE isn't a straight
  repackaging of a watchable upstream release feed and the old file pointed at
  `gitlab.gnome.org`. Worth a deliberate decision either way rather than leaving it silently
  absent.
- **`gtk_filter_list_model_set_watch_items()` backport** — new code written for this port
  (`gcal_create_writable_calendars_model()` in `gcal-utils.c`), not carried over from anywhere.
  Flagged in BUILD.md as needing real lifecycle/leak scrutiny once it's been exercised.

## Translations

All 77 languages from GNOME Calendar's community translations carried over unchanged — the
compatibility work didn't touch translatable string text. "Mint Calendar CE" itself is
intentionally left untranslated, since the application name was never part of upstream's
translated string set.
