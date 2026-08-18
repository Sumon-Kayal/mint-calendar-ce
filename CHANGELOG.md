Mint Calendar CE 1.0 — What's New Compared with Linux Mint GNOME Calendar

Overview

Mint Calendar CE is a community-maintained continuation of Linux Mint's GNOME Calendar, updated from the Mint-maintained GNOME Calendar 48.1+mint1 base to GNOME Calendar v50.0 and adapted to run on the GTK4/libadwaita versions available in Linux Mint 22 / Ubuntu 24.04.

This document focuses specifically on:

«What is new or different in Mint Calendar CE 1.0 compared with Linux Mint's own "gnome-calendar" 48.1+mint1?»

Not every difference between the two source trees is CE development work. A significant portion comes from GNOME Calendar's own development between versions 48 and 50.

---

1. At a Glance

Area| Linux Mint "gnome-calendar" 48.1+mint1| Mint Calendar CE 1.0
Feature base| GNOME Calendar 48| GNOME Calendar 50
Mint compatibility patch| Yes| Carried forward
Application name| GNOME Calendar| Mint Calendar CE
Binary| "gnome-calendar"| "mint-calendar-ce"
Application ID| "org.gnome.Calendar"| "org.mint.calendar.ce"
GSettings schema| "org.gnome.Calendar"| "org.mint.calendar.ce"
D-Bus identity| GNOME Calendar| Independent CE identity
Search provider| GNOME Calendar| Independent CE search provider
Versioning| Upstream/Mint based| Independent CE versioning — 1.0
GTK4 target| Mint 22 stack| Mint 22 stack with v50 compatibility port
libadwaita target| Mint 22 stack| Mint 22 stack with v50 compatibility port
Application icon| Upstream/Mint| Mint-branded icon
Flatpak| Available upstream| Removed — ".deb" focused
CI| Mint GitLab infrastructure| GitHub Actions
CodeQL| No CE workflow| Enabled
Blueprint Compiler| System version| Pinned fallback to 0.16.0 when required
Coexistence with original package| Not applicable| Designed to coexist

---

2. The Biggest Change: GNOME Calendar 50 Features

The biggest functional difference is that Mint Calendar CE moves the application from the Mint-maintained GNOME Calendar 48.1 codebase to GNOME Calendar v50.0.

Therefore, CE brings the upstream v48 → v50 development into the Mint 22 environment.

These changes were not invented by CE, but they are new to users coming from Mint's 48.1+mint1 Calendar.

New meeting attendee and organizer support

CE inherits the newer attendee/organizer functionality from GNOME Calendar 50, including:

- event attendee support;
- event organizer support;
- attendee rows;
- attendee summary;
- attendee details;
- attendee management sections;
- organizer information;
- improved bidirectional text handling.

New components include:

gcal-event-attendee.*
gcal-event-organizer.*
gcal-attendee-row
gcal-attendee-summary-row
gcal-attendee-details-page
gcal-attendees-section
gcal-organizer-row
gcal-bidi.c

These capabilities were not part of the Mint 48.1+mint1 baseline.

---

3. Newer Calendar UI Architecture

CE inherits the newer GNOME Calendar 50 interface architecture.

The old 48-era components were replaced by newer components including:

gcal-calendar-list
gcal-calendar-navigation-button
gcal-meeting-row
gcal-weather-settings

Older components such as:

gcal-calendar-button
gcal-calendar-combo-row-item
gcal-toolbar-end

were removed as part of the upstream v50 transition.

The shared GUI architecture was also expanded through:

src/gui/common/

with reusable widgets such as:

gcal-calendar-row
gcal-calendar-combo-row
gcal-date-chooser
gcal-date-chooser-day

---

4. Newer Event Editor

The event editor received substantial upstream improvements between GNOME Calendar 48 and 50.

CE therefore brings newer event-editor functionality to Mint Calendar, including the attendee and organizer interface and the newer reusable calendar/date widgets.

