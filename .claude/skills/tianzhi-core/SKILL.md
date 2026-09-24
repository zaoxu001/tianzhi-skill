---
name: tianzhi-core
description: 东方术数的计算底座。历法与真太阳时、节气精确时刻、干支五行关系、人元司令，以及八字的排盘、旺衰、调候、格局、取用、岁运引动、合盘。用户给出生辰要看盘、问某年运势、问两人合不合，或需要任何干支历法计算时使用。所有判断由库算出，不要自行推演，也不要凭记忆写典籍出处。
---

# tianzhi-core · 东方术数的计算底座

术数各门（八字、六壬、六爻、奇门、紫微、梅花）典籍与推演之法各异，底下却共用同一套坐标：
**干支、五行、节气、长生、旺相休囚死**。这个库把这套共用的底座做成代码，再在其上按门类展开。

**现阶段八字一门完整**（排盘、排运、量化、取用、引动、合盘、神煞），其余门类在同一套底座上陆续加；
历法与干支两层已完成，任何门类都能直接用。

两件事是它存在的理由：

- **算对。** 干支、藏干、节气时刻、起运岁数属于查表与推导，规则明确。模型自己推错误率不低，
  而且错得不显眼——盘能排出来，只是排的是别人的。
- **说得出依据。** 有出处的规则在代码里注明篇名，判据随结果一起返回（`evidence` 字段）。
  不必也不该凭记忆引原文。

## 装

```bash
pip install tianzhi-core
```

Python 3.10+，唯一依赖 `lunar-python`。不读系统时间、不访问网络、不碰文件系统（包内数据表除外），
同一输入永远得到同一输出，跨进程一致。

## 分层

| 层 | 内容 | 性质 |
|---|---|---|
| `calendar` | 节气精确时刻、真太阳时 | 天文计算，只有对错，无流派 |
| `core` | 干支五行、藏干、刑冲合害、十二长生、旺相休囚死、人元司令 | 基础常识，其上所有逻辑依赖于此 |
| `bazi` | 排盘、旺衰、调候、格局、取用、引动、评分、合盘、神煞 | **流派分歧集中在这一层** |
| `data` | 由典籍编码而成的数据表 | 调候一百二十格、金不换、司令分日 |

这个分层不是装饰：`calendar` 算错了是 bug，`bazi` 结论不同可能只是口径不同。

---

## calendar · 历法与纪时

```python
from tianzhi_core.calendar import jieqi, solar_time
```

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `jieqi.jieqi_table(year)` | 该年二十四节气的精确时刻 | 天文历算，精到分钟，不是查固定日期 |
| `jieqi.jie_table(year)` | 只取十二「节」（立春、惊蛰…） | 月建换月看节不看气 |
| `jieqi.month_zhi(dt)` | 这一刻属哪个月建（寅卯辰…） | **按节分界**，不按公历月初 |
| `jieqi.month_jieqi(dt)` | 当前所处的节及其时刻 | 同上 |
| `jieqi.days_after_jieqi(dt)` | 距上一个节多少天（带小数） | 人元司令分日要用它 |
| `jieqi.next_jie(dt)` | 下一个节及其时刻 | 起运推算要用 |
| `jieqi.lichun(year)` | 立春时刻 | 年柱换年看立春，不看元旦 |
| `solar_time.equation_of_time(dt)` | 时差：真太阳时与平太阳时之差 | 地球轨道偏心率与黄赤交角造成，一年内在 ±16 分钟间摆动 |
| `solar_time.longitude_offset(lon)` | 经度造成的分钟差 | 与标准经线（默认 120°E）每差 1° 为 4 分钟 |
| `solar_time.correction_minutes(dt, lon)` | 两者之和，总校正量 | 时差 + 经度差 |
| `solar_time.true_solar_time(dt, lon)` | 校正后的真太阳时 | 钟表时间 + 总校正量 |

**为什么要真太阳时**：时柱按太阳的实际位置定，而钟表走的是统一时区。乌鲁木齐与北京同用东八区，
实际日照差近两小时——不校正，时柱可能整个错一位。

---

## core · 干支与五行

```python
from tianzhi_core.core import ganzhi, wuxing
```

