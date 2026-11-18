<picture><source media="(prefers-color-scheme: dark)" srcset="assets/icon-github-dark.png"><img src="assets/icon-github.png" width="128" alt="mp3-to-m4b icon: a play sign inside a ring cut where each chapter ends"></picture>

# mp3-to-m4b: a skill for AI assistants that makes an .m4b audiobook from mp3 files

A free, open source agent skill. Point the assistant at a folder of audiobook parts and say «make an audiobook» («join these into one m4b», «for Apple Books», «for BookPlayer on my iPhone») and it joins them with [ffmpeg](https://ffmpeg.org) on the computer into one `.m4b` file, then checks the file before saying it is done.

- The parts in the book's order: numbers in names sorted as numbers (2 before 10), compared with the track numbers in the tags.
- One chapter per part, named from the tags or the file names with the leftovers stripped («01 - The Happy Prince» becomes «The Happy Prince»), or from a list of chapter names; several parts can make one chapter, and one long file can hold several.
- The cover from an image in the folder or from the art inside the files; title, author, narrator and year as tags; marked as an audiobook, so Apple Books and Apple Music file it under audiobooks.
- One encode, sample-exact: each chapter starts on the exact sample where its part starts. The sound keeps its sample rate; mono stays mono (and «stereo» with the same sound in both channels becomes mono); the bitrate never goes above the source. Apple's AAC encoder on macOS.
- Finds cut-off files (a part that plays shorter than its header says, as after an unfinished download) and the ffmpeg 9 bug that silently drops the chapter track Apple's players read.

Input: mp3, m4a, m4b, aac, ogg, opus, flac, wav, wma, aiff. Output: `.m4b` (AAC in MP4 with QuickTime and Nero chapters), which plays in Apple Books, BookPlayer, Smart AudioBook Player, VLC, Audiobookshelf and most other players.

Works in assistants that run commands on the person's own computer: terminal agents such as Claude Code, Codex, Gemini CLI and Antigravity, OpenCode, OpenClaw, Hermes and others. A chat app can use it only if its sandbox has ffmpeg and the files fit its upload limit, which rules out most whole books. One file, `SKILL.md`, carries the whole method, with a route that needs only ffmpeg; the helper script `scripts/m4b.py` (Python 3, standard library only) makes the common case one command.

## Install

Each way is marked by how it was checked: **run** means installed and used for real on the date in «Tested on»; **docs** means taken from the vendor's documentation on 2026-11-15 and not yet run.

### The tools it needs

ffmpeg, and Python 3 for the helper script. The assistant checks for them and, when one is missing, names the command and waits for a yes:

- macOS: `brew install ffmpeg`; Python 3 comes with `xcode-select --install`.
- Windows: `winget install Gyan.FFmpeg` and `winget install Python.Python.3.13`.
- Linux: `sudo apt install ffmpeg python3` or the system's own package manager.

### Terminal agents

One command for every agent found on the computer (needs Node.js; `-g` installs for the user, without it for the current project only) — **run** (2026-11-17, from this repository on GitHub into an empty home folder; the files installed identical to the repository):

```sh
npx skills add geragarazagh/mp3-to-m4b -g
```

It puts the skill into `~/.agents/skills/mp3-to-m4b`, which Codex, Cursor, Cline, OpenCode and most other agents read, and links it into `~/.claude/skills` for Claude Code.

Claude Code, as a plugin — **run** (2026-11-17, from this repository on GitHub: the marketplace added, the plugin installed, its `SKILL.md` identical to the repository):

```sh
claude plugin marketplace add geragarazagh/mp3-to-m4b
claude plugin install mp3-to-m4b@mp3-to-m4b
```

Gemini CLI, as an extension — **run** (2026-11-18, from this repository's latest release on GitHub, Gemini CLI 0.68.0, into an empty home folder: the extension installed and its skill listed by `gemini skills list`):

```sh
gemini extensions install https://github.com/geragarazagh/mp3-to-m4b
```

By hand, any agent that reads the Agent Skills folder (Codex, OpenCode, OpenClaw, Hermes, Grok Build) — **docs**: copy `skills/mp3-to-m4b` to `~/.agents/skills/mp3-to-m4b`. Claude Code reads `~/.claude/skills/mp3-to-m4b`; Antigravity reads `~/.gemini/config/skills/mp3-to-m4b`.

### No skills in the agent

Paste the text of [`SKILL.md`](skills/mp3-to-m4b/SKILL.md) into the agent's instructions (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md` or the agent's settings), or as the first message of a session. Without the script the assistant uses the ffmpeg-only route in the same text.

## Use

Name the folder and say what you want, in any language. To call the skill by name: `/mp3-to-m4b` in Claude Code (`/mp3-to-m4b:mp3-to-m4b` when installed as the plugin) and Antigravity; `$mp3-to-m4b` in Codex.

```
Make an audiobook for Apple Books from the mp3 files in ~/Downloads/Moby Dick.
Join these parts into one m4b with chapters and the cover.
The chapters are: Loomings, The Carpet-Bag, The Spouter-Inn, … (one per file).
Files 1 to 3 are chapter one, 4 to 6 chapter two.
```

The assistant answers in the language of the request and starts at once when the folder is clear. The `.m4b` goes next to the folder, named after the book, unless the request names another place. It asks only when it is unclear which files make the book.

On an iPhone: BookPlayer opens the file from the Files app or AirDrop. Apple Books on the Mac imports it by double-click or drag; the Mac's Books app treats a file as an audiobook by its `.m4b` extension.

## What this skill does on the computer

- Runs ffmpeg and ffprobe on the files in the folder, and the helper script `scripts/m4b.py` when Python is there (read it first: about 420 lines, standard library only).
- Reads only the folder it was given. Writes only the new `.m4b`, plus a temporary folder next to it that is deleted when the job ends. Never changes, renames or deletes the person's files; never overwrites a file without `--overwrite`.
- Connects to nothing. Sends nothing anywhere: no telemetry, no accounts, no keys.
- Installs and updates nothing by itself: names the command and waits for a yes.
- Stores nothing about the person.
- The installers are other people's programs with their own rules: `npx skills` (Vercel) sends an anonymous install count unless `DISABLE_TELEMETRY=1` is set.

## Tested on

Until 0.2.0 (2026-11-18) the skill was named `audiobook`, in the repository geragarazagh/audiobook, which GitHub now redirects here; the runs below used that name.

2026-11-16, in a fresh Claude Code configuration (its own login, no user instructions, only this skill and two others of the same author (YouTube downloads, text to speech), so that a near miss could start the wrong one): Claude Haiku 5.5 (`claude-haiku-5-5`) in Claude Code 2.1.325 on macOS 27.2 (Apple M3 Pro), ffmpeg 9.0.2 (Homebrew). Material: LibriVox's public-domain recording of Oscar Wilde's «The Happy Prince and Other Tales» (version 2, five mp3 parts, 1 h 45 min, 64 kbps mono). Checked by script with ffprobe and Apple's AVFoundation (what Apple Books and BookPlayer use to read chapters and cover):

- «Make an audiobook for Apple Books from the mp3 files in this folder», the five parts as published: 5 chapters named after the stories, the cover, title and author.
- «Turn these mp3s into one m4b audiobook with chapters and the cover, for BookPlayer on my iPhone», the book cut into twelve parts named «Part 1» to «Part 12» with the cover only inside the mp3s: 12 chapters in order, named «Chapter 1» to «Chapter 12», the cover.
- The skill started by itself for both, and for none of three near misses (an mp3 to m4a, a transcription, the sound out of an mp4).

2026-11-17, by hand with the helper script on the same Mac:

- A long book: the same recording joined six times and cut into 118 parts of about 5 min 20 s, named «Book 1» to «Book 118» (10 h 29 min): built in 1 min 27 s, 305 MB, 118 chapters in order, all seen by AVFoundation, the cover.
- Music-like stereo: the soundtrack of the Sintel trailer (Blender Foundation, CC BY 3.0) as three stereo mp3 parts at 192 kbps, 48 kHz: kept stereo, 128 kbps, 48 kHz, 3 chapters.
- Linux: Ubuntu 24.04 (arm64, a virtual machine) with its own ffmpeg 6.1.1 and Python 3.12: the five LibriVox parts built with ffmpeg's built-in AAC encoder in 55 s; the check passed, and AVFoundation on the Mac read all 5 chapters and the cover.

2026-11-15, the first draft: the same model, as a sub-agent inside a configured Claude Code session (not a fresh configuration), on the same LibriVox book as published, with its tags removed and files renamed, and cut into twelve parts with the cover only inside the mp3s. Checked by script: ffprobe, AVFoundation, `afinfo`.

- 4 of 4 books right: five parts as published (5 chapters named after the stories, cover, title and author from the tags); the same without tags and with names «Track 1» to «Track 5», asked in Russian with the story names in the request (answered in Russian, chapter names from the request); twelve parts with the cover only inside the mp3s (12 chapters in order, cover); the same with `SKILL.md` alone and no Python (the ffmpeg-only route: 12 chapters, cover, chapter track present). Every file: brand M4B, media kind «Audiobook», AAC 64 kbps mono 22 kHz like the source, all chapters seen by AVFoundation.
- The same requests without the skill, twice: both made a working `.m4b` with chapters AVFoundation reads, but marked as music (brand M4A, no audiobook media kind) and encoded with ffmpeg's built-in AAC, about 7 % larger.
- By hand on the same book: the helper script's chapter marks and the ffmpeg-only route's agree within 0.02 s over 1 h 45 min; a part cut off by an unfinished download was reported by the script's warning; the ffmpeg 9 bug was reproduced (cover copied from an mp3: AVFoundation 0 of 5 chapters) and the fix confirmed (5 of 5).

Version 0.2.0 renames the skill from `audiobook` to `mp3-to-m4b` and sets the catalogue icons; what the skill does is unchanged. On 2026-11-18 the installs were run again under the new name from this repository's v0.2.0 release, each into an empty home folder: `npx skills add geragarazagh/mp3-to-m4b -g` (files identical to `skills/mp3-to-m4b`), the Claude Code plugin commands above in Claude Code 2.1.326 (the plugin's skill folder identical), and `gemini extensions install` in Gemini CLI 0.68.0 (installed from the release, version 0.2.0, the skill folder identical).

Not tested: other models, other agents, Windows, chat apps, a model on Linux (the script was run by hand there), the installs marked **docs**, opening the file in Apple Books or BookPlayer themselves (AVFoundation stands in for them).

## What it cannot do

- Read a book aloud: it joins recordings that exist. Text to speech is another job.
- Split a book's single file into chapters by listening to it: chapter times inside one long file come from the person or from the file names.
- Remove DRM (Audible's AAX and the like).
- Fix the sound: no loudness, noise or speed changes.

## License

MIT