This means CE is not simply a rebranded copy of Mint's existing Calendar.

It contains the newer v50 event-management code while remaining compatible with Mint 22.

---

5. Blueprint-Based UI

GNOME Calendar 50 moved substantially further into Blueprint for its GTK interface definitions.

Compared with the Mint 48.1+mint1 source, CE contains the newer Blueprint-based UI architecture rather than the old collection of hand-written GtkBuilder ".ui" files.

The ".ui" files are generated during the build process rather than being maintained as the primary source.

This is an upstream GNOME change carried into CE, but it represents a significant architectural update compared with Mint's 48.1 Calendar.

---

6. Newer Upstream Features — Ported Back to Mint 22

The challenge is that GNOME Calendar 50 expects newer GTK4 and libadwaita versions than Linux Mint 22 provides.

GNOME Calendar 50 expects approximately:

GTK4       >= 4.21.2
libadwaita >= 1.8~alpha

Linux Mint 22 / Ubuntu 24.04 provide approximately:

GTK4       4.14.x
libadwaita 1.5.0

Instead of requiring users to upgrade their desktop libraries, CE ports the v50 functionality down to the Mint 22 stack.

This is one of the project's major pieces of original work.

---

7. Compatibility Work Added by CE

Several v50 APIs do not exist on Mint 22's library versions.

CE replaces or backports them.

"Adw.ButtonRow"

Replaced with:

Adw.ActionRow

with appropriate activation behavior and prefix/suffix images.

This affects five locations, including:

- reminders;
- event-editor delete;
- Add Calendar;
- edit-calendar export;
- edit-calendar removal.

"Adw.ToggleGroup" / "Adw.Toggle"

Replaced with linked:

GtkToggleButton

pairs.

Used for:

- AM/PM selection;
- all-day/time-slot selection.

"Adw.Spinner"

Replaced with:

Gtk.Spinner

for:

- synchronization;
- import-dialog loading.

"Adw.InlineViewSwitcher"

Replaced with:

Adw.ViewSwitcher

using:

policy: narrow

"Adw.ShortcutsDialog"

Rebuilt using:

Gtk.ShortcutsWindow

because the newer libadwaita shortcuts API is unavailable on Mint 22.

This also uncovered an existing missing action handler.

CE added:

gcal_application_show_shortcuts()

so the shortcuts menu actually works.

---

8. Backported GTK Functionality

CE also backports functionality that would otherwise be lost on GTK4 4.14.

The writable-calendar model now watches calendar "read-only" properties through "GtkExpressionWatch".

This preserves automatic refresh behavior while remaining compatible with the older GTK4 stack.

The implementation is CE-specific compatibility code rather than a direct copy of the newer API.

---

9. Mint-Specific Compatibility Is Retained

CE does not throw away Linux Mint's existing Calendar modifications.

The Mint desktop-environment-aware Settings launcher remains:

src/utils/gcal-utils.c

Function:

gcal_utils_launch_gnome_settings

The implementation was compared directly with Mint's "48.1+mint1" source and retained byte-for-byte.

So CE is effectively:

Mint Calendar 48.1
        +
GNOME Calendar 50 feature set
        +
CE compatibility work
        =
Mint Calendar CE 1.0

---

10. Independent Application Identity

Unlike Mint's original Calendar package, CE has a completely independent system-visible identity.

Binary

gnome-calendar

becomes:

mint-calendar-ce

Application ID

org.gnome.Calendar

becomes:

org.mint.calendar.ce

Desktop entry

org.gnome.Calendar.desktop

becomes:

org.mint.calendar.ce.desktop

D-Bus service

org.gnome.Calendar.service

becomes:

org.mint.calendar.ce.service

Search provider

org.gnome.Calendar.search-provider.ini

becomes:

org.mint.calendar.ce.search-provider.ini

GSettings

CE uses its own:

org.mint.calendar.ce

