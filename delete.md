这个问题很关键，尤其你是**中国人群样本**，AF 选错会直接影响结果偏倚。

我帮你按“科研规范 + 实际可操作”讲清楚👇

---

# ✅ 一、先给结论（最重要）

## 👉 如果你只有一个过滤标准：

### ✔️ **优先用：AF_popmax**

> ❗这是最推荐的过滤指标（论文级标准）

---

# 🧠 二、为什么不是 AF / AF_raw？

## ❌ 1. AF（总体频率）

问题：

* 是所有人群混在一起
* 欧洲人占比通常最大（gnomAD等数据库）

👉 对中国人不适配

---

## ❌ 2. AF_raw

问题：

* 未校正/未分人群结构
* 可能包含测序偏差

👉 基本不建议作为过滤标准

---

## ❌ 3. AF_male / AF_female

问题：

* 只是性别分层
* 跟人群频率无关

👉 只用于：

* 性别相关疾病分析
* 不用于 variant filtering

---

# ⭐ 三、为什么 AF_popmax 是关键（核心逻辑）

## 👉 AF_popmax 定义：

> 在所有人群中“最高的那个频率”

例如：

| population | AF         |
| ---------- | ---------- |
| EUR        | 0.02       |
| AFR        | 0.001      |
| EAS（东亚）    | 0.03 ← max |

👉 AF_popmax = 0.03

---

## ✔️ 它的意义：

### 👉 防止“假 rare variant”

如果你不用 popmax：

* 某 variant 在欧洲很常见（AF=0.05）
* 但在东亚很罕见（AF=0.001）

你可能会错误认为它是“rare variant”

---

## ✔️ popmax解决的问题：

> 保证“在任何人群中都不常见”

---

# 🇨🇳 四、你的情况（中国人群）

你是：

> Chinese / East Asian cohort

---

## ✔️ 最优策略（强烈推荐）

### Step 1（必须）：

用 **AF_popmax 过滤**

### Step 2（更严格，可选）：

再加：

* AF_EAS（东亚频率） < 0.01 或 0.05

---

# 🔥 五、最佳实践组合（论文级）

## ✔️ 推荐 pipeline：

### ① functional filter

* keep: LoF / missense / splice

---

### ② frequency filter（核心）

#### 严格版（rare variant study）：

* AF_popmax < 0.01

#### 宽松版：

* AF_popmax < 0.05

---

### ③ optional（针对中国人群优化）

* AF_EAS < 0.01（更精准）

---

# ⚠️ 六、一个重要误区

很多人会犯这个错误：

> “我是中国人 → 只用 EAS AF”

❌ 错误原因：

* EAS sample size 比 EUR 小
* 某些 variant 会被低估
* 会漏掉“全球 common variant”

---

# 🧠 七、一句话总结

👉 **AF_popmax 是跨人群的“最保守频率”，适合所有种族分析，是过滤 common variant 的黄金标准；AF_raw 和 sex-specific AF 不适合做过滤依据。**

---

如果你要，我可以帮你再升级一步👇

👉 给你写一套 **VCF filtering bash + bcftools pipeline（直接能跑）**
👉 或帮你设计“中国人群专用 variant filtering strategy（论文方法段）”
