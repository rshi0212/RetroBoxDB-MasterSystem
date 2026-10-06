Master System Catalog, storage v4 (128 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release: Master System / Mark III cartridges (TMR SEGA, Codemasters and SDSC headers) from the No-Intro folders including Aftermarket and Private, and the RetroAchievements Master System set; 4 Parent-Clone DAT versions, DB Export and Dump Log 20260918-065535, RetroAchievements snapshot and Chinese names.
- Storage measured on the whole collection: 128 KiB blocks, 256 MiB groups (`assessment/data/storage-experiment-sms.json`).
- Source: 1,875 ZIPs (nointro 1674, retroachievements 201), 177.4 MiB (1,875 ROM files, 392.5 MiB uncompressed). Populated database: 84.6 MiB (47.7% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 1,230 ROM records, 774 games, 1,238 releases; DAT versions: 20260527-203639, 20260706-223420, 20260809-210908, 20260918-065535.
- RetroAchievements: 181 of 183 games with achievements have a local ROM.
- Chinese names: 542 of 699 CSV rows translated; 563 local ROMs with Chinese names.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 43.9 MiB/s (1,189 files); single file with a cold cache 1.783 s (ROM) / 1.799 s (TorrentZip) on average.
- Full audit of the populated database: 1,236 objects, 2 groups, 1,523 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-SMS/blob/main/README.zh-CN.md)
