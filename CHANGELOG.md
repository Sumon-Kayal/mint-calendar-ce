# Changelog

## [1.0] — Unreleased

First independent release. Built on GNOME Calendar 50.0, with Linux Mint's own compatibility
patch carried forward, ported to run on Linux Mint 22 / Ubuntu 24.04's own GTK4/libadwaita
stack, and packaged to install alongside Mint's original `gnome-calendar`.

### New, compared with Linux Mint's `gnome-calendar` 48.1+mint1

- **Meeting attendees & organizer support** — attendee list, RSVP-style rows, and
  attendee/organizer details in the event editor
- **Refreshed interface** — newer calendar sidebar, toolbar, and shared date/calendar-picker
  widgets
- **Runs on Mint 22 / Ubuntu 24.04 as-is** — no PPA, container, or newer Ubuntu base needed; the
  newer feature set was ported down to Mint 22 / Ubuntu 24.04's own GTK4/libadwaita versions
  (not yet manually validated on real Mint 22, validated on Ubuntu 24.04 CI runners)
- **Installs alongside the original package** — separate binary, application ID, D-Bus
  service, GSettings schema, search provider, and data directory (see table below)
- **Mint Calendar CE branding** — new name, new Mint-Y-Dark-inspired icon
- **Automated builds** — GitHub Actions CI with CodeQL security scanning on pushes/PRs to main,
  weekly schedule, and manual dispatch

### Kept

- Linux Mint's desktop-environment-aware Settings launcher patch, carried forward unchanged
- All 77 existing translation languages

### Identity

| Component | Linux Mint's `gnome-calendar` | Mint Calendar CE |
|---|---|---|
| Binary | `gnome-calendar` | `mint-calendar-ce` |
| Application ID | `org.gnome.Calendar` | `org.mint.calendar.ce` |
| D-Bus service | `org.gnome.Calendar.service` | `org.mint.calendar.ce.service` |
| GSettings schema | `org.gnome.Calendar` | `org.mint.calendar.ce` |
| Search provider | `org.gnome.Calendar.search-provider.ini` | `org.mint.calendar.ce.search-provider.ini` |
| Data directory | `gnome-calendar` | `mint-calendar-ce` |
| Versioning | Tracks upstream/Mint (`48.1+mint1`-style) | Independent, starting at `1.0` |

---

For the full technical story — why GNOME Calendar 50, the compatibility porting work, build/CI
decisions, and outstanding cleanup — see [DEVELOPMENT_HISTORY.md](DEVELOPMENT_HISTORY.md).
