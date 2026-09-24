<!-- 与 .claude/skills/tianzhi-core/SKILL.md 同源，去掉 frontmatter。改动以 SKILL.md 为正本。 -->

# tianzhi-core

[天秩](https://tianzhi.live)的计算底座。东方术数各门共用同一套坐标——干支、五行、节气、长生——
这个库把它做成代码。现阶段八字一门完整，六壬等门类在同一套底座上陆续加；历法与干支两层任何门类都能用。

**为什么要用它**：模型自己推八字会算错（干支、藏干、起运岁数），也会把出处编出来
（「《穷通宝鉴》云」后面接一句自己组织的话）。把这两件事交给库，你只负责讲人话。

```bash
pip install tianzhi-core
```

## 有什么

```python
from tianzhi_core.calendar import jieqi, solar_time   # 节气精确时刻、真太阳时
from tianzhi_core.core import ganzhi, wuxing          # 干支五行、藏干、刑冲合害、人元司令
from tianzhi_core.bazi import (
    chart,       # 排盘
    strength,    # 五行力量、日主旺衰
    tiaohou,     # 调候
    geju,        # 月令格局
    yongshen,    # 用喜忌仇闲
    luck_cycle,  # 起运、大运、流年、流月
    interact,    # 岁运对原局的刑冲合害
    score,       # 某年的相对分与曲线
    hepan,       # 两张盘的关系
    shensha,     # 神煞（参考层，取用不采信）
)
```

## 场景

下面是常见的几种，不止这些——凡是「先算、再讲」的问法都照这个路子来。

### 给了生辰，要看这张盘

```python
from datetime import datetime
from tianzhi_core.bazi import chart, strength, geju, yongshen

c = chart.build_chart(datetime(1996, 4, 18, 14, 6), longitude=114.93, gender=0)
s = strength.day_master_strength(c.quad, month_siling=c.siling)
g = geju.month_pattern(c.quad, month_siling=c.siling)
y = yongshen.select(c.quad, month_siling=c.siling)
```

`longitude` 是出生地经度，用来校正真太阳时——**不给就按钟表时间算，时柱可能整个错一位**。
`gender` 0 男 1 女，只影响大运顺逆。**凡是带 `month_siling` 的都要传 `c.siling`。**

讲的时候按这个顺序：先四柱与旺衰（确定的事实）→ 再格局与用神（结构，`y.method` 说明走的哪条路）
→ 最后才是倾向。

### 只知道四柱，不知道出生时刻

```python
quad = {"year": ("丙", "子"), "month": ("壬", "辰"), "day": ("乙", "酉"), "hour": ("癸", "未")}
strength.day_master_strength(quad)
```

`quad` 就是普通字典，上面所有函数都收它。**只有三柱时如实说时辰不详，不要挑一个合理的填进去。**

### 问某一年怎么样

```python
from tianzhi_core.bazi import luck_cycle, interact, score

dy = next(d for d in luck_cycle.dayun_list(c) if d.start_year <= 2026 <= d.end_year)
it = interact.combine(c.quad, year_gz='丙午', dayun_gz=dy.ganzhi)   # 刑冲合害、十神、伏吟反吟
r = score.score_year(c.quad, '丙午', dayun_gz=dy.ganzhi,
                     favorable=y.favorable, unfavorable=y.unfavorable)
```

**先定位大运再算流年**，不带大运的流年是半张。分数 **50 为不偏**，58.8 是「略偏顺」不是「及格」。
`it` 里的刑冲合害是用来解释**为什么**的，光报分数没有意义。

按月细看用 `luck_cycle.liuyue(c, 2026)`——**流月按节气分界，不按公历月**。

### 问两个人合不合

```python
from tianzhi_core.bazi import hepan

card = hepan.relation_card(a.quad, b.quad, a_favorable=ya.favorable, b_favorable=yb.favorable)
```

两张盘各自先取一次用神，再传进来——**不传的话互补度算不出来**。

### 要说依据

读 `y.evidence`，它给的是这一盘**实际走过的判据**：

```
('旺衰·中和', '格·偏印格', '病·水最旺', '药·土制之', '调候·火土', ...)
```

**不要凭记忆引古籍原文。** 需要出处就读这个字段，或者不引。

### 只要历法，不排盘

```python
jieqi.month_zhi(dt)                      # 这一刻属哪个月建，按节不按月初
jieqi.days_after_jieqi(dt)               # 距上一个节多少天，司令分日要用
solar_time.true_solar_time(dt, 114.93)   # 真太阳时
ganzhi.zhi_relation('子', '午')           # 两支的刑冲合害，返回 list（可能同时成立多种）
ganzhi.siling_gan('辰', 12.5)            # 人元司令
```

## 约束

- **凡是库能算的，一律调库**，不要因为「这个我会算」就自己推。
- **不凭记忆写典籍出处**，读 `evidence`。
- **流派分歧不替用户选边**。分歧处是参数：`late_zi`（晚子时归哪天）、`use_true_solar`、
  `yongshen.select(priority=...)`。结论对分歧敏感时说明用的哪一派。
- **结论落在库算出来的东西上。** 讲倾向、讲节奏、讲哪一步顺哪一步滞，都要能指回某个字段或某条判据；
  不要给医疗、法律、投资方面的专业建议。

## 相关

- 算法库 <https://github.com/zaoxu001/tianzhi-core> · <https://pypi.org/project/tianzhi-core/>
- 这份说明书 <https://github.com/zaoxu001/tianzhi-skill>
- 平台 <https://tianzhi.live>