with:

/org/mint/calendar/ce/

This prevents CE from intentionally sharing configuration with the original GNOME Calendar installation.

---

11. CE Can Coexist with Mint's Original Calendar

The independent identity is important because CE is not intended to replace Mint's package at the system level.

The target is:

Linux Mint GNOME Calendar
        +
Mint Calendar CE

on the same system.

The applications have separate:

- binaries;
- desktop entries;
- application IDs;
- D-Bus services;
- GSettings schemas;
- search-provider identities;
- private data directories.

A final package file-list comparison remains part of release validation to confirm that the two packages do not overwrite each other's installed files.

---

12. Independent Versioning

Mint's package followed the upstream/Mint versioning model.

CE starts its own release series:

Mint Calendar CE 1.0

rather than using versions such as:

50.0+mint1

This marks CE as an independently maintained project rather than simply another Mint packaging revision.

---

13. Mint Calendar CE Branding

CE introduces its own application branding.

The application is now named:

Mint Calendar CE

The application icon was also redesigned.

The upstream purple-gradient icon was replaced with a flat Mint-inspired design using:

- dark neutral tones;
- Mint green accenting;
- separate symbolic icon assets that remain theme-recolorable.

This gives CE a distinct identity while retaining the visual relationship with the Linux Mint desktop.

---

14. Packaging Changes

Mint's original Calendar packaging is replaced by an independently maintained Debian package.

CE:

- ships as a native ".deb";
- removes the upstream Flatpak build target;
- publishes releases through its own project infrastructure;
- uses its own Debian metadata;
- uses its own maintainer information;
- points VCS and project URLs to the CE repository;
- uses independent release versioning.

No PPA or alternate Ubuntu base is required.

---

15. New Build System Compatibility

CE introduces a pinned Blueprint Compiler fallback.

Ubuntu 24.04 ships Blueprint Compiler 0.12.0, while the v50 source requires functionality introduced later.

CE therefore adds:

subprojects/blueprint-compiler.wrap

pinned to:

Blueprint Compiler 0.16.0

when the system compiler is too old.

This is a build-time compatibility mechanism and does not create a runtime dependency on Blueprint Compiler.

---

16. New CI Infrastructure

Mint's original GitLab CI setup is replaced by CE's own GitHub Actions workflows:

build-debug-release.yml
codeql.yml
release-deb.yml
weekly-sanitizer.yml

CE therefore gains its own:

- automated builds;
- Debian release builds;
- CodeQL security scanning;
- sanitizer testing;
- release automation.

The workflows use "ubuntu-24.04" to match the intended Linux Mint 22 / Ubuntu 24.04 platform.

---

17. Translations

The 77 existing GNOME Calendar translation languages are retained.

The compatibility port does not intentionally alter translated strings.

The application name:

Mint Calendar CE

remains untranslated.

---

18. What CE Does NOT Claim

Mint Calendar CE is not an official Linux Mint project.

It is an independent/community-maintained fork that:

- starts from Linux Mint's maintained Calendar;
- advances the feature base to GNOME Calendar 50;
- carries forward useful Mint compatibility work;
- adapts the newer code to Mint 22;
- provides independent packaging and branding.

The relationship is therefore:

Linux Mint
   │
   └── gnome-calendar 48.1+mint1
              │
              │  CE uses this as its Mint baseline
              ▼
       GNOME Calendar v50
              │
              │  newer upstream features
              ▼
       Compatibility port
              │
              │  GTK4 4.14 / libadwaita 1.5
              ▼
       Mint Calendar CE 1.0

---

19. Comparison: What's Actually New for Mint Users?

