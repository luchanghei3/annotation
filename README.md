

---

## 🧩 一、变异基本信息

| 列名    | 中文含义   |
| ----- | ------ |
| Chr   | 染色体    |
| Start | 变异起始位置 |
| End   | 变异结束位置 |
| Ref   | 参考碱基   |
| Alt   | 突变碱基   |

---

## 🧬 二、基因与功能注释

| 列名                        | 中文含义                     |
| ------------------------- | ------------------------ |
| Func.refGeneWithVer       | 基因组功能区域（外显子/内含子/UTR/剪接区） |
| Gene.refGeneWithVer       | 所在基因名称                   |
| GeneDetail.refGeneWithVer | 基因结构详细位置（如intron编号等）     |

---

## 🧬 三、蛋白功能影响

| 列名                        | 中文含义                  |
| ------------------------- | --------------------- |
| ExonicFunc.refGeneWithVer | 外显子变异类型（错义/无义/同义/移码等） |
| AAChange.refGeneWithVer   | 氨基酸变化信息（蛋白改变）         |

---

## 🧬 四、ClinVar 临床注释

| 列名          | 中文含义                   |
| ----------- | ---------------------- |
| CLNALLELEID | ClinVar变异编号            |
| CLNDN       | 相关疾病名称                 |
| CLNDISDB    | 疾病数据库来源（如OMIM/MedGen等） |
| CLNREVSTAT  | 临床证据等级（审查可信度）          |
| CLNSIG      | 临床意义（致病/良性/不确定）        |

---

## 🧬 五、人群频率（gnomAD）

| 列名        | 中文含义            |
| --------- | --------------- |
| AF        | 总体人群等位基因频率      |
| AF_popmax | 最大人群频率（最关键过滤指标） |
| AF_raw    | 原始频率            |
| AF_male   | 男性人群频率          |
| AF_female | 女性人群频率          |

---

## 🌍 六、不同人群频率（gnomAD分群）

| 列名     | 中文含义        |
| ------ | ----------- |
| AF_afr | 非洲人群频率      |
| AF_sas | 南亚人群频率      |
| AF_amr | 美洲混合人群频率    |
| AF_eas | 东亚人群频率      |
| AF_nfe | 非芬兰欧洲人群频率   |
| AF_fin | 芬兰人群频率      |
| AF_asj | 阿什肯纳兹犹太人群频率 |
| AF_oth | 其他人群频率      |

---

# 🧠 一句话理解整个表

👉 这个表可以拆成4层信息：

| 层级 | 内容                         |
| -- | -------------------------- |
| 1  | 变异位置（Chr/Start/End）        |
| 2  | 基因与功能（Gene/Func）           |
| 3  | 蛋白影响（ExonicFunc/AAChange）  |
| 4  | 临床+人群（ClinVar + AF/gnomAD） |

---

# 🔥 如果你做TS500 / driver分析，这些是核心

👉 最重要5列：

* Gene.refGeneWithVer
* ExonicFunc.refGeneWithVer
* CLNSIG
* AF_popmax
* AAChange.refGeneWithVer

---


