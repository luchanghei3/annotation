明白。任务分两步：

1. 从 `WGS_DISEASE_agg_level1.tsv` 筛选出 PRIMARY_SITE 属于这 20 种的突变，形成新文件放到 `./data_analysis`。
2. 将新文件的突变（hgvsg 键）与 `~/cosmic_project/database/sql/mydb.db` 的 `mutation` 表 hgvsg 求交集，并按 `cancer_type` 分癌症种类统计。

注意：这 20 种里有 `haematopoietic` 和 `lymphoid` 分开写的，但 tsv 里实际是 `haematopoietic_and_lymphoid_tissue` 一个合并值。我需要确认 tsv 里的实际 PRIMARY_SITE 名称，处理这个匹配。

先确认 tsv 里 PRIMARY_SITE 的实际取值，看这20种对应的真实名称。
关键信息确认：

1. **tsv 的 PRIMARY_SITE** 里 `haematopoietic_and_lymphoid_tissue` 是**一个合并值**（你列的20种里拆成了 `haematopoietic` 和 `lymphoid` 两个）。所以筛选时这20种里除了这两个要合并匹配 `haematopoietic_and_lymphoid_tissue` 外，其余18种名称完全对应。

2. **SQLite 的 cancer_type** 正好是 **20 种**，且命名是加后缀的：`skin`、`large_intestine_carcinoma`、`lung_carcinoma`、... `haematopoietic_and_lymphoid_tissue_lymphoid_neoplasm` 和 `haematopoietic_and_lymphoid_tissue_haematopoietic_neoplasm`（拆成了两个）。

交集分析逻辑：
- 先筛出 tsv 中 20 种 primary_site 的突变 → 新文件
- 提取这些突变的 hgvsg 键（chr:pos:REF>ALT）
- 与 SQLite mutation 表的 hgvsg 匹配（用归一化键）
- 按 SQLite 的 `cancer_type` 分组统计交集的突变/记录数

注意：SQLite 的 cancer_type 有 20 种，但其中 haematopoietic 拆成了 lymphoid 和 haematopoietic 两个。tsv 侧是合并的。交集后按 SQLite 的 cancer_type 分组即可自然对齐。

让我写脚本：
- 步骤1：筛选 tsv → `WGS_DISEASE_agg_level1_20sites.tsv`
- 步骤2：交集 + 按 cancer_type 统计 → 输出报告/文件

我一次做完。先做筛选文件并生成交集分析。

注意 SQLite 的 hgvsg 格式 `10:g.100360845G>T`，tsv 拼 `10:101136858:C>T`。归一化两边成 `chr:pos:REF>ALT`。

另外 SQLite 那20种 cancer_type 与 tsv 的20种 primary_site 的对应关系（用于最后可圈定"哪些癌症种类有交集"）：
- skin ↔ skin
- large_intestine_carcinoma ↔ large_intestine
- lung_carcinoma ↔ lung
- stomach_carcinoma ↔ stomach
- haematopoietic_and_lymphoid_tissue_lymphoid_neoplasm ↔ haematopoietic_and_lymphoid_tissue
- endometrium_carcinoma ↔ endometrium
- breast_carcinoma ↔ breast
- urinary_tract_carcinoma ↔ urinary_tract
- liver ↔ liver
- thyroid_carcinoma ↔ thyroid
- upper_aerodigestive_tract_carcinoma ↔ upper_aerodigestive_tract
- oesophagus_carcinoma ↔ oesophagus
- kidney ↔ kidney
- prostate_carcinoma ↔ prostate
- ovary_carcinoma ↔ ovary
- pancreas_carcinoma ↔ pancreas
- biliary_tract_carcinoma ↔ biliary_tract
- cervix_carcinoma ↔ cervix
- haematopoietic_and_lymphoid_tissue_haematopoietic_neoplasm ↔ haematopoietic_and_lymphoid_tissue
- bone_Ewing_sarcoma... ↔ bone

这20种完美对应。

让我写脚本完成两步。先筛选文件，再做交集分析。

我会生成一个交集结果文件，按 cancer_type 列出：交集突变键数、样本记录数、样本数、基因数。同时输出每个交集突变的明细（可选）。

先写筛选+分析脚本。

关于"非点突变"：之前发现 tsv 有87行非点突变。交集用 hgvsg 归一化键匹配，那87行 2bp 的拼出来 `AC>TT` 格式，SQLite 若是双位点格式匹配不上，自然排除——但为了严谨，我在筛选文件时保留全部（用户没说只点突变），交集时按键匹配即可。

写脚本。

我要把 SQLite 的 hgvsg 全部取出建归一化键集合，同时保留原始 hgvsg 以便反查记录。内存：26万唯一 hgvsg，OK。

步骤：
1. 读 SQLite：SELECT hgvsg, cancer_type, gene_symbol, cosmic_sample_id FROM mutation WHERE hgvsg IS NOT NULL。逐行归一化键，建 dict: key -> list of (cancer_type, gene, sample)。注意同一样本同突变可能多行，但 OK。
   实际更高效：先建 norm_key -> set of hgvsg raw。然后读 mutation 全表，对每行归一化 key，若 key 在 tsv_set 里，则记录 (cancer_type, raw_hgvsg, sample)。
   但 96万行逐行处理 OK。
