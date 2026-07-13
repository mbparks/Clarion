# CLARION

**Radio memory bench.** A single-file, no-build, offline web app that reads, edits, and writes
handheld radio memories over the programming cable, using Web Serial. Runs from `file://`.
No server, no build step, no telemetry, no analytics.

Version: v0.5.0
Licence: GPL-3.0

## Supported radios

| Model | Protocol | Image | Channels | Names |
| --- | --- | --- | --- | --- |
| Baofeng UV-5R and clones | Baofeng clone mode, 9600 baud | 6472 bytes | 128 | 7 characters |
| Baofeng UV-S9 Plus | same | 6472 bytes | 128 | 7 characters |
| Baofeng x Radioddity UV-5RX3 | same | 6472 bytes | 128 | 7 characters |
| Baofeng 5RM | UV-17Pro family, 115200 baud | 33664 bytes | 1000 | 12 characters |
| Virtual radio | either, in memory | matches the model | matches the model | matches the model |

## New in v0.5.0

**A security fix, and it was a real one.** Every version up to v0.4 built its UI by concatenating
strings into `innerHTML`, and several of those strings came out of the image file: channel names,
DTMF code names, the ASCII column of the hex editor, region labels from an imported map. The 5RM's
name charset even includes angle brackets. That is a script injection path from an untrusted `.img`,
CSV, or map file straight into the page. v0.5.0 has no `innerHTML` anywhere. Every node is built
through a DOM helper that sets text as text, and the helper throws if asked to accept markup.
Imported map JSON is validated against a schema that takes only an offset and a boolean; labels,
spans, and sources always come from CLARION's own table and can never be supplied by a file.
Seven self test checks now enforce all of this, including a check that scans the render functions
themselves and fails if any of them ever reaches for `innerHTML` again.

**Bulk channel operations.** Select with the checkbox, the X key, or select-all. Then set power,
mode, scan membership, busy lockout, tone, or signal code across the selection. Move a selection up
or down. Insert a blank and shift everything below. Delete and close the gap. Clear in place.

**Find.** Filter by name, frequency, or channel number.

**Semantic diff.** The Map and diff tab now leads with what changed rather than which bytes moved:
"Channel 007 power: high to low", "Squelch: 3 to 5", "FM preset 4: empty to 101.1". The byte view is
still there, one dropdown away, because that is the tool for hunting an unknown offset.

**Cross-model channel import.** Open a UV-5R image while working on a 5RM and pull its channels in.
Names too long for the destination are shortened, DCS codes the destination lacks are dropped, out of
band frequencies are flagged, and every one of those is reported rather than silently swallowed.

**Keyboard and touch.** Table rows are focusable and operable: arrows move, Enter selects, X marks.
Focus outlines are restyled, never stripped. Coarse pointers get 44 px targets.

**PWA layer.** Manifest and icon are embedded as data URIs, so the app installs when served. The
service worker ships as an optional companion file with a version-stamped cache and an in-app
update-and-reload prompt. It is additive: with `clarion-sw.js` absent, or opened from disk, nothing
changes. A service worker cannot be registered from a `file://` URL, so this is the only honest way
to satisfy both the PWA standard and the no-server rule.

**Backups are archived, not deleted.** The way back from a bad write is never destroyed.

**One row at a time.** Editing a channel used to re-render the whole table, which on the 5RM meant a
thousand rows per keystroke. Now it replaces one row.

## Self test

131 checks, in the browser, no radio required. Security, codecs, obfuscation, channel round trips,
bulk operations, lint, semantic diff, cross-model import, map validation, address plans, region
overlap, writability, dirty tracking, and full protocol round trips against the virtual radio for
every model. All 131 pass.

## Embedded assets

One SVG icon and one manifest, both original work, both under the app's licence, both inline as data
URIs. No fonts, images, or audio are shipped. Total asset budget: under 4 KB.

## Bench procedure

1. Pick the model. This sets the protocol, the baud rate, the image size, and the band table.
2. Radio on, volume mid, cable seated all the way in.
3. Connect, then **Read from radio**. A backup is stored automatically.
4. Edit. Run **Preflight check**. Review the semantic diff.
5. **Write to radio** with the changed-blocks scope, verification on. Power cycle the radio.

## Known Limitations

- **Nothing here has met a physical radio.** The virtual radio proves CLARION is self consistent. It
  cannot prove CLARION agrees with Baofeng, because it was built from the same understanding of the
  protocol that the app uses. A shared misunderstanding passes every test. The first real read is
  still the moment of truth.
- **UV-5R settings, DTMF, and FM offsets remain guesses** and ship marked unverified. The 5RM
  equivalents come from the CHIRP driver and are marked as such, which is better but still second hand.
- Verification reads back only the blocks written. A radio that mangles bytes outside them would not
  be caught.
- Rollback needs a reference image, so a read earlier in the session. Without one, restore a backup
  and write the full image.
- Backups live in this browser's storage for this file. Clearing site data removes them.
- Bulk move shifts by one slot per press. There is no drag reorder.
- The 5RM's thousand channels are paged 128 at a time. Find and used-only searches across all of them.
- Power is High or Low. Models with a medium step will not round-trip.
- Out of band entries are flagged, not blocked. Verifying transmit privileges is the operator's job.

## Next, toward v1.0

Verify the UV-5R settings, DTMF, and FM offsets against real hardware and ship them as defaults.
Then the remaining v1.0 candidates: drag reorder, per-channel notes that live in the CSV rather than
the radio, and a printable channel card.

## Credits

The Baofeng 5RM memory layout and transfer protocol are derived from the CHIRP project's
`baofeng_uv17Pro` driver by Sander van der Wel, GPL. The UV-5R clone protocol follows the long
standing CHIRP `uv5r` driver. No third party code is vendored into this file.

## Files

- `clarion-v0.5.0.html` - the whole application
- `clarion-sw.js` - optional service worker, only used when served over HTTP
- `README.md` - this file

Make. Hack. Learn. Share. Repeat.