**数据表**（直接读，不要自己背）：
`JIAZI` 六十甲子 · `GAN_WUXING` `ZHI_WUXING` 干支五行 · `GAN_YANG` `ZHI_YANG` 阴阳 ·
`HIDDEN` 地支藏干 · `PURE_ZHI` 纯气支（子卯酉，只藏一个字）·
`GAN_HE` `GAN_CHONG` 天干合冲 · `ZHI_LIUHE` `ZHI_SANHE` `ZHI_SANHUI` 地支六合三合三会

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `ganzhi.gan_relation(a, b)` | 两天干的合或冲 | 查表，一对最多成立一种 |
| `ganzhi.zhi_relation(a, b)` | 两地支的刑冲合害 | 查表。**返回 list**——一对地支可能同时成立多种关系 |
| `ganzhi.zhi_groups(zhis)` | 三合、三会 | 要三支齐才成组，所以接整组而非两两 |
| `ganzhi.dishi(gan, zhi)` | 十二长生位（长生、沐浴…帝旺…） | 天干在地支上的得地程度 |
| `ganzhi.wangxiang(target, month_zhi)` | 旺相休囚死 | 五行在某月令下的状态 |
| `ganzhi.siling_gan(month_zhi, days)` | 人元司令：当令的是哪个天干 | 按**距节天数**查分日表，同一个月里前后期当令者不同 |
| `ganzhi.jiazi_index(ganzhi)` | 在六十甲子中的序号 | 0–59 |
| `wuxing.relation(target, me)` | 生我、我生、克我、我克、同我 | 五行生克，十神由此推出 |
| `wuxing.counts_to_ratio(counts)` | 五行力量归一化 | 各占全盘的比例 |

---

## bazi · 八字

```python
from tianzhi_core.bazi import chart, strength, tiaohou, geju, yongshen, luck_cycle, interact, score, hepan, shensha
```

### 排盘

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `chart.build_chart(dt, *, longitude=0, gender=0, use_true_solar=True, late_zi='next_day')` | 一张完整的盘 | 先校正真太阳时，再定年月日时四柱、命宫身宫胎元 |
| `chart.nayin(ganzhi)` | 纳音 | 查表 |

返回的 `Chart`：`bazi` 四柱字符串 · `quad` 四柱字典 · `day_master` 日主 · `siling` 月令司令 ·
`minggong` `shengong` `taiyuan` 命宫身宫胎元 · `solar` 校正后时刻 · `solar_correction` 校正了多少分钟

`quad` 是个普通字典，**不知道出生时刻也能用**：

```python
quad = {"year": ("丙", "子"), "month": ("壬", "辰"), "day": ("乙", "酉"), "hour": ("癸", "未")}
```

### 量化

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `strength.day_master_strength(quad, *, month_siling)` | 日主旺衰五档 | 全盘拆成成分，每个成分权重 = 基础分 × 根气 × 纯气 × 月令 × 司令 × 贴身 × 虚透；同党（比劫+印）比全盘，阈值按三千张随机盘的分位数标定 |
| `strength.element_power(quad, *, month_siling)` | 五行力量 | 同一套权重汇总到五行 |
| `strength.ten_god_power(quad, *, month_siling)` | 十神力量 | 同一套权重汇总到十神 |
| `strength.climate_index(quad)` | 寒暖燥湿标量 | 金水为寒木火为暖，辰丑湿未戌燥，按位置加权 |

`Strength`：`label` 五档标签 · `ratio` 归一化比值 · `has_root` **强根布尔闸**（得长生/临官/帝旺，
或藏干同五行多于两个）· `items` 每个成分的明细，**可以摊开给人看这个数怎么来的**

**旺衰与强根并联、不相乘**：从格、专旺成不成立要看闸，不能只看连续分。

### 取用

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `tiaohou.climate_need(quad)` | 调候所需 | 《穷通宝鉴》十天干乘十二月令一百二十格，忌神另参《金不换大运》 |
| `geju.month_pattern(quad, *, month_siling)` | 月令格局 | 《子平真诠》月令取格，`basis` 说明凭透干还是会支 |
| `geju.scan_patterns(quad, *, month_siling)` | 本盘另外成象的格局 | 扫全盘，与月令格并列 |
| `geju.pattern_ops(name)` / `ops_conflicts(...)` | 该格的顺用逆用与相神，以及本盘相抵之处 | 《子平真诠》八条相神规则照原文编成判据 |
| `yongshen.select(quad, *, month_siling, priority=('格局','扶抑','通关','病药','调候'))` | 用喜忌仇闲 | 按 priority 依次尝试，先成立者胜；`method` 告诉你走的哪条路 |

`YongShen`：`yong` `xi` `ji` `chou` `xian` 五位 · `method` 取用路径 · **`evidence` 这一盘实际走过的判据**

### 排运与引动

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `luck_cycle.is_forward(chart)` | 大运顺排还是逆排 | 年干阴阳配性别 |
| `luck_cycle.start_age(chart)` | `(起运岁数, 交运时刻)` | 到下一个节（或上一个节）的天数换算，**岁数带小数** |
| `luck_cycle.dayun_list(chart, count=10)` | 大运各步 | 每步十年；`start_year` `end_year` 已算好，**不要自己拿出生年加岁数** |
| `luck_cycle.liunian(chart, y0, y1)` / `liuyue(chart, year)` | 流年 / 流月 | 流月按节气分界，不按公历月 |
| `interact.combine(quad, *, year_gz, dayun_gz)` | 岁运对原局的作用 | 刑冲合害、十神、伏吟反吟、移位、化气，逐条列出 |
| `score.score_year(quad, year_gz, *, dayun_gz, favorable, unfavorable)` | 该年 0–100 的相对分 | 以 50 为不偏，喜用加分忌仇减分，`breakdown` 给出每一项的来源 |
| `score.curve(quad, cycles, ...)` | 逐年评分曲线 | 同上批量 |

