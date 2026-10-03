# supii-data

The data half of [supii](https://github.com/logbucket/supii) — every dictionary,
analyzer model and media file the app reads, laid out per language.

**This repo is not the app.** The app is a separate repository and reads from
whatever directory this is cloned into. Nothing here is code that runs in the
app; `scripts/` holds the build tools that produced these files.

---

## Where to clone it

The app resolves one directory, and it must be this repo's root:

| OS | Path |
|---|---|
| Linux | `~/.local/share/supii` (`$XDG_DATA_HOME/supii`) |
| macOS | `~/Library/Application Support/supii` |
| Windows | `%LOCALAPPDATA%\supii` |

```bash
git clone git@github.com:logbucket/supii-data.git \
  ~/.local/share/supii          # Linux
```

The folder name `supii` is part of the contract — it is not the repository
name. The app appends it to the OS data dir rather than using Tauri's
identifier-based directory, which would be `com.gray.supii`.

**700 MB.** Cloning with `--depth 1` is much faster if you do not need history.

---

## Layout

```
supii/
├── user.db                 ← YOUR data. Never overwritten, never in git.
├── assets/                 ← YOUR images (custom loading animation). Never in git.
│
├── ja/                     ← Japanese
│   ├── jmdict.sqlite        96 MB   lookup dictionary (words, kanji, glosses)
│   ├── sudachi/            208 MB   analyzer: system.dic + sudachi.json
│   ├── anim_ja/             54 MB   stroke-order SVGs + licences
│   ├── grammar_ja/         6.4 MB   hanabira grammar notes (807 files)
│   ├── audio_ja/           1.0 MB   102 kana recordings
│   └── presets_ja.db       704 KB   JLPT N5–N1 decks + radicals
│
├── ko/                     ← Korean
│   ├── krdict.db           201 MB   lookup dictionary
│   ├── anim_ko/            3.6 MB   hangul stroke SVGs + licences
│   ├── grammar_ko/         5.5 MB   hanabira grammar notes (700 files)
│   ├── audio_ko/           320 KB   40 jamo recordings
│   └── presets_ko.db       1.6 MB   TOPIK 1–2 vocab + grammar
│
├── zh/                     ← Chinese
│   ├── cedict.txt          9.4 MB   lookup dictionary
│   ├── anim_zh/             65 MB   simplified + traditional stroke SVGs
│   ├── audio_zh/             34 MB   1,632 pinyin recordings
│   └── presets_zh.db       808 KB   HSK 1–6 chars, vocab, grammar
│
└── scripts/                ← build tools for the above. See its README.
```

Each language folder owns **all** of its content. A Japanese-only install
carries nothing for Korean or Chinese, and the app never looks outside the
folder for the language it is showing.

### Why two dictionaries per language

The analyzer and the lookup dictionary are different jobs, and neither one does
both:

| | Tokenizes | Looks up glosses |
|---|---|---|
| `system.dic` (Sudachi) | yes | **no** — no English glosses in any variant |
| `jmdict.sqlite` | no | yes |

Korean and Chinese work the same way: `garu-core` and `jieba-rs` segment, and
`krdict.db` / `cedict.txt` answer lookups. So both files are always needed.

---

## What is *not* here

**Your own data stays in the same folder but is never committed.** `user.db`
holds your cards, collections, ratings, grammar progress and SRS state;
`assets/` holds images you added. Both are in `.gitignore`. If you pull and the
app suddenly has no cards, you overwrote `user.db` — restore it from a backup.

**Plugin binaries are not here.** `plugins/*.so` are build output of the code
repo, not data.

---

## Rebuilding a file

`scripts/` has one folder per tool, each with its own README and a link back to
this file's credits table. The short version:

| Rebuild | Tool |
|---|---|
| `ja/jmdict.sqlite` | `jmdict-kanjidict/` — run upstream's converter |
| `ko/krdict.db` | `krdict-to-sqlite/` — **our edited copy, not a fresh clone** |
| `zh/cedict.txt` | plain download, no build |
| `*/anim_*/` | `animcjk` upstream, five directories only |
| `ja/sudachi/` | Sudachi source build |
| `*/presets_*.db` | `scripts/` in the code repo |
| `*/audio_*/` | see the audio notes in the scripts README |

---

## Credits and licences

Every file here is somebody else's work. **One licence is unresolved** — the TOPIK
grade column in `ko/presets_ko.db`, marked ⚠ below. Everything else has confirmed
terms, several of which still require attribution or share-alike.

### Japanese

| What | Source | Licence |
|---|---|---|
| `ja/jmdict.sqlite` | [jmdict-sqlite](https://github.com/shirakaba/jmdict-sqlite) (Jamie Birch), data from [jmdict-yomitan](https://github.com/yomidevs/jmdict-yomitan) | MIT (tool) / JMdict terms (data) |
| `ja/sudachi/` | [Sudachi](https://github.com/WorksApplications/Sudachi) | Apache-2.0 |
| `ja/anim_ja/` | [animCJK](https://github.com/parsimonhi/animCJK) | **Three:** Arphic Public License (character SVGs), LGPL-3.0-or-later (kana SVGs), Unihan (`dictionary*.txt`) |
| `ja/grammar_ja/` | [hanabira.org-japanese-content](https://github.com/tristcoil/hanabira.org-japanese-content) | **CC BY 4.0 — attribution required** |
| `ja/audio_ja/` | generated in-house with Google TTS (`gTTS`, MIT — a local build tool only) | **ours** — no third-party licence |
| `ja/presets_ja.db` | [OpenJLPT](https://github.com/evanclan/OpenJLPT) JLPT data + [kanjium](https://github.com/mutsuhiro/kanjium) radicals | **CC BY-SA 4.0** (share-alike) / **MIT** |

### Korean

| What | Source | Licence |
|---|---|---|
| `ko/krdict.db` | [krdict-to-sqlite](https://github.com/ketzu/krdict-to-sqlite) (MIT), data from the National Institute of the Korean Language | **CC BY-SA 2.0 KR** (attribution + share-alike) |
| `ko/anim_ko/` | [animCJK](https://github.com/parsimonhi/animCJK) | Arphic Public License |
| `ko/grammar_ko/` | hanabira (as above) | **CC BY 4.0 — attribution required** |
| `ko/audio_ko/` | generated in-house with Google TTS (`gTTS`, MIT — a local build tool only) | **ours** — no third-party licence |
| `ko/presets_ko.db` | hanabira (grammar) + NIKL/TOPIK (vocab) | **CC BY 4.0** (grammar) / **CC BY-SA 2.0 KR** (NIKL) / ⚠ **TOPIK grade column unconfirmed** |

### Chinese

| What | Source | Licence |
|---|---|---|
| `zh/cedict.txt` | [CC-CEDICT](https://www.mdbg.net/chinese/dictionary?page=cc-cedict) | **CC BY-SA 4.0 — share-alike** |
| `zh/anim_zh/` | [animCJK](https://github.com/parsimonhi/animCJK) | Arphic Public License |
| `zh/audio_zh/` | [mp3-chinese-pinyin-sound](https://github.com/davinfifield/mp3-chinese-pinyin-sound) | public domain |
| `zh/presets_zh.db` | [HSK 3.0](https://github.com/krmanik/HSK-3.0), built from CC-CEDICT + SUBTLEX-CH + Pleco | **CC BY-SA 4.0** (share-alike) |

### The one unresolved item

Down from three. **The TOPIK grade column** is all that is left open.

NIKL is settled: the institute distributes 한국어기초사전 under **CC BY-SA 2.0 KR**
since 2019-03-11 (attribution + share-alike), which covers `ko/krdict.db` and the
NIKL half of the vocab tables. The Japanese decks moved to **OpenJLPT**
(CC BY-SA 4.0) on 2026-10-03, replacing *coto jlpt*, which had no licence file
anywhere upstream. The Chinese decks are **HSK 3.0** rebuilt from CC-CEDICT and
SUBTLEX-CH (both CC BY-SA 4.0) plus Pleco (MIT); only the New HSK (2021) lists are
used, because the BCT/YCT lists in that repo carry no licence statement.

What is *not* settled is TOPIK (`한국어능력시험`), a separate body, whose level
assignment is mixed into the same `ko/presets_ko.db` vocab tables. Measured
against the shipped file:

| | rows |
|---|---|
| vocab rows in total | 5,741 |
| graded 초급/중급 by TOPIK | 5,741 (100%) |
| graded A/B/C by NIKL | 4,275 |
| **TOPIK-only** (no NIKL level) | **1,466** |
| NIKL-only (no TOPIK level) | 0 |

Every row carries a TOPIK grade, and NIKL's licence does not reach the 1,466 rows
that appear in no NIKL list — so nothing in that file can be called clean until
TOPIK's own terms turn up. `results.tsv` is a derived work of both, so NIKL's
share-alike does reach it. Note NIKL's own caveat: example sentences pulled from
published material are fair-use only and are **not** open; media files are not
redistributable. We use neither.

If you need to be strict, install everything except `presets_ko.db` — the Korean
grammar decks are unaffected (CC BY 4.0, hanabira.org).

Also note the app's own credits dialog lists these sources; if a licence is
missing from this table, check `src/lib/data/credits.ts` in the code repo.

---

## Licence of this repository

The data itself is not ours to license — it carries the terms above, several of
which require attribution or share-alike. Add a `LICENSE` file here once the
TOPIK question is settled and a single answer exists for what the repo as a whole
can be distributed under.

The **app** (code repo) is MIT.