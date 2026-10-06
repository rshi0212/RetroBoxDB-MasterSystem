# RetroBoxDB SMS

[English](README.md) | 中文

世嘉 Master System／Mark III的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 1,875 个，177.4 MiB（No-Intro 1,674 个，RetroAchievements 集合 201 个）；解压后 ROM 1,875 个，392.5 MiB |
| 入库后大小 | 完整库 84.6 MiB；公开 Catalog 18.8 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 47.7%，为解压后 ROM 总量的 21.6% |
| 使用的技术 | 存储 v4：128 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，1,189 个文件，逐个按 DAT 哈希校验）：43.9 MiB/s，平均 5 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 1.783 秒，TorrentZip 平均 1.799 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.SMS.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-SMS/releases/latest/download/RetroBoxDB.SMS.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-sms-games.csv)／[汇总](reports/ra-sms.json)、[构建报告](reports/sms-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 22 种块／组组合（`assessment/data/storage-experiment-sms.json`）：最小为 256 KiB / 256 MiB 62.50 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 128 KiB / 256 MiB 62.61 MiB。ZIP 177.42 MiB，逐文件 LZMA 105.80 MiB。

- ROM 为 8 KiB–1 MiB（少数新作达 4 MiB）；No-Intro 目录有 1,177 个授权 ZIP 和 497 个 Aftermarket／Private ZIP，自制游戏占了相当比例。导入本地 4 版 DAT，与最新版做差异。
- 头部：0x7FF0 的 `TMR SEGA`（也读取 0x3FF0、0x1FF0，并给出告警，因为海外版 BIOS 只读 0x7FF0）：产品码、版本、地区和声明容量；按声明范围重算校验和（0x8000 前的 16 字节不计）。记录 0x7FE0 的 Codemasters 头（日期、校验和对）和 SDSC 自制软件头。日版 Mark III 卡带大多没有这个头，记为 `unclassified`。
- RetroAchievements 主机 11（整文件 MD5）。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 1,230／774／1,238 |
| 各版 DAT 覆盖 | 20260527-203639：1,191/1,215；20260706-223420：1,191/1,216；20260809-210908：1,191/1,240；20260918-065535：1,189/1,238 |
| 不在任何 DAT 的本地 ROM | 39 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 169，仅 RA 收录 32，哈希不在最新 RA 快照 0（[清单](reports/ra-sms-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-sms-missing.csv) |
| No-Intro DB Export＋Dump Log 20260918-065535 | 1,245 个档案、1,279 个文件身份、367 条有文档的硬件声明；Dump Log Verified 435 |
| RetroAchievements（console 11） | 有成就的游戏 183 个：本地有 ROM 181（235 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 2 |
| 中文名 | 699 条记录中 542 条有中文（342 个唯一名）；本地 ROM 563 个有中文名 |
| 完整库审计 | 1,236 个对象、2 个组、1,523 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.SMS.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.SMS.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.SMS.sqlite --discover --ra --catalog RetroBoxDB.SMS.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
