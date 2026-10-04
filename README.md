# supii-data

This is the data half of [supii](https://github.com/logbucket/supii) — every
dictionary, analyzer model and media file the app reads, sorted by language.

**It is not the app.** Nothing here runs inside the app; `scripts/` just holds
the build tools that made these files. The app is a separate repo and reads
whatever directory this one was cloned into.

---

## Where to clone it

| OS | Path |
|---|---|
| Linux | `~/.local/share/supii` |
| macOS | `~/Library/Application Support/supii` |
| Windows | `%LOCALAPPDATA%\supii` |

```bash
git clone git@github.com:logbucket/supii-data.git \
  ~/.local/share/supii        # Linux
```

The folder has to be called `supii` — the app looks for that exact name. The
repo itself is named `supii-data`.

About **700 MB**. Use `git clone --depth 1` if you don't need history.

---

## What's in it

```
supii/
├── user.db                 ← YOUR cards, ratings, progress. Never in git.
├── assets/                 ← YOUR images. Never in git.
│
├── ja/                     ← Japanese
│   ├── jmdict.sqlite       96 MB  word dictionary
│   ├── sudachi/            208 MB analyzer
│   ├── anim_ja/            54 MB  stroke-order SVGs
│   ├── grammar_ja/         6.4 MB grammar notes
│   ├── audio_ja/           1.0 MB kana audio
│   └── presets_ja.db       704 KB JLPT decks + radicals
│
├── ko/                     ← Korean
│   ├── krdict.db           201 MB word dictionary
│   ├── anim_ko/            3.6 MB stroke SVGs
│   ├── grammar_ko/         5.5 MB grammar notes
│   ├── audio_ko/           320 KB jamo audio
│   └── presets_ko.db       1.6 MB NIKL-level vocab + grammar
│
├── zh/                     ← Chinese
│   ├── cedict.txt          9.4 MB word dictionary
│   ├── anim_zh/            65 MB stroke SVGs
│   ├── audio_zh/           34 MB pinyin audio
│   └── presets_zh.db       808 KB HSK decks
│
└── scripts/                ← build tools, with its own README
```

Each language folder owns all of its content. A Japanese-only install carries
nothing for Korean or Chinese, and the app never reaches outside the folder for
the language currently shown.

### Two dictionaries per language

Tokenizing and looking up glosses are different jobs:

| | Tokenizes | Has English glosses |
|---|---|---|
| `system.dic` (Sudachi) | yes | **no** |
| `jmdict.sqlite` | no | yes |

Korean and Chinese are the same shape: `garu-core` / `jieba-rs` split the text,
`krdict.db` / `cedict.txt` answer the lookups. So both files are always needed.

---

## What's *not* here

**Your data stays in the same folder but never commits.** `user.db` (cards,
collections, ratings, grammar progress, SRS state) and `assets/` are in
`.gitignore`. If you `git pull` and your cards vanish, you overwrote `user.db` —
restore it from backup.

**Plugin binaries are not here.** `plugins/*.so` are build output of the code
repo.

---

## Rebuilding a file

One folder per tool in `scripts/`, each with its own README:

| Rebuild | Tool |
|---|---|
| `ja/jmdict.sqlite` | `jmdict-kanjidict/` — upstream converter |
| `ko/krdict.db` | `krdict-to-sqlite/` — **our edited fork** |
| `zh/cedict.txt` | plain download |
| `*/anim_*/` | `animcjk` upstream, five folders only |
| `ja/sudachi/` | Sudachi source build |
| `*/presets_*.db` | scripts in the code repo |
| `*/audio_*/` | see the scripts README |

---

## Credits and licences

Every file here is someone else's work. Everything has confirmed terms — some
ask for attribution, some for share-alike. **Nothing blocks publication.**

### Japanese

| What | Source | Licence |
|---|---|---|
| `ja/jmdict.sqlite` | [jmdict-sqlite](https://github.com/shirakaba/jmdict-sqlite), data from [jmdict-yomitan](https://github.com/yomidevs/jmdict-yomitan) | MIT (tool) / JMdict terms (data) |
| `ja/sudachi/` | [Sudachi](https://github.com/WorksApplications/Sudachi) | Apache-2.0 |
| `ja/anim_ja/` | [animCJK](https://github.com/parsimonhi/animCJK) | Arphic Public License (characters), LGPL-3.0-or-later (kana), Unihan (`dictionary*.txt`) |
| `ja/grammar_ja/` | [hanabira.org-japanese-content](https://github.com/tristcoil/hanabira.org-japanese-content) | **CC BY 4.0 — attribution required** |
| `ja/audio_ja/` | generated in-house with Google TTS (`gTTS`) | **ours** — gTTS is MIT and was a local build tool only |
| `ja/presets_ja.db` | [OpenJLPT](https://github.com/evanclan/OpenJLPT) + [kanjium](https://github.com/mifunetoshiro/kanjium) | **CC BY-SA 4.0** / **MIT** |

### Korean

| What | Source | Licence |
|---|---|---|
| `ko/krdict.db` | [krdict-to-sqlite](https://github.com/ketzu/krdict-to-sqlite), data from the National Institute of the Korean Language | **CC BY-SA 2.0 KR** |
| `ko/anim_ko/` | [animCJK](https://github.com/parsimonhi/animCJK) | Arphic Public License |
| `ko/grammar_ko/` | hanabira (as above) | **CC BY 4.0** |
| `ko/audio_ko/` | generated in-house with `gTTS` | **ours** — same as above |
| `ko/presets_ko.db` | hanabira grammar + NIKL vocab | **CC BY 4.0** / **CC BY-SA 2.0 KR** |

### Chinese

| What | Source | Licence |
|---|---|---|
| `zh/cedict.txt` | [CC-CEDICT](https://www.mdbg.net/chinese/dictionary?page=cc-cedict) | **CC BY-SA 4.0** |
| `zh/anim_zh/` | [animCJK](https://github.com/parsimonhi/animCJK) | Arphic Public License |
| `zh/audio_zh/` | [mp3-chinese-pinyin-sound](https://github.com/davinfifield/mp3-chinese-pinyin-sound) | public domain |
| `zh/presets_zh.db` | [HSK 3.0](https://github.com/krmanik/HSK-3.0), built from CC-CEDICT + SUBTLEX-CH + Pleco | **CC BY-SA 4.0** |

### TOPIK — keep it out of the app

Pitfall worth knowing: don't reuse TOPIK level labels in anything shipped. NIIED
(the body behind TOPIK) allows free use for personal purposes only, so TOPIK
levels are not redistributable — NIKL's `vocabulary_level` from krdict is the
one licence-safe alternative, and it's what the current decks use.

If a licence is missing from these tables, check `src/lib/data/credits.ts` in
the code repo — the app's credits dialog lists the same sources.

---

## Licence of this repository

The data is not ours to license — it keeps whatever terms are listed above.
The app (code repo) is MIT.
