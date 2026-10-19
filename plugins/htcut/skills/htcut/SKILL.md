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
Times accept `"2.5s"`, `"75f"`, `"00:02.5"`. Prefer the purpose-built commands over hand-written ops:
`ripple cut|insert|trim` (whole-timeline, speech-aware), `transcript [clean|cut]` (text-based editing, fillers,
silences, retakes), `stumbles`, `captions`, `transition add`, `speed`, `audio normalize|denoise|enhance|duck`,
`mask set`, `device`, `grade`, `chapters`, `description`. Use `--dry-run` where offered.

## Graphics

Overlays are HTML templates in the project's `graphics/` folder with typed fields declared on `<html
data-composition-variables='[…]'>`; set them per clip with `setClip … {"title":{"vars":{…}}}`. Animated templates put
`data-animated` on `<html>` and expose `window.htcut.seek(t)` (GSAP is bundled offline in `graphics/vendor/`).
Iterate with `htcut graphic <template> --at 1,2.5 --vars '{…}'` (stills + a cut-off-text check) before placing.

## Verify, then ship

```bash
htcut check                      # overlaps, gaps, missing media, clipped words (reads audio)
htcut frame 4.2s -o /tmp/f.png   # one composited frame — look at it
htcut sheet -o /tmp/sheet.png    # contact sheet of the whole timeline — look at it
htcut preflight                  # loudness, safe-area text, cut-off text, silences, chapters
htcut jobs --wait                # let background renders finish
htcut export -o out.mp4          # prints a loudness / true-peak check
```

Look at frames before calling anything done (start, middle, end). Run `htcut check` before handing the user a cut
to watch. Renders and recipes fail loudly instead of guessing — read the error, fix the cause.
