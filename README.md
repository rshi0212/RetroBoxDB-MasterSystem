# RetroBoxDB SMS

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Sega Master System / Mark III. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 1,875 source ZIPs, 177.4 MiB (No-Intro 1,674, RetroAchievements sets 201); 1,875 ROM files, 392.5 MiB uncompressed |
| Stored size | populated database 84.6 MiB; public Catalog 18.8 MiB (no ROM data) |
| Ratio | 47.7% of the source ZIPs, 21.6% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 128 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (1,189 files, each checked against the DAT hashes): 43.9 MiB/s, 5 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 1.783 s, TorrentZip 1.799 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.SMS.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-SMS/releases/latest/download/RetroBoxDB.SMS.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-sms-games.csv) / [summary](reports/ra-sms.json), [build report](reports/sms-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

22 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-sms.json`): smallest 256 KiB / 256 MiB at 62.50 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 128 KiB / 256 MiB at 62.61 MiB. ZIPs 177.42 MiB, per-file LZMA 105.80 MiB.

- ROMs are 8 KiB–1 MiB (a few aftermarket titles reach 4 MiB); the No-Intro folders hold 1,177 licensed and 497 aftermarket/private ZIPs, so homebrew is a large part of the collection. Four local DAT versions are imported and diffed against the newest.
- Header: `TMR SEGA` at 0x7FF0 (0x3FF0 and 0x1FF0 are also read, with a warning, because the export BIOS only reads 0x7FF0): product code, version, region and declared ROM size; the checksum is recomputed over the declared range (the 16 bytes before 0x8000 are excluded). Codemasters (date, checksum pair) and SDSC homebrew headers at 0x7FE0 are recorded. Japanese Mark III cartridges usually have no header and stay `unclassified`.
- RetroAchievements console 11 (whole-file MD5).

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 1,230 / 774 / 1,238 |
| DAT coverage per version | 20260527-203639: 1,191/1,215; 20260706-223420: 1,191/1,216; 20260809-210908: 1,191/1,240; 20260918-065535: 1,189/1,238 |
| Local ROMs in no DAT | 39 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 169, RA only 32, hash not in the latest RA snapshot 0 ([list](reports/ra-sms-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-sms-missing.csv) |
| No-Intro DB Export + Dump Log 20260918-065535 | 1,245 archives, 1,279 file identities, 367 documented hardware assertions; Dump Log Verified 435 |
| RetroAchievements (console 11) | 183 games with achievements: 181 with a local ROM (235 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 2 without a No-Intro counterpart |
| Chinese names | 542 of 699 rows translated (342 unique); 563 local ROMs have a Chinese name |
| Populated-database audit | 1,236 objects, 2 groups, 1,523 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.SMS.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.SMS.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.SMS.sqlite --discover --ra --catalog RetroBoxDB.SMS.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