**分数是相对值，50 为不偏。** 58.8 的意思是「略偏顺」，不是「及格」。

### 合盘与神煞

| 方法 | 作用 | 怎么算的 |
|---|---|---|
| `hepan.relation_card(a_quad, b_quad, *, a_favorable, b_favorable)` | 两张盘的关系指标 | 日干生克、互看十神、四柱对位、五行互补度，出 `score` 与 `grade` |
| `shensha.shensha(quad)` | 神煞落点 | 查表。**取用逻辑不采信，只作参考层** |
| `shensha.xun_kong(day_gan, day_zhi)` | 旬空 | 由日柱定 |

---

## 用法：最短一条路

```python
from datetime import datetime
from tianzhi_core.bazi import chart, strength, yongshen, geju, score

c = chart.build_chart(datetime(1996, 4, 18, 14, 6), longitude=114.93, gender=0)
print(c.bazi)          # 丙子 壬辰 乙酉 癸未
print(c.day_master)    # 乙    ← 日主，整张盘以它为「我」
print(c.siling)        # 戊    ← 月令司令，下面几乎每个函数都要带上

s = strength.day_master_strength(c.quad, month_siling=c.siling)
print(s.label, round(s.ratio, 3))        # 中和 0.453

y = yongshen.select(c.quad, month_siling=c.siling)
print(y.yong, y.xi, y.ji, y.chou)        # 土 火 木 水
print(y.method)                          # 病药
print(y.evidence)                        # ('旺衰·中和', '格·偏印格', '病·水最旺', '药·土制之', ...)

g = geju.month_pattern(c.quad, month_siling=c.siling)
print(g.name, g.ten_god, g.basis)        # 偏印格 偏印 透干

r = score.score_year(c.quad, '丙午', favorable=y.favorable, unfavorable=y.unfavorable)
print(r.score, r.stance, r.ten_god)      # 58.8 顺 伤官
```

`longitude` 是出生地经度，用于真太阳时校正；不给就按钟表时间算。`gender` 0 男 1 女，只影响大运顺逆。

---

## 硬约束

**一、凡是库能算的，一律调库。**
旺衰、格局、用神、刑冲合害、起运岁数、大运干支、流年评分——全部调函数。
不要因为「这个我会算」就自己推。

**二、不要凭记忆写典籍出处。**
需要依据时读 `evidence`。**不要写「《穷通宝鉴》云……」然后接一句自己组织的话**——
这是这一行最常见的编造，而且读者查不动。

**三、流派分歧不替用户选边。**
分歧处做成了参数，默认值只是默认值：

- `late_zi`：晚子时（23:00–23:59）的日柱算今天还是明天，默认 `"next_day"`
- `yongshen.select(priority=...)`：取用的优先顺序
- `use_true_solar`：是否做真太阳时校正，默认 `True`

结论对分歧敏感时，说明存在两派、本次用的是哪一派。

**四、时柱缺失就是缺失。**
只有三柱时如实说明「时辰不详，时柱与由它推出的部分无法确定」，
**不要挑一个看起来合理的时辰填进去**。

**五、不作吉凶断言，不预测具体事件。**
库给的是结构——旺衰分档、格局成破、喜忌、岁运与原局的作用关系、一个相对分。
**这些是「势」，不是「事」。** 可以讲倾向、讲节奏；不要讲「某年必离婚」这类具体事件，
也不要给医疗、法律、投资建议。

## 怎么把结果讲给人听

1. **先摆事实**：四柱、日主、月令司令、旺衰分档。这些是确定的。
2. **再说结构**：格局是什么、凭什么（`basis`）、用神取谁、走的哪条路（`method`）。
3. **最后才是倾向**：`stance` 是顺还是逆，配合 `interact.combine` 说清楚**为什么**。

## 常见错误

| 错误 | 正确做法 |
|---|---|
| 自己推四柱 | `chart.build_chart` |
| 自己判旺衰 | `strength.day_master_strength` |
| 忘了传 `month_siling` | 有 `chart` 就一定要传 `c.siling` |
| 用「出生年 + 起运岁数」算换运年份 | 起运岁数带小数，直接读 `DaYun.start_year` |
| 按公历月分流月 | 流月按节气分界，用 `luck_cycle.liuyue` |
| 把 `shensha` 当判断依据 | 它只是参考层，取用不采信 |
| 引一句古籍原文 | 读 `evidence`，或者不引 |

## 更多

- 库：<https://github.com/zaoxu001/tianzhi-core> · <https://pypi.org/project/tianzhi-core/>
- 平台：<https://tianzhi.live>
