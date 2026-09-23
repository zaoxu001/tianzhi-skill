# tianzhi-skill · 让 AI 正确地排八字

给 AI 用的一份说明书：**八字里所有能算的部分，交给 [tianzhi-core](https://github.com/zaoxu001/tianzhi-core) 算，模型只负责讲人话。**

## 为什么需要它

模型自己推八字会犯两类错：

**算错。** 干支、藏干、起运岁数、刑冲合害——这些是查表与推导，规则明确，但模型算起来错误率不低，而且错得不显眼：盘能排出来，只是排的是别人的。

**编造。** 「《穷通宝鉴》云……」后面接一句自己组织的话。听起来很像那么回事，读者也查不动。这是这一行最普遍的问题，不只出现在模型身上。

这份 skill 把前者交给一个有 181 个测试锁着的确定性库，把后者约束成「读 `evidence` 字段，或者不引」。

## 装

```bash
git clone https://github.com/zaoxu001/tianzhi-skill
```

Claude Code 从仓库根目录的 `.claude/skills/` 读取 skill。放进你的项目，或者把 `.claude/skills/bazi/` 复制到你自己的仓库里。

底下的库单独装：

```bash
pip install tianzhi-core
```

## 它约束了什么

- 凡是库能算的，一律调库，不自行推演
- 不凭记忆写典籍出处，需要依据时读 `evidence`
- 流派分歧做成参数，说明用的是哪一派，不替用户选边
- 不作吉凶断言，不预测具体事件

## 相关

- [tianzhi-core](https://github.com/zaoxu001/tianzhi-core) —— 底下的算法库，MIT
- [天秩](https://tianzhi.live) —— 东方术数的研究与学习平台

## 许可

MIT
