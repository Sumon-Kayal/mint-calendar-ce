# Building from Source

This covers building Mint Calendar CE yourself — either a local dev build, or
a `.deb` for Linux Mint Cinnamon. For coding style, see [HACKING.md](HACKING.md); for the
contribution process, including how AI-assisted contributions are handled, see
[CONTRIBUTING.md](CONTRIBUTING.md).

## Version requirements

This tree is rebased onto upstream GNOME Calendar's tagged **v50.0 stable** release (it
previously tracked upstream's `main` branch mid-development, at `51.beta` — see
[debian/changelog](debian/changelog)). v50.0 itself needs GTK4 `>= 4.21.2` / libadwaita
`>= 1.8~alpha` — newer than Linux Mint 22 / Ubuntu 24.04 (Noble) ship out of the box. Two
things were tried and ruled out before landing on the current approach:
[`ppa:savoury1/gtk4`](https://launchpad.net/~savoury1/+archive/ubuntu/gtk4) doesn't work
(confirmed via a real CI run — it 404s for the Noble suite; its actual purpose is backporting
an *old* GTK4 to ancient LTS releases, not providing a current one for Noble), and building on
Ubuntu 26.04 instead worked but meant the resulting `.deb` couldn't install on actual Mint 22.

**Current approach: the v50.0 feature set is ported down to Mint 22 / Ubuntu 24.04's real
libraries**, instead of chasing a newer base:

| Dependency | Required    | Ships in Mint 22 / Ubuntu 24.04 (Noble)? |
|------------|-------------|-------------------------------------------|
| GTK4       | `>= 4.14.2` | Yes — noble ships 4.14.2, noble-updates 4.14.5 |
| libadwaita | `>= 1.5.0`  | Yes — noble ships 1.5.0 |
| GLib       | `>= 2.80.0` | Yes — noble ships 2.80.0 |
| fribidi    | any (powers the attendee-list bidirectional-text support) | Yes |

Checked directly against Ubuntu's package archive: no PPA, container, or newer base needed.
libical, evolution-data-server (`libecal2.0-dev` and friends), gweather, and geoclue's floors
were already satisfied at this level and are unchanged. The full, versioned dependency list
lives in [`meson.build`](meson.build)'s `dependency()` calls and in
[`debian/control`](debian/control)'s `Build-Depends`; both are kept in sync.

## Compatibility fixes from the v50.0 base

Getting the v50.0 feature set to build against this older GTK4/libadwaita floor took real
per-widget compatibility fixes, not just lowering the version numbers — several libadwaita
widgets the newer code used don't exist yet at this floor:

- `Adw.ButtonRow` (5 call sites, across the reminders section, the event editor's delete row,
  the calendar-management "Add Calendar" row, and the edit-calendar page's export/remove rows)
  → `Adw.ActionRow` with `activatable: true`, using `[prefix]`/`[suffix]` `Image` children for
  icons ButtonRow set as plain properties
- `Adw.ToggleGroup` / `Adw.Toggle` (the event editor's AM/PM period picker, and the all-day /
  time-slot switch) → a linked `GtkToggleButton` pair
- `Adw.Spinner` (the sync indicator and the import dialog's loading placeholder) → `Gtk.Spinner`
  with `spinning: true`
- `Adw.InlineViewSwitcher` (the narrow-window view switcher) → `Adw.ViewSwitcher` with
  `policy: narrow`, which has shipped this same adaptive behavior since early libadwaita 1.x
- `Adw.ShortcutsDialog` / `Adw.ShortcutsItem` / `Adw.ShortcutsSection` (the whole keyboard
  shortcuts window) → rebuilt on plain `Gtk.ShortcutsWindow`. This also surfaced that the
  `app.shortcuts` action the header menu already called was never actually implemented
  anywhere — added `gcal_application_show_shortcuts()` in `gcal-application.c` so the menu
  item works.
- `GtkListBox:tab-behavior` (the quick-add popover's calendar list) → removed; falls back to
  GTK's default tab-focus behavior
- `GtkSettings:gtk-interface-reduced-motion` (month-view scroll animation) → removed; the code
  already had an older `gtk-enable-animations` fallback read right next to it, which is now the
  only check (see the comment in `gcal-month-view.c`)
- `gtk_filter_list_model_set_watch_items()` (used so the writable-calendars combo-row model
  auto-refreshes when a calendar's read-only flag flips in place) → **backported rather than
  dropped**: `gcal_create_writable_calendars_model()` in `gcal-utils.c` now manually watches
  each calendar's `read-only` property via `GtkExpressionWatch` (core GTK4 API since 4.0) and
  re-filters when it changes, adding/removing watches as calendars come and go. This is new
  code written for this port, not carried over from anywhere upstream — it hasn't been
  exercised by an actual run yet, so give it real scrutiny (leak/lifecycle correctness
  especially) before relying on it.
- The AppStream metainfo had a `<url type="contact">mailto:…</url>` entry, which
  `appstreamcli validate` rejects — removed; the homepage/bugtracker/vcs-browser URLs cover it

These are checked against the current source (no leftover references to the newer widgets
remain — verified both by search and by diffing this tree's `src/` against a clean upstream
v50.0 checkout, file by file) and the feature-parity question was checked deliberately: every
file that differs from upstream maps to one of the items above, so nothing else should have
been silently dropped in the process. The compatibility port has since been exercised by CI:
the build reached **350/350 targets with all 10 tests passing**. The remaining release gate is
the real Mint installation/coexistence test of the generated `.deb`.

## blueprint-compiler needs to be newer than Ubuntu 24.04 ships

This one isn't a library version issue like the ones above — it's the *build tool*.
Ubuntu 24.04 ships `blueprint-compiler` 0.12.0, and this codebase's `.blp` files use the
`not-swapped` signal flag, which that version's parser doesn't recognize (`error: Expected
';'` right after the flag — confirmed by an actual CI run). This affects the whole codebase,
not anything specific to this fork: genuine upstream v50.0 uses `not-swapped` in 30 files.

Since `blueprint-compiler` only runs at build time — it compiles `.blp` to `.ui` XML, and
nothing about which version did that compiling is retained in the installed app — there's no
need to match it to the target system the way GTK4/libadwaita do. Fixed the way
blueprint-compiler's own docs recommend for exactly this situation: a meson wrap
(`subprojects/blueprint-compiler.wrap`) pins a known-good version (v0.16.0) fetched from
GNOME's GitLab, and every `find_program('blueprint-compiler', ...)` call across `src/gui/`'s
`meson.build` files now requires `>= 0.14.0`, so meson automatically falls back to building
the pinned version whenever the system one is too old. Also added `git` (to fetch the wrap)
and `python3-gi` / `gir1.2-gtk-4.0` (blueprint-compiler's own runtime dependencies) to
`debian/control`'s Build-Depends, and dropped the plain `blueprint-compiler` entry there since
it's not actually what gets used anymore.

One more knock-on fix: `debhelper`'s meson support still installs via plain `ninja install`
rather than `meson install` (a known, still-open debhelper limitation — Debian bug #1006805),
but it *also* tries to pass `--skip-subprojects blueprint-compiler` along with that call, to
keep the wrap subproject's own files out of this package. Plain `ninja` has never had that
flag, so the combination fails outright. `debian/rules`' `override_dh_auto_install` now calls
`meson install --skip-subprojects blueprint-compiler` directly instead of going through
`dh_auto_install`'s default path.

One real consequence: the build now needs network access to fetch the wrap subproject. For a
plain `meson setup`, this happens automatically on first configure. `dpkg-buildpackage` is
different: it runs meson with `--wrap-mode=nodownload` (a deliberate safety default, so package
builds can't reach out to arbitrary git URLs mid-build), which blocks the automatic fetch —
`nodownload` only blocks *downloading*, not *using* a wrap already on disk. `release-deb.yml`
handles this with an explicit `meson subprojects download` step before `dpkg-buildpackage` runs,
while the job still has network access; do the same locally before a `.deb` build (`## Debian
package build` below), or run it once and vendor the resulting `subprojects/blueprint-compiler/`
directory for a fully offline build.

## Its own GSettings schema

This fork uses its own GSettings schema (`org.mint.calendar.ce`, path
`/org/mint/calendar/ce/`) instead of reusing upstream's `org.gnome.calendar`. If both this
app and real GNOME Calendar are ever installed on the same system, they now have independent
settings rather than silently sharing one GSettings key space.

## Independent package identity and coexistence

Mint Calendar CE is intended to be installable alongside Linux Mint's original
`gnome-calendar` package. System-visible identities are deliberately separate:

| Resource | Original GNOME Calendar | Mint Calendar CE |
|---|---|---|
| Binary | `gnome-calendar` | `mint-calendar-ce` |
| Application ID | `org.gnome.Calendar` | `org.mint.calendar.ce` |
| Desktop entry | `org.gnome.Calendar.desktop` | `org.mint.calendar.ce.desktop` |
| D-Bus service | `org.gnome.Calendar.service` | `org.mint.calendar.ce.service` |
| Search provider | `org.gnome.Calendar.search-provider.ini` | `org.mint.calendar.ce.search-provider.ini` |
| GSettings schema | `org.gnome.Calendar` | `org.mint.calendar.ce` |
| GSettings path | `/org/gnome/calendar/` | `/org/mint/calendar/ce/` |
| AppStream ID | `org.gnome.Calendar` | `org.mint.calendar.ce` |
| gettext domain | `gnome-calendar` | `mint-calendar-ce` |
| Private data directory | `gnome-calendar` | `mint-calendar-ce` |

The internal `gcal_*` C symbols and embedded `/org/gnome/calendar/...` resource
paths are not mechanically renamed. They are internal implementation/resource
identifiers and do not by themselves create Debian filesystem ownership conflicts.

### Search-provider profile paths

The search-provider object path is kept identical between the generated metadata
and the runtime D-Bus registration code:

```text
Release:     /org/mint/calendar/ce/SearchProvider
Development: /org/mint/calendar/ce/Devel/SearchProvider
```

Both registration and unregistration use the same development profile path, and
the `.ini` template uses the matching `profile_path`.

### Coexistence test

After building the `.deb`, install the distro GNOME Calendar first, then install CE:

```sh
sudo apt install gnome-calendar
sudo apt install ./mint-calendar-ce_*.deb
```

Compare the package file lists:

```sh
dpkg -L gnome-calendar | grep -v '/$' | sort > /tmp/gnome-calendar-sorted.txt
dpkg -L mint-calendar-ce | grep -v '/$' | sort > /tmp/mint-calendar-ce-sorted.txt

comm -12 /tmp/gnome-calendar-sorted.txt /tmp/mint-calendar-ce-sorted.txt
```

Expected output: **nothing**.

Then launch both applications independently and verify desktop activation,
D-Bus activation, search integration, and uninstalling CE without damaging the
original GNOME Calendar package.

## Dev build (meson)

For local development or testing, without producing a `.deb`:

```sh
meson setup builddir
ninja -C builddir
meson test -C builddir
```

Install into a prefix with `ninja -C builddir install`, or run uninstalled via
`meson devenv -C builddir`. Run the test suite before opening a pull request, per
[CONTRIBUTING.md](CONTRIBUTING.md).

A few build options worth knowing about (see [`meson_options.txt`](meson_options.txt) for the
full list):

```sh
# Development profile build (installs alongside a stable version, distinct app ID/icon)
meson setup builddir -Dprofile=development

# Enable extra debugging/tracing output
meson setup builddir -Dtracing=true
```

## Debian package build

This is what the release workflow runs on every `v*` tag push, on a plain `ubuntu-24.04`
runner — no Distrobox or alternate base needed anymore:

```sh
# Packaging tools
sudo apt-get install -y build-essential devscripts equivs dpkg-dev fakeroot

# Pull in this package's Build-Depends (see debian/control)
sudo mk-build-deps --install --remove \
  --tool="apt-get -y --no-install-recommends" debian/control

# dpkg-buildpackage runs meson with --wrap-mode=nodownload, so fetch the
# blueprint-compiler wrap now, while network access is still fine to use
meson subprojects download

# Build
dpkg-buildpackage -us -uc -b

# Result lands one directory up
ls ../*.deb
```

Every package in `debian/control`'s Build-Depends is available directly from Ubuntu 24.04's
`main` and `universe` components (evolution-data-server's `-dev` packages are in `universe` —
make sure that component is enabled, which it is by default on a standard Mint/Ubuntu desktop
install). If `mk-build-deps` still reports it can't install the dummy `-build-deps` package,
run `dpkg-checkbuilddeps` directly from the source root — its error names the specific package
and version it couldn't satisfy, which is more useful than mk-build-deps's own summary line.

Install the resulting package with:

```sh
sudo apt install ./mint-calendar-ce_*.deb
```

## Icons and branding

The application icon, launcher entry (`data/org.mint.calendar.ce.desktop.in.in`), and app ID
(`org.mint.calendar.ce`, set in `meson.build`) were restyled for this fork —
flat, Mint-Y-Dark-inspired palette in place of upstream's gradient purple. The symbolic icons
(`data/icons/hicolor/symbolic/`) were left as generic single-fill icons deliberately: GTK
recolors symbolic icons automatically based on the active theme, so hardcoding a dark-mode
color into them would actually break that adaptive behavior rather than help it.

If you have access to Linux Mint's actual Mint-Y-Dark theme assets and want a closer palette
match than this best-effort pass, `data/icons/hicolor/scalable/apps/` is where to start — see
the comments at the top of the app icon's `.svg` file.

## Before opening a release

This rebase was carried out at the source level: cross-checked against Linux Mint's own
`gnome-calendar` pin for the one behavioral patch that matters (the desktop-environment-aware
Settings launcher in `src/utils/gcal-utils.c` — confirmed byte-for-byte identical to Mint's
48.1+mint1 version by diffing both directly), and checked file-by-file against a clean upstream
v50.0 checkout to confirm the compatibility fixes above are complete and nothing else was
dropped in the process. The GTK4/libadwaita version floor is now confirmed satisfiable on
Mint 22 / Ubuntu 24.04 directly (table above, checked against Ubuntu's package archive).

CI has now exercised the compatibility build successfully: the tree reached
350/350 build targets and all 10 tests passed. CodeQL and the other repository workflows are
already used as the normal automated checks.

Before publishing a release, perform the real Linux Mint coexistence test described above.
The strongest package-level proof is that the generated `.deb` has no filesystem path in
common with the installed `gnome-calendar` package (compare the filtered, sorted output of
`dpkg -L` for both packages using `comm`, as shown above), and that both applications can
be launched and uninstalled independently.
