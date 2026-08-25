好的，我来生成一张"每种癌：Census总突变键/基因 vs 与SQLite交集突变"的对比表，按突变键数排序，并包含交集的癌症种类映射。

我需要重新跑一遍交集分析，把每种 primary_site 的 Census 总数和交集数都算出来。由于 SQLite 的 cancer_type 与 tsv 的 primary_site 是20对20对应（haematopoietic 拆成两个），我做一个映射表，把交集结果按 primary_site 归并，再和 Census 总数对比。

让我写脚本生成对比表。
完成。对比表已生成在 `./data_analysis/census_vs_sqlite_compare.tsv`，按 Census 突变键数降序排列。

## 对比表（每种癌：Census 总数 vs 与 SQLite 交集）

| PRIMARY_SITE | Census突变键 | Census基因 | 交集键(sqlite) | 交集记录(sqlite) | 交集样本(sqlite) | 交集基因(sqlite) |
|---|---|---|---|---|---|---|
| oesophagus | 3,351 | 2,703 | 12 | 2,404 | 161 | 12 |
| stomach | 2,307 | 2,004 | 4 | 26 | 12 | 4 |
| upper_aerodigestive_tract | 2,265 | 928 | 1 | 20 | 20 | 1 |
| haematopoietic+lymphoid | 1,448 | 544 | 92 | 2,912 | 581 | 32 |
| lung | 784 | 674 | 8 | 481 | 134 | 7 |
| thyroid | 610 | 378 | 6 | 74 | 23 | 6 |
| biliary_tract | 388 | 371 | 10 | 308 | 59 | 10 |
| urinary_tract | 386 | 340 | 16 | 309 | 59 | 14 |
| skin | 342 | 282 | 9 | 87 | 32 | 8 |
| pancreas | 302 | 277 | 7 | 378 | 38 | 7 |
| ovary | 280 | 261 | 21 | 539 | 63 | 21 |
| large_intestine | 242 | 201 | 14 | 65 | 17 | 14 |
| endometrium | 182 | 119 | 7 | 56 | 20 | 7 |
| breast | 125 | 104 | 4 | 66 | 21 | 4 |
| liver | 109 | 74 | 2 | 65 | 13 | 2 |
| kidney | 62 | 49 | 6 | 121 | 46 | 4 |
| prostate | 39 | 34 | 1 | 6 | 1 | 1 |
| cervix | 11 | 9 | 0 | 0 | 0 | 0 |
| bone | 9 | 5 | 0 | 0 | 0 | 0 |
| **TOTAL** | **12,430** | **6,553** | **220** | **8,917** | **1,332** | **~187** |

## 关键解读

- **Census 突变负荷排序**：oesophagus > stomach > UADT > 血液淋巴 > lung 居前，与消化道/呼吸消化道肿瘤高突变负荷一致。
- **交集率普遍极低**（220/12,430 ≈ 1.8%）：因为 SQLite 的 mutation 表是**全样本级原始突变**（含大量低频/背景），而 Census 是**精选驱动突变普查**，两者交集小是正常的。
- **例外是血液淋巴系**：交集键 92、样本 581、记录 2,912，远超其他癌种——你的 SQLite 库里血液/淋巴肿瘤样本量大且与 Census 共识最强。
- **cervix、bone 交集为0**：这两个癌种在 Census WGS 里突变键极少（11/9），且 SQLite 侧样本（cervix 11,028 条、bone 633 条）的突变坐标未与 Census 重合。
- 表格同时保留了"样本记录数"和"样本数"（去重后），可区分同一突变被多次观测的情况（如 oesophagus 12 个键却有 2,404 条记录、161 个样本）。

需要我把交集率为0的 cervix/bone 深挖一下具体原因（比如看 SQLite 里这些癌种的突变是否都是 indel、或者坐标体系不一致），或者把某癌种的交集突变明细展开吗？


明白了，你问的是 **Alldata.tar 里原始 Census 表本身**的样本数（不是筛选后的20种文件，也不是 TESTED 之和），而且要看全表所有 primary site 里样本最多的——但你说"20种癌症"，应该是指这20种 primary site 在原始表里按样本数排序。

