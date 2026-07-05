# glasses-recommendation

成人单光近视/近视散光配镜建议 skill。

这个仓库提供一套面向 AI agent 的配镜决策流程，用来帮助用户把验光数据、瞳距、镜框参数、使用场景和预算整理成可执行的购物建议。它强调先判断数据可信度和镜框适配性，再推荐折射率、镜片档位和品牌示例。

> 重要说明：本项目不是医疗诊断工具，不能替代医生、视光师或线下专业验光。出现眼痛、突然视力变化、闪光感、视野缺损、外伤、复视等情况时，应优先就医。

## 适用场景

- 成人单光近视配镜
- 成人近视合并散光配镜
- 根据 `OD / OS / PD / 镜框参数 / 用途 / 预算` 做普通框架眼镜购物建议
- 判断镜框尺寸、框心距、片高、鼻托稳定性是否适合当前度数
- 解释折射率、镜片档位、防蓝光、变色片、蔡司泽锐/智锐等问题
- 在完成验光和镜框判断后，给出更稳妥的镜片品牌/系列候选

## 不适用场景

- 儿童青少年近视防控
- 渐进、多焦点、老花、抗疲劳或视功能类验配
- 隐形眼镜
- 眼病诊断、术后场景或急性眼部症状
- 仅凭电脑验光结果做高端定制镜片定案

## 核心思路

这个 skill 不会一上来推荐品牌或贵镜片，而是按下面的顺序工作：

1. 同时判断字段完整度和处方可信度
2. 将案例分为 `Green / Yellow / Red`
3. 先推导镜框约束，并展示 `FPD`、移心量、目标框心距、片高压力的计算公式
4. 再给折射率建议，例如 `1.60 / 1.71 / 1.74`
5. 最后才给普通单光、中端升级或高端定制档位建议
6. 如果用户需要品牌，再按档位和场景给 2-4 个候选系列

镜框推荐必须展示计算口径，不能只写“选小框”。常用公式包括：

```text
FPD = A + DBL
只有总 PD 时：单眼移心量 ≈ abs(FPD - 总PD) / 2
目标 FPD 优秀范围：<= 总PD + 3 mm
目标 FPD 实用范围：<= 总PD + 6 mm
```

品牌推荐必须排在数据、镜框、折射率和档位之后。用户询问“最新款”“今年新款”“某地区是否能买到”时，应重新查品牌官方产品页或当地官方渠道，不只依赖内置清单。

## 目录结构

```text
glasses-recommendation/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── brand-recommendation-reference.md
    ├── conversation-templates.md
    ├── linked-sources.md
    ├── selection-reference.md
    └── test-cases.md
```

- `SKILL.md`：主流程和触发规则
- `agents/openai.yaml`：界面展示元数据
- `references/selection-reference.md`：公式、阈值、档位模板
- `references/brand-recommendation-reference.md`：镜片品牌/系列候选、特殊需求分流和最新产品核验规则
- `references/conversation-templates.md`：问诊话术和完整示例
- `references/linked-sources.md`：来源、证据边界和延伸阅读
- `references/test-cases.md`：人工检查和 forward-testing 用例

## 安装方式

不同工具对 skills 目录的约定不完全相同。安装时要保留整个 `glasses-recommendation/` 目录结构，确保 `SKILL.md`、`agents/` 和 `references/` 都在同一个目录下。

### Codex

```bash
mkdir -p ~/.codex/skills
cp -R glasses-recommendation ~/.codex/skills/glasses-recommendation
```

使用示例：

```text
使用 $glasses-recommendation 审查我的验光数据，推导镜框参数，并给出镜片建议。
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
cp -R glasses-recommendation ~/.claude/skills/glasses-recommendation
```

使用示例：

```text
使用 $glasses-recommendation 审查我的验光数据，推导镜框参数，并给出镜片建议。
```

### OpenCode / Open Code

OpenCode 不同版本和配置对 skills 的加载方式可能不同。推荐两种方式：

1. 如果你的 OpenCode 已配置 Claude/Codex 风格的 skills 目录，把本目录复制到对应 skills 目录。
2. 如果没有统一 skills 目录，放到一个固定共享位置，并在对话中显式引用 `SKILL.md`。

示例：

```bash
mkdir -p ~/.agents/skills
cp -R glasses-recommendation ~/.agents/skills/glasses-recommendation
```

使用示例：

```text
请读取 ~/.agents/skills/glasses-recommendation/SKILL.md，按 glasses-recommendation skill 帮我审查验光数据并给出配镜建议。
```

## 输入示例

```text
右眼 OD：-5.00
左眼 OS：-4.50 / -0.50 x 180
PD：64
用途：日常通勤 + 电脑办公
预算：300-600
旧镜体验：整体舒服，但镜框有点大，久戴容易下滑
看中镜框：50□19，片高 42
```

skill 应优先输出镜框适配判断，再给折射率和镜片档位建议，而不是直接按品牌或价格下结论。

镜框评估应包含类似计算：

```text
FPD = A + DBL = 50 + 19 = 69
单眼移心量 ≈ abs(69 - 64) / 2 = 2.5 mm
```

如果用户继续问品牌，skill 应先确认档位，再给候选。例如中端升级型可讨论蔡司 `ClearView`、依视路中端单光 / `Crizal` 膜层相关产品、豪雅常规或中端单光线、明月高性价比升级线等。儿童近视管理产品如 `MyoCare / MiYOSMART / Stellest` 不应推荐给成人普通单光场景。

## 验证

如果你有 Codex skill creator 的校验脚本，可以运行：

```bash
python3 /path/to/skill-creator/scripts/quick_validate.py /path/to/glasses-recommendation
```

也可以用 `references/test-cases.md` 做人工回归检查，重点看这些行为是否稳定：

- 纯球镜没有 `AXIS` 时不误判为信息缺失
- 只有电脑验光时不进入高端定制定案
- 已给镜框参数时先算 `FPD` 和移心量，并展示公式和代入过程
- 防蓝光不作为默认必选项
- 品牌推荐不越过验光、镜框、折射率和档位判断
- 儿童防控、眼病、术后和急性症状进入停止购物建议路径

## 贡献建议

欢迎提交 issue 或 pull request。较适合贡献的方向包括：

- 新增真实但脱敏的测试用例
- 补充不同度数、散光轴位和镜框尺寸组合的边界案例
- 改进输出模板的可读性
- 补充可公开访问的权威来源
- 补充品牌官方产品页和地区上市差异说明

贡献时请保持 `SKILL.md` 精简，把长示例、测试用例和来源说明放入 `references/`。

## License

MIT
