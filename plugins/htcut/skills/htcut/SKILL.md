---
name: htcut
description: Make or edit videos with HTCUT, a Mac video editor you drive with the `htcut` CLI while the user edits the same project in the HTCUT app — cuts, Shorts, text and motion graphics, captions, music and sound effects, cleanup of fillers/silences/retakes, transitions, colour, cut-outs, export and YouTube packaging. Use it for any "make a video / cut this / add captions / add music / export" request when HTCUT is installed (`htcut` on PATH).
---

# HTCUT

First check it's installed: `htcut --version`. If the command isn't found, tell the user to download HTCUT (free, Mac,
Apple Silicon) from https://htcut.hanley.world, open it once and choose **HTCUT ▸ Install Command Line Tool…** — then
stop; don't try to build or fetch it yourself.

**Add-ons are the user's call.** Transcripts, captions, stumble checks and voice match need speech recognition
(~2.5 GB); Video and Points cut-outs need their own models. When a command says something isn't installed, or before
you start work that needs it, run `htcut doctor` and tell the user what's missing, what it's for and how big it is,
then **ask before installing** — `htcut models install <name>` or HTCUT ▸ Settings ▸ Add-ons. Never install one
unprompted. `htcut doctor` is also the first thing to run when something about the setup seems wrong.

A project is a folder (`project.json`, `timeline.json`, `ops.jsonl`, `graphics/`, `media/`). The user may have it open
in the HTCUT app and be editing at the same time, so **every change is an op** — never edit `project.json`,
`timeline.json` or `ops.jsonl` yourself (the project's hook blocks it). `htcut help` lists every command; it is the
reference. Inside a project, its `CLAUDE.md` has the op shapes.

## Start

```bash
htcut init "<dir>" --name "<Title>" --width 1920 --height 1080 --fps 30   # 1080x1920 for Shorts
cd "<dir>"                     # init also writes the project CLAUDE.md + Claude hooks
htcut add-media <files…>       # register video / audio / images
htcut read                     # tracks and clips; 8-char ids work in ops
htcut ui                       # what the user is looking at: playhead, clips under it, selection
```

End a prompt with `;;` inside a project and the app's playhead/selection is attached automatically — that's how the
user says "this clip" or "here".

## Edit

```bash
htcut status                                             # current rev
htcut op apply ops.json --base-rev <rev> --label "what"  # a JSON array of ops, applied atomically, undoable
```

A conflict means the user changed the same clips since your rev: re-read and redo, never force over their edits.
Times need a unit — `"2.5s"`, `"75f"`, `"00:02.5"`; a bare number (`1.2`) is an error. Prefer the purpose-built
commands over hand-written ops:
`ripple cut|insert|trim` (whole-timeline, speech-aware), `transcript [clean|cut]` (text-based editing, fillers,
silences, retakes), `stumbles`, `captions`, `transition add`, `speed`, `audio normalize|denoise|enhance|duck`,
`mask set`, `device`, `grade`, `chapters`, `description`. Use `--dry-run` where offered.
`speed --clip ID --fit` slows a rendered clip that's shorter than its slot so it fills it with transition handles.
`speed --clip ID 1.5 | --ramp hero | --ramp "1,1,0.3,1,2" | --off` keeps the clip's slot (it plays more or less
source; errors if the media is too short), so beat-synced rows stay in sync; `--ripple` changes its length instead.
Volume automation without splitting clips: `audio dip --clip ID --from A --to B` (a silent hole, back at full level
exactly at B — the beat before a drop), `audio swell --clip ID --to T --len 2s --db -24`, `audio key --clip ID --at T
--db -12` (`--clear`; no flags lists them). Times are timeline seconds; `read` shows `vol[…]`.
`compound beats [--elements]` makes one compound per "NN ·"-named beat: transitions then move the whole stack (titles
with the picture), or with `--elements` each beat's picture enters and leaves on its own. `transition add|remove --clip`
reaches clips inside compounds too.
Cutting to music: `beats --clip MUSIC [--grid | --json]` (BPM, beat/bar grid, drops and the dips before them, in source
AND timeline seconds, frame-exact) — never find beats by hand; `beats --clip MUSIC --align-drop 8s --apply` slips the bed
so its strongest drop lands on that cut. Punch-ins / pushes / whips / shakes on a clip: `move --clip ID punch|push|pull|
whip-in|shake|drift [--amount 1.8] [--at T] [--focus auto|face|x,y] [--dry-run]` frames the face/subject Vision finds
(`focus --clip ID -o f.png` shows what it found) — never guess transform x/y for a zoom.
Landscape footage on a vertical Short: `reframe --report` (dry run + contact sheet: now | planned, safe area and numbered
subjects drawn on) then `reframe --apply` keeps each shot's people inside the Shorts safe area — follows #1 or frames the
group, and when the group is too wide fits it at full width over a matched canvas built from the shot's own edges
(`fill: "blur.match"`; `--canvas dark|blur` = old looks; `--fallback blur|pan|split|flag`). Read
the sheet; fix a wrong pick with `--clips ID --subjects 2,3 --force` / `--primary N` / `--group all` / `--focus x,y`.
Shots with moves: add `--keep-motion --punch-aware`. `check` warns when an analysed clip's subject leaves the safe area.
When something detection can't know matters (a fingertip on glass, the girl behind), SAVE it on the clip instead of a
one-off `--focus`: `focus add --clip ID --at T x,y [--box w,h] --label spark [--must|--nice] [--track]` (source 0…1 —
read it off `focus --clip ID -o f.png`; `--canvas X,Y` = a point on the frame). Reframe then keeps the must points (+ nice
ones that fit), else blur-fits or pans between them (≥ 1 s each) and says why; `move` aims at the first must point;
`check` warns when one leaves the safe area. `focus list`, `focus remove --clip ID --label L`.
Trailer looks: transitions `glitch`, `rgb.split`, `flash` (exact white/`--color` frames, 2 by default, `--dur 1f`–`3f`),
`zoom.blur`, `stutter` (footage only) — `transition list` has the catalog. Clip effects: `fx add <kind> --clip ID
[--from T --to T] [--amount 0–1] [--rate N]` (grain, rgb.split, zoom.blur, shake, light.sweep, vignette, `freeze
--at T --dur D [--photo]`, echo); without `--clip`, `--from/--to` puts the look on the whole frame (an adjustment clip).
Check each with `htcut frame` at its peak.
Branded transitions: `transition add html:<template> --at T [--vars JSON]` — a graphics/ page with
`<html data-transition="matte+overlay">`: `[data-layer="matte"]` alpha reveals the next shot, `[data-layer="overlay"]`
draws on top, timed by `htcut.seek(t)` / `htcut.duration`. Starters: blueprint-wipe, shape-reveal, word-slam, ink-bars
(`transition list`). Check a look with `transition preview html:<t> --at 0.25,0.5,0.75`, then `htcut frame` on the cut.
Before asking the human to use a panel, prove it's set up; never say "ready, go record" from the docs. `htcut vo ready`
(exit 1 + the fix if the recorder's Line by line wouldn't list the lines: stand-ins go on a row like "Script", clips named
`VO · <line>`, never on the "Voiceover" row takes land on). If you can't check a panel, say so.

