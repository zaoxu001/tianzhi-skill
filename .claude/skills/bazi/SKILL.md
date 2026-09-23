---
name: bazi
description: 用 tianzhi-core 排八字并读盘——四柱、旺衰、调候、格局、用神、大运流年、岁运引动、合盘。用户给出生辰问八字、问某一年运势、问两个人合不合时使用。所有判断一律由库算出，不要自行推演，也不要凭记忆写出处。
---

# 八字：排盘与读盘

模型自己推演八字会出两种错：**算错**（干支、藏干、起运岁数），和**编造**（引一句《穷通宝鉴》，但原文里没有这句）。
这个 skill 的全部意义是把这两件事交给一个确定性的库，你只负责把算出来的结果讲成人话。

## 装

```bash
pip install tianzhi-core
```

纯 Python，唯一依赖是 `lunar-python`。不读系统时间、不访问网络，同一输入永远同一输出。

## 最短路径

```python
from datetime import datetime
from tianzhi_core.bazi import chart, strength, yongshen, geju, luck_cycle, score

c = chart.build_chart(datetime(1996, 4, 18, 14, 6), longitude=114.93, gender=0)
print(c.bazi)          # 丙子 壬辰 乙酉 癸未
print(c.day_master)    # 乙    ← 日主，整张盘以它为「我」
print(c.siling)        # 戊    ← 月令司令，下面几乎每个函数都要带上

s = strength.day_master_strength(c.quad, month_siling=c.siling)
print(s.label, round(s.ratio, 3))        # 中和 0.453

y = yongshen.select(c.quad, month_siling=c.siling)
print(y.yong, y.xi, y.ji, y.chou)        # 土 火 木 水
print(y.method)                          # 病药   ← 这一盘是按哪条路取的用神
print(y.evidence)                        # ('旺衰·中和', '格·偏印格', '病·水最旺', '药·土制之', ...)

g = geju.month_pattern(c.quad, month_siling=c.siling)
print(g.name, g.ten_god, g.basis)        # 偏印格 偏印 透干

r = score.score_year(c.quad, '丙午', favorable=y.favorable, unfavorable=y.unfavorable)
print(r.score, r.stance, r.ten_god)      # 58.8 顺 伤官
```

`longitude` 是出生地经度，用来做真太阳时校正；不给就按钟表时间算。`gender` 0 男 1 女，只影响大运顺逆。

## 只知道四柱、不知道出生时刻

`quad` 就是一个普通字典，可以手搓：

```python
quad = {"year": ("丙", "子"), "month": ("壬", "辰"), "day": ("乙", "酉"), "hour": ("癸", "未")}
strength.day_master_strength(quad)        # 能算
yongshen.select(quad)                     # 能算
```

不传 `month_siling` 时按本气中气余气的通例处理，精度略低于传了的。

**时柱缺失就是缺失**：只有三柱时如实说明「时辰不详，时柱与由它推出的部分（如起运精确时刻）无法确定」，**不要挑一个看起来合理的时辰填进去**。

## 各层能拿到什么

