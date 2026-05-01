| 类型    | 数据来源                         |
| ----- | ---------------------------- |
| 基因来源  | panel list（Dynegene / TS500） |
| 突变频率  | COSMIC + TCGA                |
| 生物学属性 | COSMIC Cancer Gene Census    |
| 临床属性  | OncoKB                       |
| 技术属性  | 自定义规则                        |


🔧 二、字段逐个怎么实现（重点）
1️⃣ Gene + Panel_source
✔ 方法
cat dynegene.txt ts500.txt | sort | uniq

然后标记来源：

Gene	Panel_source
EGFR	Both
KRAS	TS500

👉 Python/pandas：

df['Panel_source'] = df.apply(...)
2️⃣ COSMIC突变频率

来源：COSMIC mutation census / gene summary

✔ 方法（推荐两种）
方法A（简单）

用 COSMIC gene-level 数据：

“% of samples mutated”
方法B（更准）

自己算：

freq = mutated_samples / total_samples

👉 按癌种 or pan-cancer 都可以

3️⃣ TCGA突变频率
来源：MC3 MAF（最标准）

👉 步骤：

# 按基因统计
freq = df.groupby('Hugo_Symbol')['Tumor_Sample_Barcode'].nunique() / total_samples

👉 或直接用：

cBioPortal（更方便）
4️⃣ 是否有 hotspot（关键）

👉 直接用 hotspot 数据库：

✔ 推荐来源
Cancer Hotspots
✔ 实现
if gene in hotspot_db:
    hotspot = "Yes"
else:
    hotspot = "No"

👉 或更严格：

≥2 independent occurrences at same AA site
5️⃣ 是否 driver

👉 三种方法（推荐组合）

✔ 方法1（最简单）

来自 CGC：

if gene in CGC:
    driver = Yes
✔ 方法2（更严谨）

结合：

OncoKB（是否有 oncogenic mutation）
hotspot
✔ 方法3（高级）

用 dNdScv / MutSig（研究级）

👉 实际项目里：

CGC ∪ OncoKB → driver
6️⃣ 是否在 CGC

👉 最简单

if gene in CGC_list:
    CGC = Yes

来源：

COSMIC Cancer Gene Census
7️⃣ 技术风险（这个最容易乱，我帮你定标准）

👉 你可以直接用规则打标签：

✔ High risk（强烈建议标）
HLA区（如 HLA-A）
高同源基因（pseudogene）
GC-rich / repeat 区域
✔ Medium risk
大基因（TTN）
CNV主导基因
fusion-only 基因（如 ALK）
✔ Low risk
hotspot SNV gene（如 BRAF）
✔ 实现（规则化）
if gene in HLA_list:
    risk = "High"
elif gene in fusion_genes:
    risk = "Medium"
else:
    risk = "Low"
🧩 三、整合（核心代码逻辑）
df = gene_list

df = df.merge(cosmic_freq)
df = df.merge(tcga_freq)
df = df.merge(cgc_flag)
df = df.merge(hotspot_flag)
df = df.merge(oncokb_flag)

df['driver'] = ...
df['tech_risk'] = ...
📊 四、最终表结构（你要的）
Gene	Panel_source	COSMIC_freq	TCGA_freq	Hotspot	Driver	CGC	Tech_risk
EGFR	Both	12%	15%	Yes	Yes	Yes	Low
HLA-A	TS500	1%	2%	No	No	No	High
🚨 五、最关键提醒（很多人会做错）

👉 不要把：

CGC = driver
高频 = 有用
OncoKB = 全覆盖

👉 这些都是不完全成立的