New in CE compared with Mint 48.1+mint1| Source of change
GNOME Calendar 50 feature base| Upstream GNOME
New attendee support| Upstream GNOME
New organizer support| Upstream GNOME
New attendee/event-editor UI| Upstream GNOME
Newer sidebar/toolbar architecture| Upstream GNOME
New shared calendar/date widgets| Upstream GNOME
Blueprint-based UI| Upstream GNOME
Newer v50 calendar architecture| Upstream GNOME
GTK4/libadwaita 4.14/1.5 compatibility| CE
Backported writable-calendar watching| CE
v50 API substitutions for older libraries| CE
Shortcuts compatibility implementation| CE
Missing shortcuts action implementation| CE
Independent application identity| CE
Independent GSettings schema| CE
Independent D-Bus identity| CE
Independent search-provider identity| CE
Independent versioning| CE
Mint Calendar CE branding| CE
Mint-inspired application icon| CE
Independent ".deb" release pipeline| CE
GitHub Actions CI| CE
CodeQL scanning| CE
Blueprint Compiler fallback| CE
Linux Mint compatibility patch| Carried forward
Existing 77 translations| Carried forward

---

20. Development History

An earlier development stage used Ubuntu 26.04 to work through the GNOME Calendar v50 port.

The newer Ubuntu environment successfully demonstrated that the v50 code could build, but the resulting package could not be installed on the actual Linux Mint 22 target because of newer library requirements.

This established the final strategy:

1. Use GNOME Calendar v50 as the feature base.
2. Keep Linux Mint 22 as the target.
3. Port incompatible v50 APIs to GTK4 4.14/libadwaita 1.5.
4. Build and test on Ubuntu 24.04.
5. Ship the result as a native ".deb".

Ubuntu 26.04 is therefore development history, not the supported v1.0 target.

---

21. Known Issues / Release Cleanup

In-app icons

The top-level icon resource currently has a namespace mismatch.

Current prefix:

/org/gnome/calendar/icons/scalable/emblems/

Application resource base:

/org/mint/calendar/ce

The required CE prefix is:

/org/mint/calendar/ce/icons/scalable/emblems/

This affects approximately 15 in-app emblem icons.

Partial internal namespace isolation

Several internal resource files still use "/org/gnome/calendar/..." paths.

They currently remain self-consistent and functional, but they are not fully migrated to the CE namespace.

Debian executable bit

The uploaded source comparison showed "debian/rules" with mode "0664" rather than "0755".

The real repository should be checked to ensure the executable bit is preserved.

Debian watch file

CE does not currently ship a "debian/watch" file because CE is independently released rather than being a direct repackaging of the upstream release feed.

This should remain a deliberate project decision.

---

22. CE 1.0 in One Sentence

Mint Calendar CE 1.0 takes Linux Mint's GNOME Calendar 48.1+mint1, brings it forward to the GNOME Calendar 50 feature set, backports that newer code to Mint 22's older GTK4/libadwaita stack, and packages the result as an independently branded, independently versioned, coexistence-friendly ".deb".

---

Summary

For an existing Linux Mint 22 user, the important change is not simply the name or icon.

Mint Calendar CE provides a newer Calendar feature base than Mint's original 48.1+mint1 package while preserving Mint's compatibility work.

The project adds two distinct layers of change:

New functionality inherited from GNOME Calendar 50

- newer event management;
- attendee and organizer support;
- newer event-editor components;
- newer calendar/sidebar architecture;
- newer shared widgets;
- Blueprint-based UI;
- other upstream v48 → v50 improvements.

New work created specifically for Mint Calendar CE

- compatibility with Mint 22's GTK4/libadwaita versions;
- backported APIs and behavior;
- independent application identity;
- independent GSettings and D-Bus identities;
- independent versioning;
- Mint Calendar CE branding;
- independent ".deb" packaging;
- GitHub Actions CI;
- CodeQL scanning;
- Blueprint Compiler fallback;
- coexistence with Mint's original Calendar.

In short:

«Mint Calendar CE is not a new calendar written from scratch. It is Linux Mint's Calendar brought forward to the GNOME Calendar 50 generation and rebuilt to remain compatible with the Mint 22 platform.»