Voiceover takes: `vo check` (each take vs its script line, incl. 1–2-word false starts), `vo spans FILES… --json` (speech
span + suggested false-start cuts per take file, for build scripts), `transcribe FILE…` (word-timed transcript of any
file through HTCUT's Whisper — never run mlx_whisper yourself).

## Graphics

Overlays are HTML templates in the project's `graphics/` folder with typed fields declared on `<html
data-composition-variables='[…]'>`; set them per clip with `setClip … {"title":{"vars":{…}}}`. Animated templates put
`data-animated` on `<html>` and expose `window.htcut.seek(t)` (GSAP is bundled offline in `graphics/vendor/`).
Iterate with `htcut graphic <template> --at 1,2.5 --vars '{…}'` (stills + a cut-off-text check) before placing.
Make soft glows and blurred shapes with gradients (`radial-gradient(closest-side, …, transparent)`), never a large CSS
`filter: blur()` — those make 4K renders fail. Fit any giant word or number to ≤ 90% of the frame width in the page's
own script (measure it and scale the font down): text running off the frame edge isn't caught by the cut-off check.
Built-in explainer graphics: `stamp`, `glass-pill`, `price-math`, `screen-compare`, `cost-over-time`, `storm-sky` +
`storm-fx` (`htcut templates`). `stamp` slams in by default; for calm titles on an opener use `motion: smooth` +
`landed: true` (up on frame 0, one gentle pulse), `tintAmount`/`glassBlur` for clearer glass, `padX`/`padY` for roomier
pills; `ranking-list` = a numbered liquid-glass list with swatches and fine-print notes. Liquid glass (`data-htcut-glass` on an element; `headline` has `panel: liquid`) refracts
the picture beneath, so use it over footage or renders — over a flat background it reads as a ghost box, and on an
opaque page it's hidden; `htcut check` warns on both.
A ring/label marking a spot on a moving picture (push-in, pan): place it over the still picture, then `htcut follow
--clip RING --to PICTURE` — it moves with the picture's keyframes; never copy the picture's keyframes onto it.

## Verify, then ship

```bash
htcut check                      # overlaps, gaps, missing media, clipped words (reads audio)
htcut frame 4.2s -o /tmp/f.png   # one composited frame — look at it
htcut sheet -o /tmp/sheet.png    # contact sheet of the whole timeline — look at it
htcut preflight                  # loudness, safe-area text, cut-off text, silences, chapters
htcut polish [--video review.mp4] # clean-up sweep: cut-out over nothing, broken cut-out frames, looking away at clip edges,
                                 # hidden clips, framing; --video = flash frames in a render (export / review make run it too)
htcut polish --framing           # framing only: strips with black bars, upscaled sources, subject cropped / too wide, face cut,
                                 # tiny subject, text on a face — run it after placing footage of a different aspect than the
                                 # canvas (16:9 on a Short); each hit prints an image path: look at it
htcut polish allow --clip ID     # deliberate fast motion: its flash-frame hits print as "allowed" from now on
htcut polish still 1s            # flag renders still ≥ 1 s (auto on YouTube Shorts; else 3 s); one-off: polish --video F --still 1s
htcut review add out.mp4 --title "v2" --group ab   # a preview on the review page (version picker, A/B, frame stepping)
htcut jobs --wait                # let background renders finish
htcut export -o out.mp4          # prints a loudness / true-peak check
```

For YouTube: `htcut master --limit -1.5` (AAC encoding pushes a −1 dBTP limit back over it) and set the master gain
from a review copy's loudness first (`htcut master --gain auto --file review.mp4`; no --file = the last export), so the
full-size export only runs once. Credits: `htcut description --credits CREDITS.tsv` adds a "Sources:" block. Times in `--from/--to/--at` take units
(`206s`).

Look at frames before calling anything done (start, middle, end). Run `htcut check` before handing the user a cut
to watch. Renders and recipes fail loudly instead of guessing — read the error, fix the cause.

## When the video is finished

Suggest **File ▸ Project Done…** (or `htcut retro`): it reads what HTCUT recorded and this project's Claude Code
sessions and lists fixes for HTCUT, this skill and the user's own setup. It runs on the user's Claude (about a minute,
some usage) and sends nothing unless the user reviews the suggestions and presses Send — let them decide.
