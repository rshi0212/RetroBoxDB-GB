GB Catalog, storage v4 (64 KiB blocks, 5 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- RetroAchievements ROM set imported: 720 ZIPs; 495 ROM files are also in a No-Intro DAT, 222 are only in the RA set, 3 have a hash absent from the latest RA snapshot. RA games with achievements that have a local ROM: 394 → 492.
- Source collections (`source_collections`, `v_collection_files`) and RetroAchievements links per file (`v_ra_collection`).
- ROMs outside every DAT join the family of the stored ROMs they share the most blocks with (hacks next to their original).
- Imports and DAT packaging no longer decode solid groups for already stored blocks; DAT formats (e.g. FDS/QD, NES headered/headerless) are handled separately.
- Source: 3,391 ZIPs (nointro 2,671, retroachievements 720), 401.1 MiB (3,392 ROM files, 1.05 GiB uncompressed). Populated database: 177.7 MiB (44.3% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 2,538 ROM records, 1,419 games, 2,299 releases; DAT versions: 20260602-070215, 20260707-013717, 20260814-115131, 20261001-130150.
- RetroAchievements: 492 of 505 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 41.0 MiB/s (2,232 files); single file with a cold cache 1.594 s (ROM) / 1.812 s (TorrentZip) on average.
- Full audit of the populated database: 2,546 objects, 5 groups, 2,776 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-GB/blob/main/README.zh-CN.md)
