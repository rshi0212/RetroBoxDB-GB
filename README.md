# RetroBoxDB GB

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Game Boy. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 2,671 No-Intro ZIPs, 295.8 MiB; 2,672 ROM files, 820.0 MiB uncompressed |
| Stored size | populated database 164.4 MiB; public Catalog 33.6 MiB (no ROM data) |
| Ratio | 55.6% of the source ZIPs, 20.0% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 64 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. Whole-set export (2,672 ROM files in storage order, each group decoded once): 33.2 MiB/s, 9 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 1.669 s, TorrentZip 1.537 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.GB.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GB/releases/latest/download/RetroBoxDB.GB.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for all six platforms |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-gb-games.csv) / [summary](reports/ra-gb.json), [build report](reports/gb-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

Change against 32 MiB groups on real data (all 18 groups, 551 MiB): 64 MiB −1.31%, 128 MiB −2.36%, 256 MiB −3.73%; 256 MiB chosen by the rule.

- ROMs are 32 KiB–2 MiB; the whole library is a few hundred MiB, so even large groups hold many families and cross-family similarity (shared engines, series) is what the bigger groups gain. The cost is read latency: a single small ROM may require decoding a large part of its group.
- Cartridge header at 0x100–0x14F: title, CGB/SGB flags, old/new licensee, cartridge type (MBC, battery, RTC, rumble), ROM/RAM size codes, destination, version, header checksum (verified by the boot ROM) and global checksum. Only a SHA1 of the Nintendo logo bitmap is stored; `v_gb_headers.logo_is_common` compares it with the most frequent value. Invalid header checksums occur mainly on pirate multicarts.
- The ROM folder contains two frontend text files (`metadata.txt`, `systeminfo.txt`); they are stored as `metadata` files, not ROMs.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 2,315 / 1,419 / 2,299 |
| DAT coverage per version | 20260602-070215: 2,218/2,276; 20260707-013717: 2,218/2,284; 20260814-115131: 2,222/2,292; 20261001-130150: 2,224/2,299 |
| Local ROMs in no DAT | 91 |
| No-Intro DB Export + Dump Log 20261001-130150 | 2,335 archives, 2,450 file identities, 2,810 documented hardware assertions; Dump Log Verified 719 |
| RetroAchievements (console 4) | 505 games with achievements: 394 with a local ROM (506 ROMs), 2 DAT only, 0 DB file only, 109 without a No-Intro counterpart |
| Chinese names | 1,816 of 1,974 rows translated (1,237 unique); 1,829 local ROMs have a Chinese name |
| Populated-database audit | 2,323 objects, 3 groups, 2,491 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GB.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.GB.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.GB.sqlite --discover --ra --catalog RetroBoxDB.GB.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
