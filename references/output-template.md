# Output card template (**never use tables**)

Hard requirement: each recommended paper is presented as a **card**, and the abstract is **translated into the user's conversation language** (never pasted in raw English).

## Standard card format

The example below is rendered for a Chinese-speaking user; for other users translate the abstract into their language instead.

```
Found N candidates, M kept after triage:

### [1] Optimal ZVS Modulation of Single-Phase Single-Stage Bidirectional DAB AC–DC Converters

🔗 https://doi.org/10.1109/tpel.2013.2292026
📅 2014 · 📍 IEEE TPEL · 📊 citations 471
👤 Jordi Everts; Florian Krismer; Jeroen Van den Keybus; Johan Driesen; Johann W. Kolar (ETH/KU Leuven)

**中文摘要**：提出针对单相单级双向 DAB AC-DC 转换器的全工况零电压开关（ZVS）调制方案推导流程。传统方法基于电流/能量分析，本文用更精确的电流相关电荷分析，考虑开关寄生输出电容的充电电荷需求。证明"换流电感"是实现全工况 ZVS 的关键要素。方法应用于 3.7 kW 双向单位功率因数电动汽车充电器（400V DC母线 / 230V 50Hz 电网），高功率密度高效率原型实验验证。

💡 **Verdict**: **must-read** (field anchor, 471 citations, the founding paper of charge-based ZVS analysis)

---

### [2] Closed-Form Solution for Efficient ZVS Modulation of DAB Converters

🔗 https://doi.org/10.1109/tpel.2016.2633507
📅 2016 · 📍 IEEE TPEL · 📊 citations 159
👤 Jordi Everts

**中文摘要**：给出双向 DAB DC-DC 转换器全工况 ZVS 调制方案的**闭式解析解**——可直接代入控制器使用。和已有解析方案不同，本文考虑了 MOSFET 输出电容 Coss 的充电电荷，保证换流彻底完成。除准无损 ZVS 外，方案还逼近最小高频环流电流，最小化通态损耗。3.7kW 原型实验验证（400V 电池 / 230V/50Hz 电网）。

💡 **Verdict**: **representative** (Everts follow-up, closed-form solution usable in engineering)

---

Dropped X papers (state the reason):
- "Akshara to Prosodeme Converter in Hindi" — NLP noise, false hit
- "Integrated Magnetics for ZVS Cuk Converter" — Cuk topology, not DAB

**Next step**: which ones do you want in full text? (give me the numbers and I fetch them with ieee-fetch)
```

## Translation requirements

**Always translate** — never paste the raw English abstract.

Translation strategy (not word-for-word; **information compression + natural phrasing** in the user's language):

| Keep | Cut |
|---|---|
| Method / topology / algorithm | "In recent years...", "With the rapid development of..." |
| Key novelty (difference vs existing methods) | generic statements |
| Experimental results (accuracy / efficiency / power / frequency) | boilerplate courtesy |
| Validation scale (kW class, number of prototypes) | future work |
| Technical terms (ZVS / TPS / Coss / GaN / EPS / DAB / LLC / PFC...) stay in English, untranslated | — |

Aim for an **engineering register**, not a student-essay register. Example (Chinese):

❌ Student-style:
> 近年来，随着 DAB 转换器的快速发展，零电压开关变得非常重要。本文提出了一种新型的调制方法...

✅ Engineering-style:
> 提出针对 DAB 转换器的 TPS（三相移）调制，引入相移 + 占空比 + 频率多变量优化，1kW GaN HEMT 200kHz 原型实测峰值效率 97.1%。

Length: **2-4 sentences**. Longer is padding, shorter loses information.

## Field reference

| Field | Source | Note |
|---|---|---|
| 🔗 link | `https://doi.org/<doi>` | DOI redirect, most stable |
| 📅 year | `published_date` | year only |
| 📍 venue | inferred from DOI prefix (10.1109/tpel → TPEL etc.) | short name, do not expand |
| 📊 citations | `citations` field | write "0" explicitly, never omit |
| 👤 authors | `authors` field | first author in full, "et al." for the rest; add affiliation in parentheses when recognizable (ETH / Tsinghua / TU Delft ...) |

## Verdict vocabulary

| Verdict | Trigger condition |
|---|---|
| **must-read** | high citations (>50) + top journal (TPEL / TPAMI ...) + keywords in title |
| **representative** | mid-high citations (10-50) + top journal or conference |
| **new-method** | low citations but <2 years + top journal + novel angle |
| **background** | old foundational paper, for building concepts |
| **optional** | related but off the main line |
| **weakly-related** | keywords match but direction is off |

Follow each verdict with **one short reason** (a dozen words at most).

## Handling drops

Never drop silently — **close with a dedicated paragraph**:

```
Dropped X papers (reasons):
- "<title>" — <reason>
```

The user then knows you looked at them and did not miss anything.

## Closing question

After the cards, **always ask for the next step**:

> Which ones do you want in full text? Give me the numbers and I download them with ieee-fetch.
> Or say the word and I re-search with different keywords.

**No proactive bulk download** — wait for confirmation.
