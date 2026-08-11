<div align="center">
  <img src="data/icons/hicolor/scalable/apps/org.mint.calendar.ce.svg" width="128" height="128">

  # Mint Calendar CE 
  #      v1.0

  Using the latest stable GNOME release as a base, with patches from Linux Mint's version of
  GNOME Calendar applied on top.

  <img width="922" src="https://static.gnome.org/appdata/gnome-50/org.gnome.Calendar/month-view.png">
</div>

A community-maintained, Linux Mint-flavored fork of [GNOME Calendar](https://apps.gnome.org/Calendar/),
distributed independently — not an official Linux Mint or GNOME project.

## How this compares

| | Linux Mint's `gnome-calendar` | GNOME Calendar (stock) | Mint Calendar CE |
|---|---|---|---|
| Based on | GNOME Calendar 48 | GNOME Calendar 50 | GNOME Calendar 50 |
| Linux Mint's compatibility patch | Yes | No | Yes |
| Branding | Stock GNOME | Stock GNOME | Mint-branded, Mint-Y-Dark icon |
| Official Mint project | Yes | — | No — independent/community |
| Where to get it | Mint's own repos | GNOME / Flathub | This project's [Releases page](https://github.com/Sumon-Kayal/mint-calendar-ce/releases) |

In short: the newer GNOME Calendar, with the same Mint compatibility patch Mint's own package
uses, under independent Mint branding.

## Installing

Distributed as a `.deb`. Grab the latest from the
[Releases page](https://github.com/Sumon-Kayal/mint-calendar-ce/releases) and install it with:

```sh
sudo apt install ./mint-calendar-ce_*.deb
```

The v50.0 feature set has been ported to build against Mint 22 / Ubuntu 24.04's own GTK4
and libadwaita stack directly — no PPA, container, or newer base is required. See
[BUILD.md](BUILD.md) for the compatibility work, dependency list, and build details. See [CHANGELOG.md](CHANGELOG.md) for the project history and release notes.

The package is designed to coexist with Linux Mint's original `gnome-calendar` package.
Mint Calendar CE uses its own executable, application ID, desktop entry, D-Bus service,
GSettings schema, AppStream metadata, gettext domain, and private data directory, so it
does not intentionally replace or claim the original package's installed files.

The build and release automation — GitHub Actions building the `.deb`, validating package
ownership/identity, and publishing tagged releases, with CodeQL security scanning on every
change — runs on an `ubuntu-24.04` runner.


## Package coexistence

Mint Calendar CE is packaged as an independent application rather than as a replacement
for Linux Mint's `gnome-calendar`.

Its system-visible identities are kept separate:

| Component | Mint Calendar CE |
|---|---|
| Package | `mint-calendar-ce` |
| Binary | `mint-calendar-ce` |
| Application ID | `org.mint.calendar.ce` |
| Desktop entry | `org.mint.calendar.ce.desktop` |
| D-Bus service | `org.mint.calendar.ce.service` |
| GSettings schema | `org.mint.calendar.ce` |
| AppStream ID | `org.mint.calendar.ce` |
| Private data directory | `mint-calendar-ce` |

The development search-provider path is `/org/mint/calendar/ce/Devel/SearchProvider`;
release builds use `/org/mint/calendar/ce/SearchProvider`.

This separation is intended to allow CE and the original GNOME Calendar package to remain
installed at the same time. Final releases are still validated by installing the generated
`.deb` alongside the distro package and checking for overlapping installed paths.

## Links

- Homepage: <https://github.com/Sumon-Kayal/mint-calendar-ce>
- Report an issue: <https://github.com/Sumon-Kayal/mint-calendar-ce/issues>
- Changelog: [CHANGELOG.md](CHANGELOG.md)
- Contact: <sumankayalsuman4@proton.me>

## Credits

Based on [GNOME Calendar](https://gitlab.gnome.org/GNOME/gnome-calendar), with
[Linux Mint's compatibility patch](https://gitlab.com/linuxmint/pins/mint/gnome-calendar) carried
forward. All credit for the underlying app goes to the GNOME Calendar authors and the Linux
Mint team.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) — including this fork's policy on AI-assisted
contributions.

## FAQ

**What is Mint Calendar CE?**
A community-maintained fork of GNOME Calendar for Linux Mint. It's based on GNOME Calendar's
latest stable release, carries forward Linux Mint's own compatibility patch, and has
independent Mint-Y-Dark branding.

**Is this an official Linux Mint project?**
No. It's independent and community-maintained, not affiliated with or endorsed by the Linux
Mint team.

**How is this different from Mint's own `gnome-calendar` package, or from plain GNOME Calendar?**
See "How this compares" above — short version: it is based on the newer GNOME Calendar,
carries forward the Mint compatibility work, and is distributed under independent branding
and package identities. It is designed to install alongside Mint's original
`gnome-calendar` package.

**How do I install it?**
As a `.deb` from the [Releases page](https://github.com/Sumon-Kayal/mint-calendar-ce/releases) — see "Installing" above.

**Is there a Flatpak or Snap version?**
No, and none is planned. This project distributes as a native `.deb` only.

**Will this work on Linux Mint 22?**
That's the target platform. The compatibility work targets the GTK4 and libadwaita versions
provided by Mint 22 / Ubuntu 24.04. See [BUILD.md](BUILD.md) if you're building it yourself.

**Has this actually been built and tested?**
Yes. The current development build has completed the full build/test targets and the
package-isolation checks are part of the validation workflow. The final release `.deb`
should still be tested on a real Linux Mint installation before being promoted as a live
release.

**Why does it say v1.0 instead of matching GNOME Calendar's version number?**
It versions independently now, since it's its own distributed product rather than a straight
rebuild of upstream. The GNOME Calendar version it's based on is documented in "How this
compares" instead.

**Will it get updated when a new GNOME Calendar version comes out?**
Check the [Releases page](https://github.com/Sumon-Kayal/mint-calendar-ce/releases)
for the current version — there's no fixed schedule to point to yet.

**Can I contribute?**
Yes — see [CONTRIBUTING.md](CONTRIBUTING.md).

**Can I use AI tools for a contribution?**
Yes, if disclosed in the pull request. See [CONTRIBUTING.md](CONTRIBUTING.md) for what to
include — acceptance is still up to the maintainer.

**Where do I report a bug or get in touch?**
See "Links" above.