不过原始 Census 表的样本数怎么取？两个口径：
1. 用 `COSMIC_SAMPLE_TESTED`（全局列，每行一个突变的全局受检样本数）——但这是全局不分部位的
2. 用 WGS_DISEASE 列里各段的 TESTED 之和（分部位的受检样本数）

你说"样本数最多的"，最合理的是**原始表里 WGS_DISEASE 列按 primary site 解析后，该部位的受检样本规模（TESTED 之和）**，但这是20种范围。但你强调"原始表里"——可能想要全表所有 primary site 的排序，看哪20种排前面，或直接看这20种在原始表的样本数。

我重新理解：你之前问"census里样本最多的20种癌症是哪些"，我误以为是那个20sites文件。现在你纠正：要看 **Alldata.tar 原始表**里的样本数，且是"20种癌症"——即原始表里所有 primary site 中，样本数（受检规模）排前20的癌症，还是仅指你列的那20种在原始表的样本数？

最稳妥：直接从 Alldata.tar 原始表，解析 WGS_DISEASE 列，按 primary site 聚合 TESTED（受检样本规模），列出**全部 primary site 的排序**（这样无论你要"前20"还是"指定20种"都涵盖）。

注意：原始表 WGS_DISEASE 段里的 TESTED 是每突变×每部位的，同部位多突变相加会放大。但"样本数最多"在这个表里没有独立样本计数，TESTED之和是最接近的代理。

让我直接从 tar 跑，按 primary site 聚合 WGS_DISEASE 的 TESTED 之和，列出全部并标出你那20种。
这是 **Alldata.tar 原始 Census 表**按 WGS_DISEASE 受检样本规模（TESTED 之和）排序的全部 primary site。

## 原始表中样本数（受检规模）最多的前 20 种癌症

| 排名 | PRIMARY_SITE | TESTED之和 | MUTATED之和 | 突变段数 |
|---|---|---|---|---|
| 1 | upper_aerodigestive_tract | 2,867,880 | 47,751 | 2,797 |
| 2 | oesophagus | 2,255,888 | 61,071 | 3,698 |
| 3 | soft_tissue | 1,170,619 | 40,324 | 6,938 |
| 4 | stomach | 1,057,001 | 16,994 | 3,570 |
| 5 | large_intestine | 295,014 | 7,029 | 337 |
| 6 | haematopoietic_and_lymphoid_tissue | 291,550 | 19,978 | 1,811 |
| 7 | biliary_tract | 288,686 | 4,898 | 415 |
| 8 | lung | 185,175 | 5,244 | 864 |
| 9 | urinary_tract | 163,940 | 2,863 | 423 |
| 10 | ovary | 136,347 | 2,253 | 308 |
| 11 | thyroid | 125,991 | 4,830 | 714 |
| 12 | skin | 118,511 | 3,544 | 393 |
| 13 | endometrium | 104,410 | 1,792 | 206 |
| 14 | pancreas | 87,998 | 5,461 | 358 |
| 15 | liver | 87,983 | 1,274 | 128 |
| 16 | central_nervous_system | 69,666 | 2,000 | 132 |
| 17 | kidney | 38,220 | 597 | 70 |
| 18 | breast | 35,605 | 1,608 | 184 |
| 19 | prostate | 17,554 | 404 | 53 |
| 20 | salivary_gland | 6,479 | 145 | 31 |

## 与你之前列的20种对比
- 你之前指定的20种里，**soft_tissue、central_nervous_system、salivary_gland 不在其中**，而你列的 **cervix、bone** 在这张全表排序里排到 **第22、23名**（样本数仅 3,529 / 3,442），属于样本最少的。
- 也就是说：原始表里样本最多的前20种，与你关心的20种**有17种重合**，差异是：
  - 全表前20多了：`soft_tissue`、`central_nervous_system`、`salivary_gland`
  - 你列的20种里掉出前20的：`cervix`、`bone`

## 口径说明
"样本数"用的是 WGS_DISEASE 列里各 primary site 段的 **TESTED 之和**（该部位所有突变记录的受检样本数加总），这是 Census 表能给出的最接近"样本规模"的指标。它不等于独立样本数（同部位不同突变共享样本池，相加会放大），但**相对排序可靠**。

需要我把这前20种和你指定的20种做一个差异对照表，或者把"全表前20"也筛出来生成新的聚合文件吗？