| 模块 | 函数 | 返回的关键字段 |
|---|---|---|
| `chart` | `build_chart(dt, *, longitude, gender, use_true_solar, late_zi)` | `bazi` `quad` `day_master` `siling` `minggong` `shengong` `taiyuan` `solar` `solar_correction` |
| `strength` | `day_master_strength(quad, *, month_siling)` | `label` `category` `ratio` `has_root` `root_note` `items` |
| | `element_power(quad, *, month_siling)` | 五行力量 dict |
| | `ten_god_power(quad, *, month_siling)` | 十神力量 dict |
| `tiaohou` | `climate_need(quad)` | `gods` `kept` `dropped` `bing` `fallback` |
| `geju` | `month_pattern(quad, *, month_siling)` | `name` `ten_god` `gan` `basis` `transparent_at` `evidence` |
| | `scan_patterns(quad, *, month_siling)` | 本盘成象的其他格局 |
| `yongshen` | `select(quad, *, month_siling, priority)` | `yong` `xi` `ji` `chou` `xian` `method` `evidence` |
| `luck_cycle` | `start_age(chart)` → `(岁数, 交运时刻)` | 起运岁数带小数 |
| | `dayun_list(chart, count=10)` | `pillar` `start_age` `end_age` `start_year` `end_year` `forward` |
| | `liunian(chart, y0, y1)` / `liuyue(chart, year)` | 流年 / 流月 |
| `interact` | `combine(quad, *, year_gz, dayun_gz)` | 岁运对原局的刑冲合害、十神、伏吟反吟、移位、化气 |
| `score` | `score_year(quad, year_gz, *, dayun_gz, favorable, unfavorable)` | `score`(0–100) `stance` `breakdown` `terms` |
| | `curve(quad, cycles, ...)` | 逐年评分曲线 |
| `hepan` | `relation_card(a_quad, b_quad, *, a_favorable, b_favorable)` | 两张盘的关系指标、互补度、`score` `grade` |
| `shensha` | `shensha(quad)` | 神煞落点（**取用逻辑不采信，只作参考**） |

## 硬约束

**一、凡是库能算的，一律调库。**

旺衰、格局、用神、刑冲合害、起运岁数、大运干支、流年评分——**全部调函数**。
不要因为「这个我会算」就自己推。模型算这些的错误率不低，而且错得不显眼。

**二、不要凭记忆写典籍出处。**

需要依据时读 `evidence` 字段，它给的是这一盘实际走过的判据（如 `('旺衰·中和', '格·偏印格', '病·水最旺', '药·土制之')`）。
**不要写「《穷通宝鉴》云……」然后接一句自己组织的话**——这是这一行最常见的编造，而且读者查不动。

**三、流派分歧不要替用户选边。**

有分歧的地方库做成了参数，默认值只是默认值：

- `late_zi`：晚子时（23:00–23:59）的日柱算今天还是明天，默认 `"next_day"`
- `yongshen.select(priority=...)`：取用的优先顺序，默认 `('格局','扶抑','通关','病药','调候')`
- `use_true_solar`：是否做真太阳时校正，默认 `True`

碰到结论对分歧敏感时，**说明存在两派、本次用的是哪一派**，而不是只报一个结果。

**四、不作吉凶断言，不预测具体事件。**

库给的是结构：旺衰分档、格局成破、喜忌、岁运与原局的作用关系、一个 0–100 的相对分。
**这些是「势」，不是「事」。** 可以讲倾向、讲节奏、讲哪一步容易顺哪一步容易滞；
不要讲「某年必离婚」「某月会破财」这类具体事件，也不要给医疗、法律、投资建议。

## 怎么把结果讲给人听

1. **先摆事实**：四柱、日主、月令司令、旺衰分档。这些是确定的。
2. **再说结构**：格局是什么、凭什么（`basis`：透干还是会支）、用神取谁、走的哪条路（`method`）。
3. **最后才是倾向**：`score.stance` 是「顺」还是「逆」，配合 `interact.combine` 里的刑冲合害说清楚**为什么**。

分数只是相对值，**50 为不偏**。58.8 的意思是「略偏顺」，不是「及格了」。

## 常见错误

| 错误 | 正确做法 |
|---|---|
| 自己推四柱 | `build_chart` |
| 自己判旺衰 | `day_master_strength` |
| 忘了传 `month_siling` | 有 `chart` 就一定要传 `c.siling` |
| 用「出生年 + 起运岁数」算换运年份 | 起运岁数带小数，直接读 `DaYun.start_year` |
| 把 `shensha` 当作判断依据 | 它只是参考层，取用不采信 |
| 引一句古籍原文 | 读 `evidence`，或者不引 |

## 更多

- 库：<https://github.com/zaoxu001/tianzhi-core> · <https://pypi.org/project/tianzhi-core/>
- 平台：<https://tianzhi.live>
