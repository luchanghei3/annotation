你是问：在 `WGS_DISEASE_agg_level1_20sites.tsv`（筛选出的20种 primary site 突变）里，按 primary site 分，每种癌症总共有多少个突变键、多少个基因。

注意：tsv 里 `haematopoietic_and_lymphoid_tissue` 是合并值（对应你列的 haematopoietic + lymphoid）。我先统计这20种文件里按 PRIMARY_SITE 的突变键数和基因数。

突变键用之前归一化的 `chr:pos:REF>ALT`（跨 transcript 去重），基因用 GENE_NAME。
结果如下（来自 `WGS_DISEASE_agg_level1_20sites.tsv`，按 PRIMARY_SITE 分）：

## 20种癌症的突变键数 / 基因数

> 注：`haematopoietic` 和 `lymphoid` 在你的20种里分开，但 Census 表里合并为 `haematopoietic_and_lymphoid_tissue` 一个值，下方以"合并"显示；其余18种名称一一对应。

| PRIMARY_SITE | 突变键数 | 基因数 |
|---|---|---|
| biliary_tract | 388 | 371 |
| bone | 9 | 5 |
| breast | 125 | 104 |
| cervix | 11 | 9 |
| endometrium | 182 | 119 |
| haematopoietic+lymphoid（合并）| 1,448 | 544 |
| kidney | 62 | 49 |
| large_intestine | 242 | 201 |
| liver | 109 | 74 |
| lung | 784 | 674 |
| oesophagus | 3,351 | 2,703 |
| ovary | 280 | 261 |
| pancreas | 302 | 277 |
| prostate | 39 | 34 |
| skin | 342 | 282 |
| stomach | 2,307 | 2,004 |
| thyroid | 610 | 378 |
| upper_aerodigestive_tract | 2,265 | 928 |
| urinary_tract | 386 | 340 |

## 合计
- **20种总突变键（跨 site 去重）：12,430**
- **20种总基因（跨 site 去重）：6,553**

### 几个观察
- 突变键最多的前三是：**oesophagus（3,351）> stomach（2,307）> upper_aerodigestive_tract（2,265）**，与消化道/上呼吸消化道肿瘤突变负荷高一致。
- `haematopoietic+lymphoid` 合并值虽只有 1,448 个键，但覆盖 **544 个基因**（基因多样性高，因为血液肿瘤涉及大量不同驱动基因）。
- `bone`、`cervix`、`prostate` 这三种突变键很少（9/11/39），与之前交集为0或极少相符——它们不是 Census WGS 的主要富集癌种。
- 基因数普遍接近突变键数（如 oesophagus 2,703 基因 / 3,351 键），说明大多数基因只有1个突变键被收录（Census 里每个基因通常只收少数热点突变）。

需要我把这20种按"突变键数"排序导出成文件，或者和上一轮的交集结果做成一张"每种癌：Census总突变 vs 与SQLite交集突变"的对比表吗？
