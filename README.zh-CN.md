# RetroBoxDB GB

[English](README.md) | 中文

任天堂 Game Boy的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 3,391 个，401.1 MiB（No-Intro 2,671 个，RetroAchievements 集合 720 个）；解压后 ROM 3,392 个，1.05 GiB |
| 入库后大小 | 完整库 177.7 MiB；公开 Catalog 35.5 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 44.3%，为解压后 ROM 总量的 16.5% |
| 使用的技术 | 存储 v4：64 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。全集合顺序导出（3,392 个 ROM 文件，每组解压一次）：21.4 MiB/s，平均 15 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 1.594 秒，TorrentZip 平均 1.812 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.GB.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-GB/releases/latest/download/RetroBoxDB.GB.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 七个平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-gb-games.csv)／[汇总](reports/ra-gb.json)、[构建报告](reports/gb-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

真实全量数据（全部 18 个组，551 MiB）上相对 32 MiB 组的变化：64 MiB −1.31%，128 MiB −2.36%，256 MiB −3.73%；按规则采用 256 MiB。

- ROM 为 32 KiB–2 MiB，整库只有几百 MiB，大组里会放进很多游戏族，增益主要来自跨族的相似内容（同一引擎、同一系列）。代价是读取延迟：读一个小 ROM 可能要解压所在组的大部分。
- 0x100–0x14F 卡带头：标题、CGB／SGB 标志、新旧授权商、卡带类型（MBC、电池、RTC、震动）、ROM／RAM 大小代码、目的地、版本、头部校验和（启动 ROM 会检查）和全局校验和。Nintendo Logo 只存 SHA1，`v_gb_headers.logo_is_common` 标出与多数卡带一致的 Logo。头部校验和无效的主要是盗版多合一卡。
- ROM 目录里有两个前端文本文件（`metadata.txt`、`systeminfo.txt`），作为 `metadata` 文件保存，不当作 ROM。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 2,538／1,419／2,299 |
| 各版 DAT 覆盖 | 20260602-070215：2,225/2,276；20260707-013717：2,226/2,284；20260814-115131：2,230/2,292；20261001-130150：2,232/2,299 |
| 不在任何 DAT 的本地 ROM | 306 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 495，仅 RA 收录 222，哈希不在最新 RA 快照 3（[清单](reports/ra-gb-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-gb-missing.csv) |
| No-Intro DB Export＋Dump Log 20261001-130150 | 2,335 个档案、2,450 个文件身份、2,810 条有文档的硬件声明；Dump Log Verified 719 |
| RetroAchievements（console 4） | 有成就的游戏 505 个：本地有 ROM 492（725 个 ROM），仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 13 |
| 中文名 | 1,974 条记录中 1,816 条有中文（1,237 个唯一名）；本地 ROM 1,835 个有中文名 |
| 完整库审计 | 2,546 个对象、5 个组、2,776 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.GB.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.GB.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.GB.sqlite --discover --ra --catalog RetroBoxDB.GB.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
