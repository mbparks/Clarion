# CLARION

**UV-5R memory bench.** A single-file, no-build, offline web app that reads, edits, and writes a
Baofeng UV-5R over the Kenwood-style two-pin programming cable, using Web Serial. Runs from
`file://`. No server, no localhost, no dev server.

Version: v0.2.0
License: GPL-3.0

## What it does

**Channels.** All 128 memories: receive, transmit, duplex and offset, CTCSS and DCS on both sides,
wide or narrow, high or low power, scan-list membership, busy channel lockout, PTT-ID, signal code,
name. The selected channel is drawn on a small emulation of the radio's own display.

**Radio settings.** Squelch, step, battery saver, VOX, backlight, dual watch, key beep, transmit
timer, voice prompt, DTMF side tone, scan resume, PTT-ID and delay, display mode per band, busy
lockout, keypad lock, the three backlight colors, alarm mode, squelch and repeater tail, power on
message, roger beep.

**DTMF.** ANI code, send and gap timing, and the fifteen signal codes that channels select through
S-CODE.

**FM presets.** Twenty five broadcast presets, 65.0 to 108.0 MHz.

**Undo and redo.** Every mutation, including CSV import and hex edits, is a labeled step on a sixty
deep stack. Ctrl+Z and Ctrl+Shift+Z, or the buttons in the header.

**Changed-blocks writing.** The image read from the radio becomes the reference. Any 16 byte block
that differs from it is dirty, and the default write scope sends only those blocks. Less time on the
wire, less exposure.

**Map and diff.** Every region has an editable image offset and a verified flag. Unverified regions
cannot be written without an explicit acknowledgment in the write dialog. The diff table compares
the working image against the last read or against any saved image file, block by block, and names
the region each changed block belongs to. There is a hex editor for everything else.

**Self test.** Twenty six browser-runnable checks over the BCD, tone, name, DTMF, FM, settings,
undo, dirty tracking, and map integrity code. No radio required.

## Finding the offsets I could not confirm

Channels and names are confirmed by shape: the frequencies decode as valid BCD in band and the names
decode as printable text. Settings, DTMF, and FM presets are not, so they ship marked unverified.
Resolving one takes about two minutes each:

1. Read the radio. Save the image as `before.img`.
2. Change one thing on the radio itself. One menu item, or one FM preset.
3. Read the radio again.
4. Map and diff, then **Compare with image file**, and pick `before.img`.
5. The block that moved contains the byte you changed. Put that offset into the map, confirm the
   field reads back correctly, tick **verified**, and save the map.

The map exports as JSON. Send it back and it becomes the shipped default in the next release.

## Requirements

- Chrome or Edge on desktop. Web Serial does not exist in Firefox, Safari, or on mobile. Files,
  editing, undo, and the self test work everywhere.
- A working programming cable. Cheap clones with counterfeit PL2303 chips are refused by the current
  Prolific driver.

## Bench procedure

1. Radio on, volume mid, cable seated all the way in.
2. Connect cable, then **Read from radio**.
3. **Save image**. That file is the way back.
4. Edit. Undo is there if you need it.
5. **Write to radio** with the changed-blocks scope. Power cycle the radio afterward.

## Known Limitations

- **Settings, DTMF, and FM preset offsets are unconfirmed.** They are reasoned guesses. Reading them
  is harmless, writing them is not. The app refuses to write an unverified region without an explicit
  acknowledgment, warns when a region falls inside a protected range, and warns when two regions
  overlap. Confirm each one with the diff workflow before trusting it.
- **The settings field list assumes an order.** Even with the base offset confirmed, an individual
  field could sit one byte away from where CLARION expects. Confirm field by field, not region by
  region, for anything that matters.
- **Tone encoding is the highest risk decode in the channel block.** CTCSS as tenths of a Hz, DCS as
  0x2800 plus the octal code with bit 15 marking inversion. Verify one of each against a known good
  image before writing a full set.
- FM presets assume the word is tenths of a MHz above a 65.0 MHz base. Unconfirmed.
- DTMF send and gap times are exposed as raw byte values in units of 10 ms rather than the menu's own
  list, because the mapping is unconfirmed.
- Writes never touch 0x0CF8 to 0x0D08 or 0x0DF8 to 0x0E30. Those hold calibration and band limits.
- Power is High or Low only. Radios with a medium step, such as the UV-82HP, will not round-trip.
- Exported `.img` is the raw dump. CHIRP appends its own metadata to image files, so a CLARION image
  may not open in CHIRP. CSV is the reliable exchange format in both directions.
- The handshake covers UV-5R, UV-82, UV-6, BF-F11, and BF-A58. Other Baofeng variants may answer the
  handshake but have different memory layouts, and are not supported.
- Out-of-band entries are flagged, not blocked. Verifying transmit privileges is the operator's job.
- Undo lives in memory only. Closing the tab loses it. The image file is the durable artifact.

## Files

- `clarion-v0.2.0.html` - the whole application
- `README.md` - this file

Make. Hack. Learn. Share. Repeat.