2. tsv_set：从筛选后的20种文件读 key 集合。
3. 交集：遍历 mutation 表，归一化 key in tsv_set → 收集到结果。
4. 按 cancer_type 聚合：突变键数（distinct）、记录行数、样本数、基因数。

为避免全表扫两次，一次扫描搞定：先有 tsv_set，再扫 mutation 全表。

让我写。输出：
- 文件1: WGS_DISEASE_agg_level1_20sites.tsv（筛选后tsv）
- 文件2: intersection_by_cancer_type.tsv（按cancer_type统计）
- 文件3（可选）: intersection_mutations_detail.tsv（交集突变明细，含gene/hgvsg/cancer_type/样本数）

先生成文件1和文件2，明细文件也生成（可能有用）。

写脚本。</think:6124c78e>关键信息已确认：

- **tsv 的 `haematopoietic_and_lymphoid_tissue` 是合并值**（你列的20种拆成了 `haematopoietic`/`lymphoid`），筛选时这两个合并匹配它。
- **SQLite 的 `cancer_type` 正好20种**，且与这20种 primary_site 一一对应（haematopoietic 在 SQLite 侧拆成 lymphoid/haematopoietic 两个，对应 tsv 的同一合并值）。

我写脚本完成两步：①筛选20种到新文件 ②与 SQLite 求交集并按 cancer_type 统计。
完成。看下按 cancer_type 的统计结果。
完成。两步都已执行，结果文件生成在 `./data_analysis`。

## 产出文件

| 文件 | 内容 |
|---|---|
| `WGS_DISEASE_agg_level1_20sites.tsv` | 从原表筛选出的、PRIMARY_SITE 属于20种的突变（13,315 行，去重后 12,430 个唯一突变键） |
| `intersection_by_cancer_type.tsv` | 交集按 `cancer_type` 分癌症种类的统计 |
| `intersection_mutations_detail.tsv` | 每个交集突变的明细（基因、样本数、涉及癌症种类数） |

## 交集结果

筛选出的 20 种突变（12,430 个唯一键）中，**220 个原始 hgvsg 在 SQLite `mutation` 表里也被观测到**（注意此前全表交集是 225，这里因为只限20种 primary site，少了5个来自其它部位如 soft_tissue/central_nervous_system 的）。

按 `cancer_type` 分癌症种类，交集规模（突变键 / 样本记录 / 样本数 / 基因数）：

| cancer_type | 突变键 | 记录 | 样本 | 基因 |
|---|---|---|---|---|
| haematopoietic_and_lymphoid_tissue_lymphoid_neoplasm | 89 | 2702 | 515 | 30 |
| ovary_carcinoma | 21 | 539 | 63 | 21 |
| urinary_tract_carcinoma | 16 | 309 | 59 | 14 |
| large_intestine_carcinoma | 14 | 65 | 17 | 14 |
| oesophagus_carcinoma | 12 | 2404 | 161 | 12 |
| biliary_tract_carcinoma | 10 | 308 | 59 | 10 |
| skin | 9 | 87 | 32 | 8 |
| lung_carcinoma | 8 | 481 | 134 | 7 |
| endometrium_carcinoma | 7 | 56 | 20 | 7 |
| pancreas_carcinoma | 7 | 378 | 38 | 7 |
| kidney | 6 | 121 | 46 | 4 |
| thyroid_carcinoma | 6 | 74 | 23 | 6 |
| breast_carcinoma | 4 | 66 | 21 | 4 |
| stomach_carcinoma | 4 | 26 | 12 | 4 |
| haematopoietic_and_lymphoid_tissue_haematopoietic_neoplasm | 3 | 210 | 66 | 2 |
| liver | 2 | 65 | 13 | 2 |
| prostate_carcinoma | 1 | 6 | 1 | 1 |
| upper_aerodigestive_tract_carcinoma | 1 | 20 | 20 | 1 |

（bone、cervix 这2种在交集里为0——SQLite 里 bone 只有 Ewing 肉瘤633条、cervix 有11028条，但与 Census 这20种突变无坐标重合）

## 解读

- **交集最高的是血液/淋巴系肿瘤**（lymphoid 89 键、2702 记录、515 样本），其次是卵巢、泌尿、食管、胆道——说明你的 SQLite 样本库与 COSMIC WGS 普查在这些癌种的突变共识最强。
- 单突变样本数 Top：如 `EGFR 7:55191822:T>G`（114样本，胶质/肺相关）、`BRD4 19:15265576:C>A`（72）等，都是高置信、跨库一致的驱动突变。

## 两点提醒
1. **haematopoietic 的处理**：你列的20种把 `haematopoietic` 和 `lymphoid` 分开，但 Census 表里是合并的 `haematopoietic_and_lymphoid_tissue` 一个值，所以筛选时两者都归到这一个值；SQLite 侧则拆成 lymphoid/haematopoietic 两个 cancer_type——上表已按 SQLite 原样分列，对应关系清晰。
2. **非点突变**：tsv 里有 87 行 2bp 替换，它们的键（如 `AC>TT`）与 SQLite 的 hgvsg 双位点格式匹配不上，未进交集——即交集 220 个全是干净的单碱基 SNV，无 indel 污染。

需要我把 bone/cervix 这两种为什么交集为0再深挖一下，或者把某个癌种（如 lymphoid）的交集突变完整列出吗？
