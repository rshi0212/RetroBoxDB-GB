GB Catalog, storage v4 (64 KiB blocks, 3 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release.
- Source: 2,671 No-Intro ZIPs, 295.8 MiB (2,672 ROM files, 820.0 MiB uncompressed). Populated database: 164.4 MiB (55.6% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 2,315 ROM records, 1,419 games, 2,299 releases; DAT versions: 20260602-070215, 20260707-013717, 20260814-115131, 20261001-130150.
- RetroAchievements: 394 of 505 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole set in storage order 33.2 MiB/s (2,672 ROM files); single file with a cold cache 1.669 s (ROM) / 1.537 s (TorrentZip) on average.
- Full audit of the populated database: 2,323 objects, 3 groups, 2,491 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-GB/blob/main/README.zh-CN.md)
