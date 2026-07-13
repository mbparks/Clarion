# The bug I shipped four times

*CLARION v0.5.0, and the value of writing your own standards down*

I have been building CLARION, a browser-based memory programmer for Baofeng handhelds. Single file,
no build step, opens from disk, talks to the radio over Web Serial. Five releases in, it reads and
writes four models, has a virtual radio for testing, verifies every block it writes, and rolls back
cleanly when a write fails.

It also had a script injection vulnerability in every one of those releases, and I did not notice
until I went back to my own development standards and re-read the line that says *no innerHTML with
user or imported data*.

## The bug

CLARION builds its channel table from the radio's memory image. Names come out of that image. In the
UV-5R they are seven characters from a restricted charset. In the 5RM they are twelve characters from
a charset that includes, among other things, angle brackets.

The table was built the way most of us build tables when we are moving fast:

    tr.innerHTML = "<td>" + channel.name + "</td>" + ...

An image file is not a trusted input. Neither is a CSV someone shared on a repeater group's forum,
nor a memory map JSON emailed to you by a helpful stranger. Any of them can carry a channel name. That
name went straight into the DOM as markup.

Nothing bad would have happened to me. I was the only user, and my radios have sensible names in
them. That is exactly the reasoning that puts these bugs in real software.

## The fix

Every node in v0.5.0 is now built through one helper. It sets text as text. It throws if you hand it
a property called `html`. There is no `innerHTML` anywhere in the file, and the self test proves it:

    t("no render function builds markup from strings", () =>
      fns.every(f => !/\.innerHTML|insertAdjacentHTML|outerHTML/.test(f.toString())));

That check reads the source of the render functions at runtime and fails if any of them ever reaches
for markup again. It is the cheapest possible enforcement of a rule, and it will outlive my memory of
having made the mistake.

Imported map files get the same treatment from the other direction. They used to be merged wholesale
into the memory map, labels and all. Now they are validated against a schema that accepts exactly two
things per region: an integer offset in range, and a boolean. Labels, spans, and provenance always
come from CLARION's own table. A file cannot supply structure, only data.

## What this is really about

The interesting thing is not the bug. Everyone has written that line of code. The interesting thing is
that I had already decided, in writing, that I would not do this, and then did it four times anyway,
because at no point did I stop and check my work against my own rules.

Standards you wrote and never re-read are decoration. Standards you check each release are engineering.
The difference between the two is a checklist and ten minutes.

v0.5.0 also brings bulk channel operations, a semantic diff that says "channel 7 power: high to low"
instead of showing you hex, cross-model channel import, keyboard-operable tables, and a service worker.
Those are the features I set out to build. The security pass is the one that mattered.

131 self test checks, all passing. Still, notably, without ever having touched a real radio. That is
the next honest step, and it is the only one that can tell me whether any of this is right.

*Make. Hack. Learn. Share. Repeat.*
